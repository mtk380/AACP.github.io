# 5. AMX：Agent 交易市场

## 5.1 AMX 定位

AMX（Agent Marketplace Exchange）是 AACP 协议的商业入口——所有 Agent 服务的 **发布、发现、定价、撮合、结算** 都通过 AMX 完成。

```
┌────────────────────────────────────────────────────────────────┐
│                      AMX 功能全景                               │
│                                                                │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│   │ 服务挂单  │   │ 搜索发现  │   │ 撮合引擎  │   │ 订单管理  │  │
│   │ Listing  │   │ Discovery│   │ Matching │   │  Order   │  │
│   └─────┬────┘   └─────┬────┘   └─────┬────┘   └─────┬────┘  │
│         │              │              │              │        │
│         └──────────────┴──────┬───────┴──────────────┘        │
│                               │                               │
│                        ┌──────┴──────┐                        │
│                        │  AMX State  │                        │
│                        │  Machine    │                        │
│                        └──────┬──────┘                        │
│                               │                               │
│              ┌────────────────┼────────────────┐              │
│              ▼                ▼                ▼              │
│        ┌──────────┐   ┌──────────┐   ┌──────────────┐        │
│        │ Cap-UTXO │   │ 信誉索引  │   │ 法币 Escrow  │        │
│        │ 能力验证  │   │ 排序加权  │   │ 资金托管     │        │
│        └──────────┘   └──────────┘   └──────────────┘        │
└────────────────────────────────────────────────────────────────┘
```

## 5.2 服务挂单（Listing）

Provider 将 Agent 能力注册到链上，形成可搜索、可撮合的 **Listing**。

```protobuf
// aacp.v1.amx — 服务挂单
message Listing {
  string  listing_id    = 1;  // 全局唯一, SHA-256(provider+nonce)[:16]
  bytes   provider      = 2;  // Provider Ed25519 公钥
  string  title         = 3;  // 人类可读标题, ≤128 chars
  string  description   = 4;  // Markdown 描述, ≤4096 chars

  // 能力声明
  repeated CapabilityRef capabilities = 5;

  // 定价
  PricingModel pricing = 6;

  // SLA 承诺
  SLATemplate sla = 7;

  // 状态
  ListingStatus status = 8;
  int64  created_at     = 9;  // Unix timestamp
  int64  updated_at     = 10;
}

message PricingModel {
  string currency   = 1;  // "CNY" | "USD"
  string base_price = 2;  // 基础价格, decimal string
  string unit       = 3;  // "per_call" | "per_hour" | "per_task" | "per_token"

  // 阶梯定价（可选）
  repeated PriceTier tiers = 4;
}

message PriceTier {
  uint64 min_quantity = 1;
  uint64 max_quantity = 2;
  string unit_price   = 3;  // decimal string
}

enum ListingStatus {
  LISTING_ACTIVE      = 0;
  LISTING_PAUSED      = 1;
  LISTING_SUSPENDED   = 2;  // 被信誉系统降权
  LISTING_DELISTED    = 3;
}
```

## 5.3 撮合引擎

AMX 撮合采用 **两阶段提交**：链上确定性排序 + 链下索引加速。

```
  撮合流程
  
  Consumer                 AMX Engine                Provider
     │                        │                        │
     │── 1. SubmitRequest ───→│                        │
     │   (需求+预算+Cap要求)   │                        │
     │                        │── 2. Query Index ──→   │
     │                        │   (链下倒排索引)        │
     │                        │←── Candidate List ──   │
     │                        │                        │
     │                        │── 3. Score & Rank ──   │
     │                        │   score = f(price,     │
     │                        │     reputation,        │
     │                        │     latency, cap_match)│
     │                        │                        │
     │                        │── 4. Match (链上Tx) ──→│
     │                        │                        │
     │←── 5. MatchResult ────│                        │
     │   (order_id, provider, │                        │
     │    agreed_price,       │                        │
     │    escrow_amount)      │                        │
     │                        │                        │
     │──────── 6. Escrow Deposit (法币) ──────────────→│
     │                        │                        │
     │←── 7. OrderConfirmed ─│──→ OrderConfirmed ────→│
     │                        │                        │
```

**撮合评分公式**：

```
Score(listing, request) = w₁·PriceScore + w₂·RepScore + w₃·LatencyScore + w₄·CapScore

其中：
  PriceScore   = 1 - (listing.price - request.min_price) / (request.max_price - request.min_price)
  RepScore     = listing.provider.reputation / MAX_REPUTATION
  LatencyScore = 1 - (estimated_latency / request.max_latency)
  CapScore     = |matched_caps| / |required_caps|

默认权重：w₁=0.30, w₂=0.30, w₃=0.20, w₄=0.20
权重可由 Consumer 在 Request 中自定义覆盖。
```

## 5.4 订单状态机

```
                         AMX 订单状态机

  ┌──────────┐   match    ┌──────────┐  escrow_ok  ┌──────────┐
  │ MATCHING │──────────→│ MATCHED  │───────────→│ ESCROWED │
  └──────────┘           └──────────┘            └─────┬────┘
       │                      │                        │
       │ timeout/             │ reject                 │ handoff_to_aap
       │ no_match             │                        │
       ▼                      ▼                        ▼
  ┌──────────┐          ┌──────────┐           ┌──────────┐
  │  FAILED  │          │ REJECTED │           │ ACTIVE   │──→ AAP 接管
  └──────────┘          └──────────┘           └─────┬────┘
                                                     │
                                          ┌──────────┼──────────┐
                                          ▼          ▼          ▼
                                    ┌──────────┐┌────────┐┌──────────┐
                                    │COMPLETED ││DISPUTED││ CANCELED │
                                    └──────────┘└────────┘└──────────┘
```

| 状态 | 触发条件 | 下一步 |
|------|---------|--------|
| `MATCHING` | Consumer 提交 Request | 撮合引擎运行 |
| `MATCHED` | 撮合成功，等待双方确认 | Provider 确认 → ESCROWED |
| `ESCROWED` | Consumer 法币托管到位 | 生成 AAP 任务 → ACTIVE |
| `ACTIVE` | AAP 任务执行中 | 完成/争议/取消 |
| `COMPLETED` | AAP 任务成功结算 | 资金释放，信誉更新 |
| `DISPUTED` | 任一方发起争议 | 进入仲裁流程 |
| `CANCELED` | 超时/双方同意取消 | 退还托管资金 |
| `FAILED` | 撮合超时或无匹配 | 退还 gas |
| `REJECTED` | Provider 拒绝撮合结果 | 重新撮合或退还 |

## 5.5 AMX Go 接口

```go
// pkg/amx/engine.go

package amx

import "context"

type Engine interface {
    // 服务挂单
    CreateListing(ctx context.Context, req *CreateListingRequest) (*Listing, error)
    UpdateListing(ctx context.Context, req *UpdateListingRequest) (*Listing, error)
    PauseListing(ctx context.Context, listingID string) error
    DelistListing(ctx context.Context, listingID string) error

    // 搜索发现
    SearchListings(ctx context.Context, query *SearchQuery) (*SearchResult, error)

    // 撮合
    SubmitRequest(ctx context.Context, req *MatchRequest) (*MatchResult, error)
    ConfirmMatch(ctx context.Context, orderID string, side Side) error
    RejectMatch(ctx context.Context, orderID string, side Side, reason string) error

    // 订单管理
    GetOrder(ctx context.Context, orderID string) (*Order, error)
    CancelOrder(ctx context.Context, orderID string, reason string) error
    ListOrders(ctx context.Context, filter *OrderFilter) ([]*Order, error)
}

type Side int
const (
    SideConsumer Side = iota
    SideProvider
)

type MatchRequest struct {
    ConsumerPubKey []byte
    RequiredCaps   []string            // Capability ID 列表
    Budget         Money               // 预算上限
    MaxLatencyMs   uint32              // 最大可接受延迟
    Weights        *ScoringWeights     // 可选，覆盖默认权重
    Deadline       int64               // 撮合截止时间
}

type Money struct {
    Currency string  // "CNY" | "USD"
    Amount   string  // decimal string, e.g. "99.50"
}

type ScoringWeights struct {
    Price   float64 // 默认 0.30
    Rep     float64 // 默认 0.30
    Latency float64 // 默认 0.20
    Cap     float64 // 默认 0.20
}
```

---
