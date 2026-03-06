# 8. Capability-UTXO 模型

## 8.1 设计动机

传统 RBAC/ABAC 权限模型是"谁可以做什么"——适合人类用户，但不适合 **Agent 经济**：

| 需求 | 传统 RBAC | AACP Capability-UTXO |
|------|----------|---------------------|
| 权限可交易 | 不支持 | UTXO 天然可转移 |
| 细粒度授权 | 角色粒度粗 | 每个 Capability 独立 UTXO |
| 可审计 | 日志散落 | 链上完整花费链 |
| 时效性 | 需额外管理 | UTXO 内置 TTL |
| 组合性 | 难以组合 | 多个 UTXO 可聚合消费 |
| 委托/转授 | 复杂 | 原生支持派生 UTXO |

> **核心思想**：Agent 的每一项能力是一个 **未花费的能力令牌（Capability-UTXO）**。
> 授权 = 创建 UTXO，使用 = 花费 UTXO，撤销 = 销毁 UTXO。

## 8.2 UTXO 结构

```protobuf
// aacp.v1.caputxo
message CapabilityUTXO {
  string  utxo_id      = 1;  // SHA-256(issuer + cap_type + nonce)[:16] hex
  bytes   issuer       = 2;  // 发行者 Ed25519 公钥
  bytes   holder       = 3;  // 当前持有者 Ed25519 公钥

  // 能力定义
  string  cap_type     = 4;  // 能力类型标识, e.g. "text_generation"
  string  cap_version  = 5;  // 语义化版本 "1.2.0"
  repeated string scopes = 6; // 细粒度权限范围

  // 约束
  int64   issued_at    = 7;
  int64   expires_at   = 8;  // TTL, 0 = 永不过期
  uint32  max_uses     = 9;  // 最大使用次数, 0 = 不限
  uint32  used_count   = 10;

  // 派生约束
  bool    delegatable  = 11; // 是否可转授
  uint32  max_depth    = 12; // 最大委托深度
  uint32  cur_depth    = 13;
  string  parent_utxo  = 14; // 派生自哪个 UTXO（根 UTXO 为空）

  // 状态
  UTXOStatus status    = 15;
  bytes   holder_sig   = 16; // 持有者签名（接受时签）
}

enum UTXOStatus {
  UTXO_UNSPENT  = 0; // 可用
  UTXO_SPENT    = 1; // 已消费
  UTXO_EXPIRED  = 2; // 已过期
  UTXO_REVOKED  = 3; // 被发行者撤销
}
```

## 8.3 UTXO 生命周期

```
              Capability-UTXO 生命周期

  Issuer                    Chain                    Holder
    │                         │                        │
    │── 1. MintCapability ──→│                        │
    │   {cap_type, scopes,   │                        │
    │    holder, ttl,        │                        │
    │    delegatable}         │                        │
    │                         │                        │
    │                         │── 2. UTXO Created ──→ │
    │                         │   status=UNSPENT       │
    │                         │                        │
    │                         │   ... Agent 执行任务 ... │
    │                         │                        │
    │                         │←─ 3. SpendCapability ──│
    │                         │   {utxo_id, task_id,   │
    │                         │    proof_of_use}       │
    │                         │                        │
    │                         │   验证:                 │
    │                         │   ✓ status==UNSPENT    │
    │                         │   ✓ holder 签名有效     │
    │                         │   ✓ 未过期              │
    │                         │   ✓ used_count<max_uses │
    │                         │                        │
    │                         │── 4. status=SPENT ───→ │
    │                         │                        │
    │                         │                        │
    │── 5. RevokeCapability ─│  (可随时撤销未花费UTXO)  │
    │   {utxo_id}            │                        │
    │                         │── status=REVOKED ────→ │
```

## 8.4 派生与委托

Agent A 可以将自己持有的 Capability 部分地委托给 Agent B，形成 **派生 UTXO 树**：

```
  派生 UTXO 树示例

  Root UTXO (由平台签发)
  ├── cap_type: "data_processing"
  ├── scopes: ["read_db", "write_db", "call_api"]
  ├── delegatable: true
  ├── max_depth: 3
  └── holder: Agent-A
        │
        ├── Derived UTXO-1 (Agent-A → Agent-B)
        │   ├── scopes: ["read_db"]         ← 范围收窄
        │   ├── max_uses: 10                ← 额外限制
        │   ├── cur_depth: 1
        │   └── holder: Agent-B
        │         │
        │         └── Derived UTXO-2 (Agent-B → Agent-C)
        │             ├── scopes: ["read_db"]
        │             ├── max_uses: 3       ← 更严格
        │             ├── cur_depth: 2
        │             └── holder: Agent-C
        │
        └── Derived UTXO-3 (Agent-A → Agent-D)
            ├── scopes: ["call_api"]
            ├── cur_depth: 1
            └── holder: Agent-D
```

**派生规则**（链上强制执行）：

| 规则 | 描述 | 违反后果 |
|------|------|---------|
| 范围只缩不扩 | 派生 UTXO 的 scopes ⊆ 父 UTXO 的 scopes | Tx 被拒绝 |
| 深度递增 | cur_depth = parent.cur_depth + 1 | Tx 被拒绝 |
| 深度上限 | cur_depth ≤ root.max_depth | Tx 被拒绝 |
| TTL 只短不长 | derived.expires_at ≤ parent.expires_at | Tx 被拒绝 |
| 级联撤销 | 父 UTXO 被撤销 → 所有子 UTXO 自动撤销 | 自动执行 |
| 使用次数只降不升 | derived.max_uses ≤ parent.max_uses（若非零） | Tx 被拒绝 |

```go
// pkg/caputxo/validate.go

package caputxo

import "fmt"

func ValidateDerivation(parent, child *CapabilityUTXO) error {
    if parent.Status != UTXOStatusUnspent {
        return fmt.Errorf("parent UTXO %s is not UNSPENT", parent.UTXOID)
    }
    if !parent.Delegatable {
        return fmt.Errorf("parent UTXO %s is not delegatable", parent.UTXOID)
    }
    if child.CurDepth != parent.CurDepth+1 {
        return fmt.Errorf("depth mismatch: expected %d, got %d",
            parent.CurDepth+1, child.CurDepth)
    }
    if child.CurDepth > parent.MaxDepth {
        return fmt.Errorf("exceeds max delegation depth %d", parent.MaxDepth)
    }
    if !isSubset(child.Scopes, parent.Scopes) {
        return fmt.Errorf("child scopes not subset of parent scopes")
    }
    if parent.ExpiresAt > 0 && child.ExpiresAt > parent.ExpiresAt {
        return fmt.Errorf("child TTL exceeds parent TTL")
    }
    if parent.MaxUses > 0 && (child.MaxUses == 0 || child.MaxUses > parent.MaxUses) {
        return fmt.Errorf("child max_uses exceeds parent max_uses")
    }
    return nil
}

func isSubset(child, parent []string) bool {
    set := make(map[string]struct{}, len(parent))
    for _, s := range parent {
        set[s] = struct{}{}
    }
    for _, s := range child {
        if _, ok := set[s]; !ok {
            return false
        }
    }
    return true
}
```

## 8.5 UTXO 集状态管理

链上维护两个集合，存储在 IAVL 树中：

```
  IAVL State Tree 中的 Cap-UTXO 存储

  key 前缀: "caputxo/"

  ┌───────────────────────────────────────────────────────┐
  │  UTXO Set (活跃集)                                    │
  │  key: "caputxo/u/{utxo_id}"                          │
  │  val: CapabilityUTXO protobuf bytes                   │
  │                                                       │
  │  按持有者索引                                          │
  │  key: "caputxo/h/{holder_hex}/{utxo_id}"              │
  │  val: [] (存在性索引)                                  │
  │                                                       │
  │  按能力类型索引                                        │
  │  key: "caputxo/t/{cap_type}/{utxo_id}"                │
  │  val: [] (存在性索引)                                  │
  │                                                       │
  │  派生关系索引                                          │
  │  key: "caputxo/d/{parent_utxo_id}/{child_utxo_id}"    │
  │  val: [] (存在性索引)                                  │
  └───────────────────────────────────────────────────────┘

  每个区块 EndBlock 时:
    1. 扫描即将过期的 UTXO → 标记 EXPIRED
    2. 级联撤销已撤销父节点的子 UTXO
    3. 更新 IAVL Commit → 新 app_hash
```

---
