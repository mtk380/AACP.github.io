# 6. AAP：行动问责协议

## 6.1 AAP 定位

AAP（Agent Action Protocol）管理 Agent 任务从 **签约到结算** 的完整生命周期，是 AACP 的问责核心——每一步操作都留下不可篡改的链上证据。

```
┌────────────────────────────────────────────────────────────┐
│                     AAP = 行动 + 问责                       │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   任务生命周期                         │  │
│  │                                                      │  │
│  │  签约 → 授权 → 执行 → 报告 → 验证 → 结算             │  │
│  │   │      │      │      │      │      │               │  │
│  │   ▼      ▼      ▼      ▼      ▼      ▼               │  │
│  │  SLA   Cap    Action  Result  Proof  Payment          │  │
│  │  签署  发放    日志    提交    验证   释放              │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   证据链（全程上链）                    │  │
│  │                                                      │  │
│  │  Block N   Block N+1  Block N+k  Block N+m           │  │
│  │  ┌─────┐  ┌─────┐    ┌─────┐    ┌─────┐             │  │
│  │  │SLA  │→│Cap  │→···→│Result│→ │Settle│             │  │
│  │  │Hash │  │Grant│    │Hash │    │Final│             │  │
│  │  └─────┘  └─────┘    └─────┘    └─────┘             │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

## 6.2 任务状态机

```
                          AAP 任务状态机

                        ┌──────────────┐
                        │   CREATED    │
                        └──────┬───────┘
                               │ both_sign_sla
                               ▼
                        ┌──────────────┐
                        │   SLA_SIGNED │
                        └──────┬───────┘
                               │ deposit_confirmed
                               ▼
                        ┌──────────────┐
                        │  DEPOSITED   │
                        └──────┬───────┘
                               │ grant_capabilities
                               ▼
                        ┌──────────────┐
                   ┌───→│  AUTHORIZED  │
                   │    └──────┬───────┘
                   │           │ agent_starts_execution
                   │           ▼
                   │    ┌──────────────┐
                   │    │  EXECUTING   │←─── heartbeat
                   │    └──────┬───────┘     (每30s)
                   │           │
                   │    ┌──────┴───────┐
                   │    │              │
                   │    ▼              ▼
          timeout/ │ ┌──────┐   ┌──────────┐
          retry    │ │SUBMIT│   │  FAILED   │──→ 自动争议
                   │ │RESULT│   └──────────┘
                   │ └──┬───┘
                   │    │ result_submitted
                   │    ▼
                   │ ┌──────────────┐
                   │ │  VERIFYING   │
                   │ └──────┬───────┘
                   │        │
                   │ ┌──────┴───────────┐
                   │ │                  │
                   │ ▼                  ▼
                   │ ┌──────────┐ ┌──────────┐
                   │ │ VERIFIED │ │ VER_FAIL │──→ 可重试(≤3次)
                   │ └────┬─────┘ └──────────┘       │
                   │      │                          │
                   └──────┘(重试)                     │(超限)
                          │                          │
                          ▼                          ▼
                   ┌──────────────┐          ┌──────────────┐
                   │   SETTLING   │          │   DISPUTED   │
                   └──────┬───────┘          └──────┬───────┘
                          │ settlement_done          │ arb_result
                          ▼                          ▼
                   ┌──────────────┐          ┌──────────────┐
                   │  COMPLETED   │          │   RESOLVED   │
                   └──────────────┘          └──────────────┘
```

| 状态 | 持续时限 | 超时后果 |
|------|---------|---------|
| CREATED | 30 min | 自动取消，退还 gas |
| SLA_SIGNED | 1 h | 取消，扣 Consumer 少量手续费 |
| DEPOSITED | 15 min | 取消，退还保证金 |
| AUTHORIZED | 5 min | 回退到 DEPOSITED |
| EXECUTING | SLA 约定 | 进入 FAILED → DISPUTED |
| VERIFYING | 10 min | 自动通过（乐观验证） |
| SETTLING | 5 min | 系统强制结算 |
| DISPUTED | 72 h | 仲裁超时默认判 Consumer 胜 |

## 6.3 SLA 模板

```protobuf
// aacp.v1.aap — SLA 定义
message SLA {
  string  task_id           = 1;
  bytes   consumer          = 2;  // Ed25519 pubkey
  bytes   provider          = 3;

  // 性能约束
  uint32  max_latency_ms    = 4;   // 最大端到端延迟
  float   min_accuracy      = 5;   // 最低准确率 (0.0–1.0)
  uint32  max_retries       = 6;   // 最大重试次数, 默认 3
  uint64  timeout_sec       = 7;   // 执行总超时

  // 经济条款
  Money   service_fee       = 8;   // 服务费
  Money   deposit_amount    = 9;   // Provider 保证金
  uint32  commission_bps    = 10;  // 佣金, 800–1500 (8%–15%)

  // 签名
  bytes   consumer_sig      = 11;
  bytes   provider_sig      = 12;
  int64   signed_at         = 13;
  int64   expires_at        = 14;
}
```

## 6.4 证据链设计

每个 AAP 任务产生一条 **不可篡改的证据链**，用于事后审计和争议仲裁：

```
  证据链结构（Merkle 链）

  EvidenceRoot (存入 IAVL State Tree)
       │
       ├── SLA_Hash ─────── SHA-256(SLA protobuf bytes)
       │
       ├── Cap_Grant_Hash ── SHA-256(granted Capability UTXO IDs)
       │
       ├── ActionLog[] ──── 执行期间的关键操作日志
       │    ├── ActionLog[0]: {timestamp, action, input_hash, output_hash}
       │    ├── ActionLog[1]: ...
       │    └── ActionLog[N]: ...
       │
       ├── Result_Hash ──── SHA-256(result payload)
       │
       ├── Verify_Hash ──── SHA-256(verification report)
       │
       └── Settle_Hash ──── SHA-256(settlement receipt)
  
  每个节点: hash = SHA-256(left_child || right_child)
  根哈希写入区块头的 app_hash (通过 IAVL Commit)
```

```go
// pkg/aap/evidence.go

package aap

import (
    "crypto/sha256"
    "encoding/binary"
)

type EvidenceChain struct {
    TaskID      string
    SLAHash     [32]byte
    CapHash     [32]byte
    ActionLogs  []ActionLogEntry
    ResultHash  [32]byte
    VerifyHash  [32]byte
    SettleHash  [32]byte
}

type ActionLogEntry struct {
    Timestamp  int64
    Action     string
    InputHash  [32]byte
    OutputHash [32]byte
}

func (ec *EvidenceChain) ComputeRoot() [32]byte {
    leaves := make([][32]byte, 0, 6+len(ec.ActionLogs))
    leaves = append(leaves, ec.SLAHash)
    leaves = append(leaves, ec.CapHash)
    for _, log := range ec.ActionLogs {
        leaves = append(leaves, log.Hash())
    }
    leaves = append(leaves, ec.ResultHash, ec.VerifyHash, ec.SettleHash)
    return merkleRoot(leaves)
}

func (e *ActionLogEntry) Hash() [32]byte {
    buf := make([]byte, 8+len(e.Action)+64)
    binary.BigEndian.PutUint64(buf[:8], uint64(e.Timestamp))
    copy(buf[8:], e.Action)
    copy(buf[8+len(e.Action):], e.InputHash[:])
    copy(buf[8+len(e.Action)+32:], e.OutputHash[:])
    return sha256.Sum256(buf)
}

func merkleRoot(leaves [][32]byte) [32]byte {
    if len(leaves) == 0 {
        return sha256.Sum256(nil)
    }
    if len(leaves) == 1 {
        return leaves[0]
    }
    if len(leaves)%2 != 0 {
        leaves = append(leaves, leaves[len(leaves)-1])
    }
    next := make([][32]byte, len(leaves)/2)
    for i := range next {
        combined := append(leaves[2*i][:], leaves[2*i+1][:]...)
        next[i] = sha256.Sum256(combined)
    }
    return merkleRoot(next)
}
```

## 6.5 心跳与活性检测

执行期间 Provider Agent 必须每 **30 秒** 提交心跳，证明任务仍在正常推进：

```
  Consumer           Chain            Provider Agent
     │                 │                    │
     │                 │←── Heartbeat ──────│  t=0s
     │                 │    {task_id,       │
     │                 │     progress: 15%, │
     │                 │     resource_usage} │
     │                 │                    │
     │                 │←── Heartbeat ──────│  t=30s
     │                 │    {progress: 40%} │
     │                 │                    │
     │                 │    ... 90s 无心跳 ...
     │                 │                    │
     │                 │── HealthCheck ────→│  t=120s (链上自动)
     │                 │                    │
     │        (若 180s 仍无响应)             │
     │                 │                    │
     │←─ FAILED ──────│                    │
     │  (自动进入争议)  │                    │
```

---
