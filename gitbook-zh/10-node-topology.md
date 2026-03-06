# 10. 节点分层与网络拓扑

## 10.1 六级节点体系

AACP 网络由 **T0–T5 六级节点** 组成，每级承担不同职责：

```
  节点层级金字塔

          ┌───────┐
          │  T0   │  Validator（共识核心）
          │ ≤ 21  │  出块 + 验证 + 治理投票
          └───┬───┘
              │
         ┌────┴────┐
         │   T1    │  Full Node（全节点）
         │ 50–200  │  完整状态 + Tx 转发 + RPC
         └────┬────┘
              │
        ┌─────┴─────┐
        │    T2     │  Archive Node（归档节点）
        │  10–50    │  历史状态 + 区块浏览器 + 数据分析
        └─────┬─────┘
              │
       ┌──────┴──────┐
       │     T3      │  Relay Node（中继节点）
       │  100–1000   │  请求路由 + 缓存 + 边缘协调
       └──────┬──────┘
              │
      ┌───────┴───────┐
      │      T4       │  Edge Node（边缘节点）
      │  1000–10000   │  Agent 执行 + 本地推理
      └───────┬───────┘
              │
     ┌────────┴────────┐
     │       T5        │  Micro Node（微节点）
     │  10000+         │  IoT 端侧 + 轻量验证
     └─────────────────┘
```

## 10.2 节点规格对照表

| 维度 | T0 Validator | T1 Full | T2 Archive | T3 Relay | T4 Edge | T5 Micro |
|------|-------------|---------|-----------|---------|---------|---------|
| **角色** | 共识出块 | 状态同步 | 历史存档 | 中继路由 | Agent执行 | 端侧感知 |
| **数量** | ≤ 21 | 50–200 | 10–50 | 100–1K | 1K–10K | 10K+ |
| **CPU** | 16C+ | 8C+ | 8C+ | 4C+ | 2C+ | 1C ARM |
| **RAM** | 64GB+ | 32GB+ | 64GB+ | 8GB+ | 4GB+ | 1GB+ |
| **Storage** | 2TB NVMe | 1TB SSD | 10TB+ HDD | 500GB SSD | 100GB | 32GB |
| **Bandwidth** | 1Gbps | 500Mbps | 200Mbps | 500Mbps | 100Mbps | 10Mbps |
| **保证金** | ¥500,000 | ¥50,000 | ¥20,000 | ¥5,000 | ¥1,000 | ¥0 |
| **佣金份额** | 40% 池 | — | — | 20% 池 | — | — |
| **状态** | 完整 | 完整 | 完整+历史 | 最近100块 | 最近10块 | 仅块头 |
| **ABCI** | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| **P2P 角色** | 种子节点 | 全网 Gossip | 被动同步 | 区域枢纽 | 叶子节点 | 叶子节点 |

## 10.3 节点注册与发现

```protobuf
// aacp.v1.node — 节点注册
message NodeRegistration {
  bytes   pubkey        = 1;  // Ed25519 公钥
  NodeTier tier         = 2;
  string  moniker       = 3;  // 节点昵称
  string  endpoint      = 4;  // 网络地址 "ip:port"
  string  region        = 5;  // 地理区域 "cn-east" | "us-west" | ...
  
  // 硬件自述
  HardwareSpec hw       = 6;
  
  // 保证金证明
  string  deposit_tx_id = 7;
  
  bytes   signature     = 8;
}

enum NodeTier {
  TIER_T0_VALIDATOR = 0;
  TIER_T1_FULL      = 1;
  TIER_T2_ARCHIVE   = 2;
  TIER_T3_RELAY     = 3;
  TIER_T4_EDGE      = 4;
  TIER_T5_MICRO     = 5;
}

message HardwareSpec {
  uint32  cpu_cores     = 1;
  uint64  ram_mb        = 2;
  uint64  storage_gb    = 3;
  uint32  bandwidth_mbps = 4;
  string  arch          = 5;  // "x86_64" | "aarch64"
  string  gpu           = 6;  // 可选
}
```

## 10.4 区域拓扑与路由

```
  跨区域网络拓扑

  ┌──────────────────┐          ┌──────────────────┐
  │   cn-east 区域    │          │   us-west 区域    │
  │                  │          │                  │
  │  T0──T0──T0     │ CometBFT │     T0──T0──T0   │
  │  │   │   │      │◄────────►│     │   │   │    │
  │  T1  T1  T1     │  P2P     │     T1  T1  T1   │
  │  │       │      │          │     │       │    │
  │  T3──────T3     │  Relay   │     T3──────T3   │
  │  │  │  │  │     │◄────────►│     │  │  │  │   │
  │ T4 T4 T4 T4    │          │    T4 T4 T4 T4   │
  │  │     │       │          │     │     │      │
  │ T5 T5 T5      │          │    T5 T5 T5     │
  │                  │          │                  │
  └──────────────────┘          └──────────────────┘

  路由策略:
    1. Consumer 请求 → 最近的 T3 Relay（延迟优先）
    2. T3 检查本区域 T4 Edge 节点是否可服务
    3. 若本区域无匹配 → 跨区域路由到另一 T3
    4. T3 维护服务目录缓存（TTL 60s）
```

## 10.5 T0 Validator 选举

T0 Validator 采用 **保证金加权 + 信誉加权** 的选举机制（非 PoS 代币质押）：

```
  选举评分 = deposit_weight × 0.5 + reputation_weight × 0.3 + uptime_weight × 0.2

  其中:
    deposit_weight    = node.deposit / max_deposit_in_candidates
    reputation_weight = node.reputation / MAX_REPUTATION
    uptime_weight     = node.uptime_30d / 1.0  (30天在线率)

  每个 Epoch（默认 7 天）:
    1. 所有 T1+ 节点可申请成为 T0 候选
    2. 按评分排序取 Top-21
    3. 当前 T0 集合与新集合 diff
    4. 新增节点激活、退出节点进入 unbonding（7天冷却）
```

```go
// pkg/node/election.go

package node

import (
    "math"
    "sort"
)

type Candidate struct {
    Pubkey     [32]byte
    Deposit    float64
    Reputation float64
    Uptime30d  float64
}

type ElectionResult struct {
    Validators []Candidate
    Epoch      uint64
}

func ElectValidators(candidates []Candidate, maxValidators int) ElectionResult {
    if len(candidates) == 0 {
        return ElectionResult{}
    }

    maxDeposit := 0.0
    for _, c := range candidates {
        maxDeposit = math.Max(maxDeposit, c.Deposit)
    }

    type scored struct {
        Candidate
        Score float64
    }

    scoredList := make([]scored, len(candidates))
    for i, c := range candidates {
        dw := c.Deposit / maxDeposit
        rw := c.Reputation / 1000.0
        uw := c.Uptime30d
        scoredList[i] = scored{c, dw*0.5 + rw*0.3 + uw*0.2}
    }

    sort.Slice(scoredList, func(i, j int) bool {
        return scoredList[i].Score > scoredList[j].Score
    })

    n := maxValidators
    if n > len(scoredList) {
        n = len(scoredList)
    }

    result := ElectionResult{Validators: make([]Candidate, n)}
    for i := 0; i < n; i++ {
        result.Validators[i] = scoredList[i].Candidate
    }
    return result
}
```

## 10.6 节点配置示例

```toml
# config/node.toml — T3 Relay 节点配置示例

[node]
moniker  = "relay-cn-east-01"
tier     = "T3_RELAY"
region   = "cn-east"

[network]
listen_addr = "0.0.0.0:26656"
rpc_addr    = "0.0.0.0:26657"
grpc_addr   = "0.0.0.0:9090"
seeds       = "t0-seed-1@10.0.1.1:26656,t0-seed-2@10.0.1.2:26656"
max_peers   = 200

[relay]
cache_ttl_sec        = 60
max_cached_listings  = 10000
cross_region_enabled = true
upstream_relays      = ["relay-us-west-01@us-west.aacp.network:26656"]

[hardware]
cpu_cores     = 4
ram_mb        = 8192
storage_gb    = 500
bandwidth_mbps = 500

[deposit]
currency  = "CNY"
amount    = "5000.00"
tx_id     = "fiat_deposit_tx_abc123"
```

---
