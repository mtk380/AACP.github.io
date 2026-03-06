# 15. 治理与演进

## 15.1 治理原则

```
┌──────────────────────────────────────────────────────────────┐
│                   AACP 治理架构                                │
│                                                              │
│  无代币治理 ─── 治理权来自:                                    │
│    1. 保证金权重 (法币保证金越高，投票权重越大)                  │
│    2. 信誉权重 (信誉越高，投票权重越大)                         │
│    3. 节点等级 (T0 > T1 > T2 > ... )                         │
│                                                              │
│  治理权重 = deposit_norm × 0.4                                │
│           + reputation_norm × 0.4                             │
│           + tier_weight × 0.2                                 │
│                                                              │
│  tier_weight: T0=1.0, T1=0.7, T2=0.5, T3=0.3, T4=0.1       │
└──────────────────────────────────────────────────────────────┘
```

## 15.2 提案类型

| 类型 | 代号 | 通过条件 | 投票期 | 示例 |
|------|------|---------|--------|------|
| 参数调整 | `PARAM` | 50%+1 投票权重 | 7 天 | 佣金率区间、心跳间隔 |
| 协议升级 | `UPGRADE` | 66%+1 投票权重 | 14 天 | ABCI 版本、新模块上线 |
| 紧急修复 | `EMERGENCY` | 80%+1 投票权重 | 48 小时 | 安全漏洞热修复 |
| 节点准入 | `ADMISSION` | 50%+1 投票权重 | 7 天 | 新 T0 候选、节点除名 |
| 保险池动用 | `INSURANCE` | 66%+1 投票权重 | 7 天 | 超大额赔付、系统补偿 |

```protobuf
// aacp.v1.gov — 治理提案
message Proposal {
  string   proposal_id   = 1;
  string   proposer      = 2;    // 提案者地址
  ProposalType type      = 3;
  string   title         = 4;
  string   description   = 5;    // Markdown
  
  // 参数变更 (PARAM 类型)
  repeated ParamChange param_changes = 6;
  
  // 升级计划 (UPGRADE 类型)
  UpgradePlan upgrade_plan = 7;
  
  // 投票
  ProposalStatus status  = 8;
  int64  voting_start    = 9;
  int64  voting_end      = 10;
  string yes_weight      = 11;   // 赞成权重总和
  string no_weight       = 12;
  string abstain_weight  = 13;
}

message ParamChange {
  string module    = 1;  // "amx" | "aap" | "rep" | "fiat" | ...
  string key       = 2;  // "commission_min_bps" | "heartbeat_interval_sec" | ...
  string old_value = 3;
  string new_value = 4;
}

enum ProposalType {
  PROPOSAL_PARAM     = 0;
  PROPOSAL_UPGRADE   = 1;
  PROPOSAL_EMERGENCY = 2;
  PROPOSAL_ADMISSION = 3;
  PROPOSAL_INSURANCE = 4;
}

enum ProposalStatus {
  STATUS_VOTING    = 0;
  STATUS_PASSED    = 1;
  STATUS_REJECTED  = 2;
  STATUS_EXECUTED  = 3;
}
```

## 15.3 链上可治理参数

| 模块 | 参数 | 当前默认值 | 允许范围 |
|------|------|----------|---------|
| AMX | `commission_min_bps` | 800 (8%) | 500–1000 |
| AMX | `commission_max_bps` | 1500 (15%) | 1000–2000 |
| AMX | `match_timeout_sec` | 300 | 60–600 |
| AAP | `heartbeat_interval_sec` | 30 | 10–120 |
| AAP | `max_retries` | 3 | 1–10 |
| AAP | `verification_timeout_sec` | 600 | 120–3600 |
| REP | `decay_epochs` | 4 | 2–12 |
| REP | `decay_rate_bps` | 200 (2%) | 50–500 |
| ARB | `auto_resolve_timeout_sec` | 600 | 120–1800 |
| ARB | `committee_size` | 5 | 3–11 |
| ARB | `full_review_timeout_hr` | 72 | 24–168 |
| FIAT | `insurance_safety_ratio` | 200 (200%) | 150–500 |
| NODE | `max_validators` | 21 | 7–51 |
| NODE | `epoch_duration_days` | 7 | 1–30 |
| NODE | `unbonding_days` | 7 | 3–21 |

---
