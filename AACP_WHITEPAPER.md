# AACP 白皮书

## Agent Accountability & Coordination Protocol

> **版本** 0.9.0-draft · **日期** 2026-03-01 · **状态** Internal Review

---

## 目录

| # | 章节 | 关键词 |
|---|------|--------|
| 1 | [执行摘要](#1-执行摘要) | 一句话定位、核心指标 |
| 2 | [问题与机遇](#2-问题与机遇) | 现状痛点、市场空间 |
| 3 | [设计哲学：三大原则](#3-设计哲学三大原则) | Marketplace-First / Fiat-Native / Edge-First |
| 4 | [协议架构总览](#4-协议架构总览) | 分层图、模块拓扑 |
| 5 | [AMX：Agent 交易市场](#5-amxagent-交易市场) | 挂单 / 撮合 / 结算 |
| 6 | [AAP：行动问责协议](#6-aap行动问责协议) | 任务生命周期、证据链 |
| 7 | [WEAVE：多 Agent 协调引擎](#7-weave多-agent-协调引擎) | DAG 编排、子任务拆分 |
| 8 | [Capability-UTXO 模型](#8-capability-utxo-模型) | 能力令牌、权限传递 |
| 9 | [法币原生经济模型](#9-法币原生经济模型) | CNY/USD 结算、佣金分配 |
| 10 | [节点分层与网络拓扑](#10-节点分层与网络拓扑) | T0–T5 六级节点 |
| 11 | [信誉系统](#11-信誉系统) | 多维评分、衰减机制 |
| 12 | [争议仲裁与反欺诈](#12-争议仲裁与反欺诈) | 三阶段仲裁、异常检测 |
| 13 | [安全架构](#13-安全架构) | 密码学、威胁模型 |
| 14 | [技术栈与实现路线](#14-技术栈与实现路线) | Go / CometBFT / IAVL |
| 15 | [治理与演进](#15-治理与演进) | 链上提案、参数治理 |
| 16 | [路线图](#16-路线图) | 里程碑 |
| 17 | [结语](#17-结语) | 愿景 |
| A | [附录 A：白皮书自检报告](#附录-a白皮书自检报告) | 术语/数值/状态机一致性 |
| B | [附录 B：Nano Babana 生图提示词](#附录-bnano-babana-生图提示词) | 关键内容视觉化提示词 |

---

# 1. 执行摘要

**AACP（Agent Accountability & Coordination Protocol）** 是一个面向 AI Agent 经济的去中心化协议，提供 **Agent 交易市场** 与 **去中心化责任网络** 双重基础设施。

```
┌─────────────────────────────────────────────────────────┐
│                   AACP 一句话定位                         │
│                                                         │
│  "让 AI Agent 像 SaaS 一样被定价、交易、问责，           │
│   同时用法币而非代币完成端到端结算。"                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 核心设计决策

| 决策维度 | AACP 选择 | 行业惯例 | 理由 |
|---------|-----------|---------|------|
| 结算货币 | **法币原生**（CNY/USD） | 平台代币 | 降低 Agent 消费者准入门槛，规避监管灰区 |
| 市场模式 | **双边交易市场** | 封闭 API 网关 | Agent 是生产要素，需要价格发现机制 |
| 执行位置 | **边缘优先** | 云中心化 | Agent 任务延迟敏感，靠近数据源执行 |
| 问责机制 | **链上证据 + 法币保证金** | SLA 合同 | 自动化执行，降低信任成本 |
| 代币 | **无平台代币** | ICO/治理代币 | 避免投机干扰协议治理 |

### 关键指标目标（Mainnet v1）

| 指标 | 目标值 |
|------|-------|
| 交易确认延迟（Finality） | ≤ 3s |
| 峰值 TPS（AMX 撮合） | ≥ 2,000 |
| 节点参与门槛（T5 Micro） | 树莓派 4B / 4GB RAM |
| 争议仲裁周期 | ≤ 72h |
| 平台佣金区间 | 8%–15% |
| 年化保险池覆盖率 | ≥ 200% 历史最大单笔赔付 |

---

# 2. 问题与机遇

## 2.1 AI Agent 经济的结构性缺陷

2025–2026 年，AI Agent 从实验室走向生产环境，但基础设施严重滞后：

```
                    AI Agent 经济断裂带
                    
  Agent 提供方                                Agent 消费方
  ┌──────────┐                              ┌──────────┐
  │ 独立开发者 │──── ??? ────────────────────│ 企业客户  │
  │ AI 团队   │     没有标准市场              │ 个人用户  │
  │ 开源项目  │     没有定价机制              │ 其他Agent │
  └──────────┘     没有质量保证              └──────────┘
                   没有责任追溯
                   没有跨Agent协作标准
```

### 五大痛点量化

| # | 痛点 | 现状 | 影响 |
|---|------|------|------|
| P1 | **发现困难** | Agent 分散在 GitHub/HuggingFace/私有 API | 消费者找不到合适 Agent，供给方无法触达市场 |
| P2 | **定价不透明** | 按 API 调用计费，无法反映真实价值 | 高质量 Agent 被低估，低质量 Agent 充斥 |
| P3 | **责任真空** | Agent 执行结果无法审计和追溯 | 出错后无法归因，多 Agent 协作时责任链断裂 |
| P4 | **结算摩擦** | 加密货币结算门槛高，法币通道缺失 | 95% 企业无法或不愿使用代币支付 |
| P5 | **协作孤岛** | 各平台 Agent 无法互操作 | 复杂任务需要人工编排，成本线性增长 |

## 2.2 市场规模预测

```
  Agent 服务市场规模（预测，十亿美元）
  
  2025  ████░░░░░░░░░░░░░░░░░░░░  $12B
  2026  ████████░░░░░░░░░░░░░░░░  $28B
  2027  ████████████░░░░░░░░░░░░  $55B
  2028  ████████████████████░░░░  $110B
  
  AACP 可切入的交易撮合 + 协调层 ≈ 总市场的 5%–8%
```

## 2.3 为什么是现在

三个技术条件在 2026 年同时成熟：

1. **模型能力跃迁**：GPT-5 / Claude-4 级模型让 Agent 能完成端到端复杂任务
2. **BFT 共识性能突破**：CometBFT 单链 10k+ TPS 使链上撮合成为可能
3. **边缘计算普及**：ARM 芯片 + 端侧推理让 T4/T5 节点经济可行

---

# 3. 设计哲学：三大原则

## 3.1 Marketplace-First（市场优先）

> Agent 经济需要的不是另一个 API 网关，而是一个 **真正的双边市场**。

```
  传统模式                          AACP 模式（Marketplace-First）
  
  Provider ──→ API Gateway ──→ Consumer     Provider ──┐
  Provider ──→ API Gateway ──→ Consumer              ┌─┴──────────┐
  Provider ──→ API Gateway ──→ Consumer              │    AMX      │
                                                     │  双边市场   │──→ Consumer
  ✗ 中心化定价                                        │  价格发现   │──→ Consumer
  ✗ 无竞争机制                                        │  质量排序   │──→ Consumer
  ✗ 平台锁定                                         └─┬──────────┘
                                             Provider ──┘
                                            
                                             ✓ 供需定价
                                             ✓ 声誉竞争
                                             ✓ 零切换成本
```

**核心设计**：

- AMX（Agent Marketplace Exchange）为 Agent 服务提供标准化挂单、撮合、结算
- 价格由供需双方在链上竞价产生，不由平台方设定
- 每个 Agent 的能力（Capability）以 UTXO 形式确权，可被发现、组合、交易

## 3.2 Fiat-Native（法币原生）

> **无平台代币**。结算层直接对接法币（CNY / USD），佣金以法币计价和清算。

| 维度 | 代币模式问题 | AACP 法币原生方案 |
|------|-------------|------------------|
| 准入 | 需要购买代币，KYC + 交易所 | 银行账户 / 支付宝 / Stripe 直接支付 |
| 波动 | 代币价格波动导致服务定价不稳 | 法币定价，稳定可预期 |
| 合规 | 多数司法管辖区存在监管风险 | 法币结算天然合规 |
| 激励 | 代币激励吸引投机者而非真实用户 | 佣金分配激励真实参与者 |

**佣金机制概览**：

```
  Agent 交易总价 = 服务费 + 平台佣金（8%–15%）
  
  ┌───────────────────────────────────────┐
  │          佣金分配（每笔交易）           │
  │                                       │
  │   验证者（T0 Validator）  ████████ 40% │
  │   中继节点（T3 Relay）    ████     20% │
  │   保险池（Insurance Pool） ████     20% │
  │   平台运营（Operations）   ████     20% │
  │                                       │
  │   合计                          = 100% │
  └───────────────────────────────────────┘
```

## 3.3 Edge-First（边缘优先）

> Agent 执行应靠近数据源和用户，而非集中在云端数据中心。

```
  ┌─────────────────────────────────────────────────┐
  │                网络拓扑示意                       │
  │                                                 │
  │           ┌─────┐    ┌─────┐                    │
  │           │ T0  │────│ T0  │   共识核心          │
  │           │Valid│    │Valid│   （≤21 节点）       │
  │           └──┬──┘    └──┬──┘                    │
  │              │          │                       │
  │         ┌────┴───┐ ┌───┴────┐                   │
  │         │T1 Full │ │T1 Full │  全节点层          │
  │         └───┬────┘ └───┬────┘                   │
  │             │          │                        │
  │      ┌──┬──┴──┬──┐  ┌─┴──┬──┐                  │
  │      │T3│  T3 │T3│  │ T3 │T3│  中继层           │
  │      └┬─┘  └┬─┘└┬┘  └─┬──┘└┬┘                  │
  │       │     │   │     │    │                    │
  │    ┌──┴┐ ┌─┴┐ ┌┴──┐ ┌┴─┐ ┌┴──┐                │
  │    │T4 │ │T4│ │T4 │ │T4│ │T4 │ 边缘执行层      │
  │    │Edg│ │  │ │Edg│ │  │ │Edg│                  │
  │    └┬──┘ └──┘ └┬──┘ └──┘ └┬──┘                  │
  │     │          │          │                     │
  │   ┌─┴──┐   ┌──┴─┐    ┌──┴─┐                    │
  │   │ T5 │   │ T5 │    │ T5 │  微节点             │
  │   │Micr│   │Micr│    │Micr│  （IoT/手机）       │
  │   └────┘   └────┘    └────┘                     │
  └─────────────────────────────────────────────────┘
```

**边缘优先的量化收益**：

| 场景 | 云中心化延迟 | Edge-First 延迟 | 改善 |
|------|------------|----------------|------|
| 本地文件处理 Agent | 200–500ms（上传+处理+下载） | 10–50ms（本地执行） | 4–50x |
| IoT 数据分析 Agent | 300–800ms | 5–20ms | 15–160x |
| 实时对话 Agent | 150–300ms | 30–80ms | 2–10x |
| 多 Agent 协作链 | N × 200ms | N × 20ms | ~10x |

---

# 4. 协议架构总览

## 4.1 四层协议栈

```
┌─────────────────────────────────────────────────────────────────┐
│                       应用层 Application                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │ Agent SDK│ │ 开发者CLI │ │ Dashboard│ │ 第三方 DApp 集成   │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                       协议层 Protocol                           │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────────┐ │
│  │  AMX   │ │  AAP   │ │ WEAVE  │ │Cap-UTXO│ │ 信誉 / 仲裁  │ │
│  │ 市场   │ │ 行动   │ │ 协调   │ │ 能力   │ │ / 反欺诈     │ │
│  └────┬───┘ └───┬────┘ └───┬────┘ └───┬────┘ └──────┬───────┘ │
│       └─────────┴──────────┴──────────┴──────────────┘         │
├─────────────────────────────────────────────────────────────────┤
│                       共识层 Consensus                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │        CometBFT + ABCI 2.0 + IAVL State Tree            │  │
│  │  ┌──────────┐ ┌────────────┐ ┌─────────────────────┐    │  │
│  │  │ 区块生产  │ │ 状态机执行  │ │ 快照 / 状态同步     │    │  │
│  │  └──────────┘ └────────────┘ └─────────────────────┘    │  │
│  └──────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                       网络层 Network                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │ P2P 八卦  │ │ Relay 中继│ │ IPFS 存储│ │ 法币网关桥接    │  │
│  │ libp2p   │ │ Protocol │ │ 大对象   │ │ Escrow Bridge   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 4.2 核心模块关系

```
  ┌──────────────────────────────────────────────────┐
  │                 交易生命周期                       │
  │                                                  │
  │  [消费者] ──发布需求──→ [AMX] ──撮合──→ [供给方]  │
  │     │                   │                  │     │
  │     │              创建订单                 │     │
  │     │                   │                  │     │
  │     │                   ▼                  │     │
  │     │              [AAP 行动协议]           │     │
  │     │              ┌──────────┐            │     │
  │     │              │ 签署SLA  │            │     │
  │     │              │ 缴纳保证金│            │     │
  │     │              │ 发放Cap  │            │     │
  │     │              └────┬─────┘            │     │
  │     │                   │                  │     │
  │     │        ┌──────────┼──────────┐       │     │
  │     │        ▼          ▼          ▼       │     │
  │     │   [Cap-UTXO]  [WEAVE]   [执行层]    │     │
  │     │    权限令牌    任务编排   Agent运行    │     │
  │     │        │          │          │       │     │
  │     │        └──────────┼──────────┘       │     │
  │     │                   ▼                  │     │
  │     │         [结果提交 + 证据上链]          │     │
  │     │                   │                  │     │
  │     │        ┌──────────┼──────────┐       │     │
  │     │        ▼          ▼          ▼       │     │
  │     │    [信誉更新]  [法币结算]  [争议?]    │     │
  │     │                                │     │     │
  │     │                          ┌─────┘     │     │
  │     │                          ▼           │     │
  │     │                     [仲裁流程]        │     │
  │     │                     [反欺诈检测]      │     │
  │     │                          │           │     │
  │     │                          ▼           │     │
  │     │                     [赔付/罚没]       │     │
  │     │                                      │     │
  └──────────────────────────────────────────────────┘
```

## 4.3 模块职责速查

| 模块 | 缩写 | 核心职责 | 依赖 | 链上/链下 |
|------|------|---------|------|----------|
| Agent Marketplace Exchange | **AMX** | 服务挂单、搜索、撮合、订单管理 | Cap-UTXO, 信誉 | 链上撮合 + 链下索引 |
| Agent Action Protocol | **AAP** | 任务生命周期、SLA 签署、证据链、结算触发 | AMX, Cap-UTXO, WEAVE | 链上状态机 |
| WEAVE | **WEAVE** | 多 Agent DAG 编排、子任务拆分、并行调度 | AAP, Cap-UTXO | 链上 DAG + 链下执行 |
| Capability-UTXO | **Cap-UTXO** | Agent 能力确权、权限授予/撤销/传递 | — | 链上 UTXO |
| 信誉系统 | **REP** | 多维评分、信誉衰减、恶意行为惩罚 | AAP | 链上评分状态 |
| 争议仲裁 | **ARB** | 三阶段仲裁、证据审查、赔付执行 | AAP, REP, 保险池 | 链上仲裁状态机 |
| 反欺诈 | **AFD** | 异常检测、Sybil 防御、作弊识别 | REP, AMX | 链下检测 + 链上处罚 |
| 法币结算 | **FIAT** | 托管、结算、保证金、退款 | AAP, ARB | 链上记录 + 链下清算 |

## 4.4 核心 Protobuf 命名空间

```protobuf
// AACP 顶层命名空间
syntax = "proto3";
package aacp.v1;

// 各模块子命名空间
// aacp.v1.amx      — 市场交易
// aacp.v1.aap      — 行动协议
// aacp.v1.weave    — 多Agent协调
// aacp.v1.caputxo  — 能力UTXO
// aacp.v1.rep      — 信誉系统
// aacp.v1.arb      — 争议仲裁
// aacp.v1.afd      — 反欺诈
// aacp.v1.fiat     — 法币结算
// aacp.v1.node     — 节点管理
// aacp.v1.gov      — 治理

// 统一消息信封
message TxEnvelope {
  bytes   sender     = 1;  // Ed25519 公钥, 32 bytes
  uint64  nonce      = 2;
  uint64  gas_limit  = 3;
  string  module     = 4;  // "amx" | "aap" | "weave" | ...
  string  action     = 5;  // "create_listing" | "accept_task" | ...
  bytes   payload    = 6;  // 各模块 Protobuf 序列化
  bytes   signature  = 7;  // Ed25519 签名, 64 bytes
}
```

## 4.5 技术栈速览

| 层 | 技术选型 | 版本 | 选型理由 |
|----|---------|------|---------|
| 语言 | **Go** | 1.23+ | 高并发、编译部署简单、区块链生态成熟 |
| 共识 | **CometBFT** | 1.x | BFT 即时终局、成熟生产验证 |
| 应用接口 | **ABCI 2.0** | — | 共识与应用解耦、支持 FinalizeBlock |
| 状态树 | **IAVL** | v2 | Merkle 证明、快照、版本化状态 |
| 大对象存储 | **IPFS** | — | 去中心化、内容寻址、证据存档 |
| 序列化 | **Protobuf** | proto3 | 高效编解码、跨语言兼容 |
| 签名 | **Ed25519** | — | 高性能、短签名、广泛支持 |
| 哈希 | **SHA-256** | — | 安全、区块链标准 |
| 对称加密 | **AES-GCM** | 256-bit | 认证加密、硬件加速 |
| 密钥分割 | **Shamir SSS** | (3,5) 默认 | 密钥恢复、门限安全 |
| P2P | **libp2p** | — | 模块化、NAT 穿透、多传输协议 |

---

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

# 7. WEAVE：多 Agent 协调引擎

## 7.1 WEAVE 定位

WEAVE（Workflow Engine for Agent Versatile Execution）解决 **复杂任务需要多个 Agent 协作** 的场景。它将一个顶层任务拆分为 DAG（有向无环图），每个节点对应一个子任务，由不同 Agent 独立执行。

```
  WEAVE 核心概念

  ┌─────────────────────────────────────────────────────────┐
  │                     WEAVE DAG                           │
  │                                                         │
  │              ┌──────────┐                               │
  │              │ 顶层任务  │                               │
  │              │ (Root)   │                               │
  │              └────┬─────┘                               │
  │           ┌───────┼───────┐                             │
  │           ▼       ▼       ▼                             │
  │      ┌────────┐┌────────┐┌────────┐                    │
  │      │ Sub-A  ││ Sub-B  ││ Sub-C  │   可并行            │
  │      │Agent-1 ││Agent-2 ││Agent-3 │                    │
  │      └───┬────┘└───┬────┘└───┬────┘                    │
  │          │         │         │                         │
  │          └─────────┼─────────┘                         │
  │                    ▼                                    │
  │              ┌────────┐                                 │
  │              │ Sub-D  │   依赖 A+B+C 的输出             │
  │              │Agent-4 │                                 │
  │              └───┬────┘                                 │
  │                  ▼                                      │
  │              ┌────────┐                                 │
  │              │ Merge  │   聚合最终结果                    │
  │              │ (Root) │                                 │
  │              └────────┘                                 │
  └─────────────────────────────────────────────────────────┘
```

## 7.2 DAG 定义

```protobuf
// aacp.v1.weave — DAG 定义
message TaskDAG {
  string dag_id     = 1;   // 全局唯一
  string root_task  = 2;   // 关联的 AAP task_id
  bytes  orchestrator = 3; // 编排者公钥

  repeated DAGNode  nodes = 4;
  repeated DAGEdge  edges = 5;

  DAGStatus status    = 6;
  int64     created_at = 7;
}

message DAGNode {
  string node_id       = 1;
  string label         = 2;   // 人类可读描述
  NodeType type        = 3;

  // 子任务绑定
  string sub_task_id   = 4;   // 关联的 AAP task_id（执行时填充）
  string listing_id    = 5;   // 指定 Agent 或留空让 WEAVE 自动撮合

  // 能力要求
  repeated string required_caps = 6;

  // 输入/输出 schema（JSON Schema）
  string input_schema  = 7;
  string output_schema = 8;

  NodeStatus status    = 9;
}

message DAGEdge {
  string from_node = 1;
  string to_node   = 2;
  string condition  = 3;  // 可选: CEL 表达式, 如 "output.score > 0.8"
}

enum NodeType {
  NODE_TASK     = 0;  // 普通任务节点
  NODE_FORK     = 1;  // 分叉（并行）
  NODE_JOIN     = 2;  // 汇合（等待所有前驱）
  NODE_COND     = 3;  // 条件分支
  NODE_LOOP     = 4;  // 循环（带终止条件）
}

enum NodeStatus {
  NODE_PENDING    = 0;
  NODE_READY      = 1;  // 所有前驱完成
  NODE_RUNNING    = 2;
  NODE_COMPLETED  = 3;
  NODE_FAILED     = 4;
  NODE_SKIPPED    = 5;  // 条件分支跳过
}

enum DAGStatus {
  DAG_PENDING    = 0;
  DAG_RUNNING    = 1;
  DAG_COMPLETED  = 2;
  DAG_FAILED     = 3;
  DAG_CANCELED   = 4;
}
```

## 7.3 调度算法

WEAVE 调度器执行 **拓扑排序 + 就绪队列** 策略：

```
  WEAVE 调度循环

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  while DAG.status == RUNNING:                       │
  │    │                                                │
  │    ├─ 1. 扫描所有 PENDING 节点                       │
  │    │     if 所有前驱 edge 的 from_node.status ==     │
  │    │        COMPLETED (且 condition 满足):           │
  │    │       → 标记为 READY                            │
  │    │                                                │
  │    ├─ 2. 对每个 READY 节点:                          │
  │    │     if node.listing_id 非空:                    │
  │    │       → 直接创建 AAP 子任务                      │
  │    │     else:                                      │
  │    │       → 调用 AMX 撮合, 选择最优 Agent           │
  │    │     标记为 RUNNING                              │
  │    │                                                │
  │    ├─ 3. 监听子任务完成事件:                          │
  │    │     on SubTaskCompleted(node_id):               │
  │    │       → 标记为 COMPLETED                        │
  │    │       → 将 output 传播给下游节点的 input         │
  │    │     on SubTaskFailed(node_id):                  │
  │    │       → 根据重试策略决定 RETRY 或 FAILED         │
  │    │       → 若 FAILED, 检查是否有备选路径            │
  │    │                                                │
  │    └─ 4. 终止检测:                                   │
  │         if 所有叶子节点 COMPLETED → DAG COMPLETED     │
  │         if 任何关键路径节点 FAILED → DAG FAILED       │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

## 7.4 数据传递

节点之间通过 **DataRef** 传递中间结果，小数据直接上链，大数据存 IPFS：

```go
// pkg/weave/dataref.go

package weave

const InlineDataMaxBytes = 4096

type DataRef struct {
    // 二选一
    Inline []byte `json:"inline,omitempty"` // ≤ 4KB 直接内联
    IPFSCID string `json:"ipfs_cid,omitempty"` // > 4KB 存 IPFS

    ContentHash [32]byte `json:"content_hash"` // SHA-256, 始终填充
    SizeBytes   uint64   `json:"size_bytes"`
    MimeType    string   `json:"mime_type"`
    Encrypted   bool     `json:"encrypted"`      // 是否 AES-GCM 加密
}

func NewDataRef(data []byte, mimeType string) *DataRef {
    ref := &DataRef{
        ContentHash: sha256Sum(data),
        SizeBytes:   uint64(len(data)),
        MimeType:    mimeType,
    }
    if len(data) <= InlineDataMaxBytes {
        ref.Inline = data
    }
    return ref
}
```

## 7.5 实际案例：电商商品上架 Agent 协作

```
  场景：商家上传一张商品图，WEAVE 编排 4 个 Agent 完成上架

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  [Input: 商品图片]                                   │
  │         │                                           │
  │         ▼                                           │
  │  ┌──────────────┐                                   │
  │  │ A: 图像识别   │  Agent-Vision                     │
  │  │ 提取商品属性  │  Cap: image_recognition            │
  │  └──────┬───────┘                                   │
  │         │ {category, color, material, ...}           │
  │    ┌────┴────┐                                      │
  │    ▼         ▼                                      │
  │ ┌────────┐ ┌────────────┐                           │
  │ │B: 文案  │ │C: 定价建议  │  可并行执行               │
  │ │ 生成   │ │ (竞品分析)  │                           │
  │ │Agent-  │ │ Agent-     │                           │
  │ │Copywr. │ │ Pricing    │                           │
  │ └───┬────┘ └─────┬──────┘                           │
  │     │            │                                  │
  │     └─────┬──────┘                                  │
  │           ▼                                         │
  │    ┌──────────────┐                                 │
  │    │D: 商品详情页  │  Agent-PageBuilder               │
  │    │  自动排版     │  Cap: html_generation            │
  │    └──────┬───────┘                                 │
  │           ▼                                         │
  │    [Output: 完整商品详情页 HTML + 建议售价]            │
  │                                                     │
  │  总耗时: max(B,C) + A + D ≈ 2s + 3s + 1s = 6s      │
  │  串行耗时: A + B + C + D ≈ 2s + 3s + 3s + 1s = 9s  │
  │  并行加速比: 1.5x                                   │
  └─────────────────────────────────────────────────────┘
```

对应 DAG 定义：

```python
# 伪代码 — 构建商品上架 DAG
from aacp_sdk import WeaveClient, DAGBuilder

dag = DAGBuilder(root_task="product_listing_001")

node_a = dag.add_task("vision", required_caps=["image_recognition"])
node_b = dag.add_task("copywriting", required_caps=["text_generation"])
node_c = dag.add_task("pricing", required_caps=["market_analysis"])
node_d = dag.add_task("page_builder", required_caps=["html_generation"])

dag.add_edge(node_a, node_b)   # B 依赖 A 的属性输出
dag.add_edge(node_a, node_c)   # C 依赖 A 的品类信息
dag.add_edge(node_b, node_d)   # D 依赖 B 的文案
dag.add_edge(node_c, node_d)   # D 依赖 C 的定价

weave = WeaveClient("https://relay.aacp.network:8443")
result = weave.execute(dag, timeout_sec=30)
print(f"页面: {result.output['html_url']}, 建议价: {result.output['price']}")
```

## 7.6 WEAVE 容错策略

| 故障类型 | 检测方式 | 恢复策略 |
|---------|---------|---------|
| 子任务超时 | AAP 心跳丢失 | 重新撮合备选 Agent，最多重试 3 次 |
| 子任务失败 | AAP 状态 → FAILED | 检查 DAG 中是否有备选路径（冗余边） |
| Agent 崩溃 | 连续 3 次心跳缺失 | 标记 Agent 不可用，信誉扣分，重新调度 |
| 数据传递失败 | IPFS CID 不可达 | Relay 节点缓存层重试，超时则 FAILED |
| DAG 死锁 | 循环依赖检测 | 构建时拓扑排序预检，运行时 watchdog 超时 |
| 编排者离线 | 编排者心跳丢失 | T0 Validator 接管编排权，继续执行 |

---

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

# 9. 法币原生经济模型

## 9.1 设计原则

```
┌─────────────────────────────────────────────────────────────┐
│                  AACP 经济模型公理                            │
│                                                             │
│  公理 1: 无平台代币                                          │
│          ─ 所有定价、结算、保证金均以法币（CNY/USD）计         │
│          ─ 杜绝投机对协议治理的干扰                           │
│                                                             │
│  公理 2: 佣金即收入                                          │
│          ─ 协议的唯一收入来源是每笔交易的佣金抽成              │
│          ─ 佣金率 8%–15%，按服务类型和交易额浮动               │
│                                                             │
│  公理 3: 保证金即信任                                        │
│          ─ Provider 必须缴纳法币保证金才能挂单                 │
│          ─ 保证金是争议赔付和反欺诈的经济后盾                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 9.2 交易资金流

```
  一笔典型 Agent 交易的资金流（以 ¥100 服务费、10% 佣金为例）

  Consumer                                          Provider
     │                                                 │
     │── 支付 ¥110 ──→ [法币网关] ──→ [Escrow 托管账户]  │
     │   (¥100服务费    │              │                 │
     │    +¥10佣金)     │              │                 │
     │                  │              │                 │
     │           任务执行 & 验证通过                       │
     │                  │              │                 │
     │                  │    ┌─────────┴──────────┐      │
     │                  │    │    佣金分配 ¥10      │      │
     │                  │    │                     │      │
     │                  │    │  验证者 ¥4  (40%)   │      │
     │                  │    │  Relay  ¥2  (20%)   │      │
     │                  │    │  保险池 ¥2  (20%)   │      │
     │                  │    │  运营   ¥2  (20%)   │      │
     │                  │    │                     │      │
     │                  │    └─────────────────────┘      │
     │                  │              │                  │
     │                  │   [Escrow] ──┼──→ Provider ¥100 │
     │                  │              │                  │
```

## 9.3 佣金率计算

佣金率不是固定值，而是在 8%–15% 区间内 **动态浮动**：

```
  commission_rate = base_rate + volume_adj + risk_adj + reputation_disc

  其中:
    base_rate       = 0.12 (12%, 基准)
    volume_adj      = -0.01 × min(floor(monthly_volume / ¥10,000), 4)
                      (月交易额每增 ¥1万, 降 1%, 最多降 4%)
    risk_adj        = +0.02 × risk_tier  (risk_tier ∈ {0, 1})
                      (高风险服务类型 +2%)
    reputation_disc = -0.01 × floor(provider_rep / 250)
                      (信誉每满 250 分, 降 1%, 上限 -2%)

  最终: commission_rate = clamp(commission_rate, 0.08, 0.15)
```

| 场景 | 月交易额 | 风险等级 | 信誉分 | 最终佣金率 |
|------|---------|---------|--------|----------|
| 新 Provider，普通服务 | ¥5,000 | 低(0) | 100 | 12% |
| 成熟 Provider，大量级 | ¥80,000 | 低(0) | 800 | 8% (触底) |
| 新 Provider，高风险服务 | ¥3,000 | 高(1) | 50 | 14% |
| 资深 Provider，高风险 | ¥50,000 | 高(1) | 900 | 10% |

## 9.4 佣金分配明细

```
┌──────────────────────────────────────────────────────────────────┐
│                    佣金分配比例（链上硬编码）                       │
│                                                                  │
│  接收方             │ 比例  │ 用途                                │
│  ──────────────────┼──────┼──────────────────────────────       │
│  T0 Validator      │  40% │ 出块奖励 + 交易验证激励               │
│  T3 Relay          │  20% │ 中继转发、索引维护、边缘节点服务       │
│  Insurance Pool    │  20% │ 争议赔付、系统性风险缓冲               │
│  Platform Ops      │  20% │ 开发团队、基础设施、社区运营            │
│                    │      │                                      │
│  合计              │ 100% │                                      │
└──────────────────────────────────────────────────────────────────┘
```

**Validator 内部分配**（40% 部分）：

```
  Validator 奖励 = 出块奖励 + 交易手续费分成

  出块 Validator:     获得该区块佣金总额的 50%
  其余参与投票的 Validator: 按质押权重均分剩余 50%

  示例（21 个 Validator, 区块佣金 ¥1000, Validator 池 = ¥400）:
    出块者:  ¥400 × 50% = ¥200
    其余20:  ¥400 × 50% / 20 = ¥10/each
```

## 9.5 法币保证金机制

```protobuf
// aacp.v1.fiat — 保证金
message DepositAccount {
  bytes   provider        = 1;  // Provider Ed25519 公钥
  string  currency        = 2;  // "CNY" | "USD"
  string  total_deposited = 3;  // 累计缴纳
  string  available       = 4;  // 可用余额（未冻结）
  string  frozen          = 5;  // 冻结中（争议锁定）
  string  slashed         = 6;  // 累计罚没
  int64   last_deposit_at = 7;
}

// 保证金等级
// Provider 挂单前必须满足对应等级的最低保证金
message DepositTier {
  string tier_name       = 1;  // "starter" | "pro" | "enterprise"
  string min_deposit_cny = 2;
  string min_deposit_usd = 3;
  uint32 max_concurrent  = 4;  // 最大并发任务数
  string max_single_order = 5; // 单笔订单上限
}
```

| 等级 | 最低保证金(CNY) | 最低保证金(USD) | 最大并发任务 | 单笔上限 |
|------|---------------|---------------|------------|---------|
| Starter | ¥1,000 | $150 | 5 | ¥2,000 |
| Pro | ¥10,000 | $1,500 | 50 | ¥20,000 |
| Enterprise | ¥100,000 | $15,000 | 500 | ¥200,000 |

## 9.6 Escrow 托管流程

```
  法币 Escrow 状态机

  ┌──────────┐  consumer_pay  ┌──────────┐   task_ok    ┌──────────┐
  │  PENDING  │─────────────→│  FUNDED   │────────────→│ RELEASING│
  └──────────┘               └──────────┘              └────┬─────┘
       │                          │                         │
       │ timeout                  │ dispute                 │ done
       ▼                          ▼                         ▼
  ┌──────────┐              ┌──────────┐             ┌──────────┐
  │ CANCELED │              │  LOCKED  │             │ RELEASED │
  └──────────┘              └────┬─────┘             └──────────┘
                                 │
                        ┌────────┴────────┐
                        ▼                 ▼
                  ┌──────────┐     ┌──────────┐
                  │REFUNDED  │     │ SLASHED  │
                  │(退Consumer)│   │(赔+罚没) │
                  └──────────┘     └──────────┘
```

## 9.7 保险池运作

```
  保险池资金来源与支出

  ┌──────────────────────────────────────────────┐
  │              Insurance Pool                  │
  │                                              │
  │  收入:                                        │
  │    ├── 每笔交易佣金的 20%     ← 主要来源       │
  │    ├── Provider 罚没金                        │
  │    └── 过期未领退款                            │
  │                                              │
  │  支出:                                        │
  │    ├── 争议仲裁赔付          ← 主要支出        │
  │    ├── 系统性故障补偿                          │
  │    └── 极端情况：保险池余额 < 安全线            │
  │         → 触发紧急治理提案                      │
  │         → 临时上调佣金率至 15% 上限             │
  │                                              │
  │  安全线:                                      │
  │    ≥ 200% × 历史最大单笔赔付额                 │
  │    (链上参数 insurance_pool_safety_ratio)      │
  │                                              │
  └──────────────────────────────────────────────┘
```

---

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

# 11. 信誉系统

## 11.1 设计目标

信誉系统（REP）为 AACP 网络中的每个参与者——Provider、Consumer、Validator、Relay——维护一个 **链上多维信誉评分**，直接影响撮合排序、佣金折扣、保证金要求和仲裁权重。

```
┌──────────────────────────────────────────────────────────────┐
│                 信誉系统核心循环                                │
│                                                              │
│    行为 ──→ 评分更新 ──→ 影响权益 ──→ 激励行为 ──→ ...        │
│                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────────┐             │
│  │ 正向行为  │   │ 评分上升  │   │ 佣金下降     │             │
│  │ 按时交付  │──→│ REP += Δ │──→│ 排序靠前     │──→ 更多订单  │
│  │ 高质量   │   │          │   │ 保证金减免   │             │
│  └──────────┘   └──────────┘   └──────────────┘             │
│                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────────┐             │
│  │ 负向行为  │   │ 评分下降  │   │ 佣金上升     │             │
│  │ 超时失败  │──→│ REP -= Δ │──→│ 排序靠后     │──→ 订单减少  │
│  │ 欺诈     │   │          │   │ 挂单暂停     │             │
│  └──────────┘   └──────────┘   └──────────────┘             │
└──────────────────────────────────────────────────────────────┘
```

## 11.2 多维评分模型

每个参与者维护 **5 个维度** 的子评分，加权合成总分：

```
  总信誉分 R = Σ(wᵢ × Dᵢ),  R ∈ [0, 1000]

  ┌──────────────┬────────┬────────────────────────────────────┐
  │ 维度 Dᵢ      │ 权重 wᵢ │ 计算依据                            │
  ├──────────────┼────────┼────────────────────────────────────┤
  │ 交付质量     │  0.30  │ 任务成功率、验证通过率、Consumer评价  │
  │ (Quality)    │        │                                    │
  ├──────────────┼────────┼────────────────────────────────────┤
  │ 时效性       │  0.25  │ 平均完成时间 / SLA约定时间           │
  │ (Timeliness) │        │                                    │
  ├──────────────┼────────┼────────────────────────────────────┤
  │ 可靠性       │  0.20  │ 心跳在线率、任务中断率、重试率       │
  │ (Reliability)│        │                                    │
  ├──────────────┼────────┼────────────────────────────────────┤
  │ 争议记录     │  0.15  │ 争议发起率、败诉率、赔付历史         │
  │ (Disputes)   │        │                                    │
  ├──────────────┼────────┼────────────────────────────────────┤
  │ 网络贡献     │  0.10  │ 节点在线时长、中继转发量、验证参与    │
  │ (Network)    │        │                                    │
  └──────────────┴────────┴────────────────────────────────────┘
```

```protobuf
// aacp.v1.rep — 信誉评分
message ReputationScore {
  bytes   entity         = 1;  // Ed25519 公钥
  string  entity_type    = 2;  // "provider" | "consumer" | "validator" | "relay"

  // 子维度分 (0–1000)
  uint32  quality_score     = 3;
  uint32  timeliness_score  = 4;
  uint32  reliability_score = 5;
  uint32  dispute_score     = 6;
  uint32  network_score     = 7;

  // 加权总分 (0–1000)
  uint32  total_score    = 8;

  // 统计数据
  uint64  total_tasks    = 9;
  uint64  success_tasks  = 10;
  uint64  failed_tasks   = 11;
  uint64  disputed_tasks = 12;

  int64   last_updated   = 13;
  uint64  update_epoch   = 14;  // 最近一次评分更新的 Epoch
}
```

## 11.3 评分更新规则

每次任务完成（成功或失败）时，链上自动更新信誉分：

```
  任务完成 → 评分增量计算

  ┌─────────────────────────────────────────────────┐
  │  输入:                                           │
  │    task_result   ∈ {SUCCESS, FAILED, DISPUTED}   │
  │    completion_ms  (实际耗时)                      │
  │    sla_timeout_ms (SLA约定耗时)                   │
  │    consumer_rating ∈ [1, 5]                      │
  │    verification_passed: bool                      │
  │                                                  │
  │  Quality 增量:                                    │
  │    if SUCCESS && verification_passed:             │
  │      Δq = +10 × (consumer_rating / 5.0)          │
  │    elif FAILED:                                   │
  │      Δq = -30                                    │
  │    elif DISPUTED && provider_lost:                │
  │      Δq = -50                                    │
  │                                                  │
  │  Timeliness 增量:                                 │
  │    ratio = completion_ms / sla_timeout_ms         │
  │    if ratio ≤ 0.5:  Δt = +8   (远快于SLA)        │
  │    elif ratio ≤ 1.0: Δt = +4  (在SLA内)          │
  │    elif ratio ≤ 1.5: Δt = -10 (轻微超时)          │
  │    else:             Δt = -25 (严重超时)           │
  │                                                  │
  │  最终: Dᵢ = clamp(Dᵢ + Δᵢ, 0, 1000)             │
  │        total = Σ(wᵢ × Dᵢ)                        │
  └─────────────────────────────────────────────────┘
```

## 11.4 信誉衰减

长期不活跃的参与者信誉自然衰减，避免"僵尸高分"：

```go
// pkg/rep/decay.go

package rep

const (
    DecayEpochs    = 4    // 连续 4 个 Epoch 无活动触发衰减
    DecayRate      = 0.02 // 每 Epoch 衰减 2%
    MinDecayScore  = 100  // 衰减下限
    EpochDuration  = 7    // 1 Epoch = 7 天
)

func ApplyDecay(score *ReputationScore, currentEpoch uint64) {
    inactive := currentEpoch - score.UpdateEpoch
    if inactive < DecayEpochs {
        return
    }

    decayEpochs := inactive - DecayEpochs + 1
    multiplier := 1.0
    for i := uint64(0); i < decayEpochs; i++ {
        multiplier *= (1.0 - DecayRate)
    }

    dimensions := []*uint32{
        &score.QualityScore,
        &score.TimelinessScore,
        &score.ReliabilityScore,
        &score.DisputeScore,
        &score.NetworkScore,
    }

    for _, d := range dimensions {
        newVal := uint32(float64(*d) * multiplier)
        if newVal < MinDecayScore {
            newVal = MinDecayScore
        }
        *d = newVal
    }

    score.TotalScore = computeTotal(score)
}
```

## 11.5 信誉等级与权益映射

| 信誉总分 | 等级 | 标识 | 权益 |
|---------|------|------|------|
| 900–1000 | 钻石 | Diamond | 佣金率降至 8%，撮合排序 +30% 加权，可参选 T0 |
| 700–899 | 铂金 | Platinum | 佣金率 -2%，撮合排序 +15% 加权 |
| 500–699 | 黄金 | Gold | 标准佣金率，标准排序 |
| 300–499 | 白银 | Silver | 佣金率 +1%，需额外保证金 20% |
| 100–299 | 青铜 | Bronze | 佣金率 +3%，限制并发任务数 50% |
| 0–99 | 受限 | Restricted | 暂停挂单，仅可接受仲裁和申诉 |

```
  信誉等级 ↔ AMX 撮合权重关系

  撮合 RepScore = raw_score × tier_multiplier

  Diamond:   ×1.30  ████████████████████████████████░░  1.30
  Platinum:  ×1.15  █████████████████████████████░░░░░  1.15
  Gold:      ×1.00  ██████████████████████████░░░░░░░░  1.00
  Silver:    ×0.85  ██████████████████████░░░░░░░░░░░░  0.85
  Bronze:    ×0.60  ███████████████░░░░░░░░░░░░░░░░░░░  0.60
  Restricted: ×0   (不参与撮合)
```

---

# 12. 争议仲裁与反欺诈

## 12.1 三阶段仲裁体系

AACP 的争议解决采用 **渐进升级** 策略，绝大多数争议在前两个阶段自动化解决：

```
  ┌──────────────────────────────────────────────────────────────┐
  │                   三阶段仲裁流程                               │
  │                                                              │
  │  阶段 1: 自动仲裁 (80% 争议在此解决)                           │
  │  ┌───────────────────────────────────────────┐               │
  │  │  规则引擎 + 证据链自动比对                    │               │
  │  │  SLA 合规检查 → 心跳记录 → 结果验证           │               │
  │  │  裁决时间: ≤ 10 分钟                         │               │
  │  └─────────────────────┬─────────────────────┘               │
  │                        │ 无法自动判定                          │
  │                        ▼                                     │
  │  阶段 2: 委员会仲裁 (15% 争议在此解决)                         │
  │  ┌───────────────────────────────────────────┐               │
  │  │  随机选取 5 名高信誉 Validator 组成仲裁委员会  │               │
  │  │  双方提交补充证据 → 投票表决 (3/5 多数)       │               │
  │  │  裁决时间: ≤ 24 小时                         │               │
  │  └─────────────────────┬─────────────────────┘               │
  │                        │ 任一方不服                            │
  │                        ▼                                     │
  │  阶段 3: 全网仲裁 (≤5% 争议到达此阶段)                         │
  │  ┌───────────────────────────────────────────┐               │
  │  │  全部 T0 Validator 参与投票 (2/3 超多数)     │               │
  │  │  证据公开、投票记录上链                       │               │
  │  │  裁决时间: ≤ 72 小时                         │               │
  │  │  裁决结果为最终裁决，不可再上诉               │               │
  │  └───────────────────────────────────────────┘               │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

## 12.2 仲裁状态机

```
                      争议仲裁状态机

  ┌────────────┐
  │  FILED     │  任一方提交争议
  └─────┬──────┘
        │ auto_evaluate
        ▼
  ┌────────────┐   auto_resolved   ┌──────────────┐
  │ AUTO_REVIEW│─────────────────→│ AUTO_RESOLVED │
  └─────┬──────┘                  └──────────────┘
        │ escalate
        ▼
  ┌────────────┐   committee_vote  ┌──────────────┐
  │ COMMITTEE  │─────────────────→│ COMM_RESOLVED │
  │ _REVIEW    │                  └──────────────┘
  └─────┬──────┘
        │ appeal
        ▼
  ┌────────────┐   full_vote       ┌──────────────┐
  │ FULL_REVIEW│─────────────────→│FINAL_RESOLVED │
  └────────────┘                  └──────────────┘

  每个 RESOLVED 状态均携带:
    verdict:   "consumer_wins" | "provider_wins" | "split"
    actions:   [refund?, slash?, rep_penalty?, ban?]
```

```protobuf
// aacp.v1.arb — 争议仲裁
message Dispute {
  string  dispute_id     = 1;
  string  order_id       = 2;   // 关联 AMX 订单
  string  task_id        = 3;   // 关联 AAP 任务
  bytes   plaintiff      = 4;   // 原告 Ed25519 公钥
  bytes   defendant      = 5;   // 被告
  string  reason         = 6;   // 争议原因描述
  
  // 证据
  repeated EvidenceItem evidence = 7;
  
  // 仲裁过程
  ArbitrationPhase phase   = 8;
  DisputeStatus    status  = 9;
  
  // 裁决
  Verdict verdict          = 10;
  
  int64   filed_at         = 11;
  int64   resolved_at      = 12;
}

message EvidenceItem {
  bytes   submitter   = 1;
  string  type        = 2;   // "sla" | "action_log" | "screenshot" | "api_trace"
  string  ipfs_cid    = 3;   // 大文件存 IPFS
  bytes   inline_data = 4;   // 小数据内联
  bytes   content_hash = 5;  // SHA-256
  int64   submitted_at = 6;
}

message Verdict {
  string   winner        = 1;  // "consumer" | "provider" | "split"
  string   refund_amount = 2;  // 退款金额
  string   slash_amount  = 3;  // 罚没金额 (从Provider保证金扣)
  int32    rep_penalty   = 4;  // 信誉扣分
  bool     ban           = 5;  // 是否封禁
  string   reasoning     = 6;  // 裁决理由 (链上存储)
  
  // 投票记录 (阶段2/3)
  repeated VoteRecord votes = 7;
}

message VoteRecord {
  bytes   validator   = 1;
  string  vote        = 2;  // "consumer" | "provider" | "split" | "abstain"
  int64   voted_at    = 3;
  bytes   signature   = 4;
}

enum ArbitrationPhase {
  PHASE_AUTO      = 0;
  PHASE_COMMITTEE = 1;
  PHASE_FULL      = 2;
}
```

## 12.3 自动仲裁规则引擎

阶段 1 自动仲裁基于预定义规则链：

```
  自动仲裁规则链 (按优先级顺序)

  ┌─────────────────────────────────────────────────────────┐
  │  Rule 1: SLA 超时检查                                    │
  │  if task.completion_time > sla.timeout_sec × 1.0:       │
  │    → verdict = consumer_wins                            │
  │    → refund = 100% service_fee                          │
  │    → slash = 10% deposit                                │
  │    → rep_penalty = -30                                  │
  │                                                         │
  │  Rule 2: 心跳丢失检查                                    │
  │  if missed_heartbeats >= 6 (连续 3 分钟):                │
  │    → verdict = consumer_wins                            │
  │    → refund = 100%                                      │
  │    → slash = 5% deposit                                 │
  │    → rep_penalty = -20                                  │
  │                                                         │
  │  Rule 3: 结果验证失败                                    │
  │  if verification.passed == false && retries >= 3:       │
  │    → verdict = consumer_wins                            │
  │    → refund = 80%                                       │
  │    → rep_penalty = -40                                  │
  │                                                         │
  │  Rule 4: Consumer 无正当理由拒绝                          │
  │  if task.status == VERIFIED && consumer refuses payment: │
  │    → verdict = provider_wins                            │
  │    → release = 100% service_fee to provider             │
  │    → consumer rep_penalty = -20                         │
  │                                                         │
  │  Rule 5: 证据不足                                        │
  │  if 以上规则均不匹配:                                     │
  │    → escalate to PHASE_COMMITTEE                        │
  └─────────────────────────────────────────────────────────┘
```

## 12.4 反欺诈引擎（AFD）

```
  ┌──────────────────────────────────────────────────────────┐
  │                  反欺诈检测层级                             │
  │                                                          │
  │   Layer 1: 实时规则检测 (链上, 每笔 Tx)                    │
  │   ┌───────────────────────────────────────────┐          │
  │   │  • Sybil 检测: 同 IP/设备 注册多 Provider   │          │
  │   │  • 自交易检测: Provider == Consumer (关联分析)│         │
  │   │  • 价格操纵: 异常低价/高价 > 3σ 偏离         │          │
  │   │  • 刷单检测: 高频微额交易模式                 │          │
  │   └───────────────────────────────────────────┘          │
  │                        │                                 │
  │   Layer 2: 批量统计分析 (链下, 每 Epoch)                   │
  │   ┌───────────────────────────────────────────┐          │
  │   │  • 交易图谱分析: 识别环路交易、资金回流       │          │
  │   │  • 行为聚类: 异常行为模式识别                 │          │
  │   │  • 信誉注水: 互评/自评模式检测                │          │
  │   │  • 地理异常: 声称区域与实际 IP 不符           │          │
  │   └───────────────────────────────────────────┘          │
  │                        │                                 │
  │   Layer 3: 深度调查 (人工 + AI, 按需触发)                  │
  │   ┌───────────────────────────────────────────┐          │
  │   │  • 仲裁委员会介入的复杂欺诈案件              │          │
  │   │  • 跨区域、跨时段的组织化欺诈                 │          │
  │   │  • 新型攻击向量的模式归纳与规则更新           │          │
  │   └───────────────────────────────────────────┘          │
  └──────────────────────────────────────────────────────────┘
```

## 12.5 Sybil 防御

```go
// pkg/afd/sybil.go

package afd

import (
    "crypto/sha256"
    "encoding/hex"
    "time"
)

type SybilDetector struct {
    ipIndex      map[string][]string  // IP → []pubkey_hex
    hwIndex      map[string][]string  // HW fingerprint → []pubkey_hex
    regWindow    time.Duration        // 检测时间窗口
    maxPerIP     int                  // 同 IP 最大注册数
    maxPerHW     int                  // 同硬件指纹最大注册数
}

type SybilAlert struct {
    AlertType string    // "same_ip" | "same_hw" | "rapid_reg"
    Pubkeys   []string  // 关联公钥
    Score     float64   // 0.0–1.0, 越高越可疑
    Timestamp time.Time
}

func NewSybilDetector() *SybilDetector {
    return &SybilDetector{
        ipIndex:   make(map[string][]string),
        hwIndex:   make(map[string][]string),
        regWindow: 24 * time.Hour,
        maxPerIP:  3,
        maxPerHW:  2,
    }
}

func (sd *SybilDetector) CheckRegistration(pubkey [32]byte, ip string, hwFingerprint string) []SybilAlert {
    var alerts []SybilAlert
    pkHex := hex.EncodeToString(pubkey[:])

    if keys, ok := sd.ipIndex[ip]; ok && len(keys) >= sd.maxPerIP {
        alerts = append(alerts, SybilAlert{
            AlertType: "same_ip",
            Pubkeys:   append(keys, pkHex),
            Score:     float64(len(keys)+1) / float64(sd.maxPerIP*2),
        })
    }

    hwHash := sha256Hex(hwFingerprint)
    if keys, ok := sd.hwIndex[hwHash]; ok && len(keys) >= sd.maxPerHW {
        alerts = append(alerts, SybilAlert{
            AlertType: "same_hw",
            Pubkeys:   append(keys, pkHex),
            Score:     float64(len(keys)+1) / float64(sd.maxPerHW*2),
        })
    }

    if len(alerts) == 0 {
        sd.ipIndex[ip] = append(sd.ipIndex[ip], pkHex)
        sd.hwIndex[hwHash] = append(sd.hwIndex[hwHash], pkHex)
    }

    return alerts
}

func sha256Hex(s string) string {
    h := sha256.Sum256([]byte(s))
    return hex.EncodeToString(h[:])
}
```

## 12.6 处罚梯度

| 违规等级 | 行为示例 | 信誉扣分 | 保证金罚没 | 其他后果 |
|---------|---------|---------|----------|---------|
| 轻微 | 单次超时、单次心跳丢失 | -10~-20 | 0% | 无 |
| 一般 | 多次交付失败、低质量结果 | -20~-50 | 5%–10% | 降低并发限制 |
| 严重 | 仲裁败诉、恶意拒绝交付 | -50~-100 | 10%–30% | 暂停挂单 7 天 |
| 极严重 | Sybil 攻击、刷单欺诈 | -200~-500 | 50%–100% | 永久封禁 |
| 系统性 | 组织化欺诈、勾结攻击 | 归零 | 100% | 永久封禁 + 公示 |

---

# 13. 安全架构

## 13.1 威胁模型

```
┌──────────────────────────────────────────────────────────────────┐
│                        AACP 威胁模型                              │
│                                                                  │
│  攻击面           │  威胁                  │  缓解措施             │
│  ────────────────┼───────────────────────┼──────────────────── │
│  共识层           │  拜占庭节点 (<1/3)     │  CometBFT BFT 容错   │
│                  │  长程攻击              │  弱主观性检查点        │
│  ────────────────┼───────────────────────┼──────────────────── │
│  P2P 网络        │  Eclipse 攻击          │  多路径连接、种子轮换  │
│                  │  DDoS                  │  Rate limiting + PoW │
│  ────────────────┼───────────────────────┼──────────────────── │
│  AMX 市场        │  价格操纵              │  3σ 异常检测          │
│                  │  抢跑 (Front-running)  │  提交-揭示方案        │
│  ────────────────┼───────────────────────┼──────────────────── │
│  AAP 任务        │  结果伪造              │  证据链 + 验证节点    │
│                  │  Provider 跑路         │  法币保证金 + Escrow  │
│  ────────────────┼───────────────────────┼──────────────────── │
│  Cap-UTXO        │  权限提升              │  链上派生规则强制执行  │
│                  │  重放攻击              │  Nonce + UTXO 单花   │
│  ────────────────┼───────────────────────┼──────────────────── │
│  法币网关        │  双花 / 退款欺诈       │  支付确认等待 + 风控  │
│                  │  内部人员作恶           │  Shamir 多签 + 审计  │
│  ────────────────┼───────────────────────┼──────────────────── │
│  密钥管理        │  私钥泄露              │  Shamir (3,5) 分割   │
│                  │  量子威胁（远期）       │  预留算法迁移接口     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 13.2 密码学方案总览

```
  ┌───────────────────────────────────────────────────────────────┐
  │                    密码学工具箱                                 │
  │                                                               │
  │  用途              算法             参数                       │
  │  ─────────────    ───────────     ──────────────             │
  │  身份签名          Ed25519          Curve25519, 32B key       │
  │  内容哈希          SHA-256          256-bit digest            │
  │  对称加密          AES-256-GCM      256-bit key, 96-bit nonce │
  │  密钥分割          Shamir SSS       (3,5) 阈值方案            │
  │  Merkle 树         SHA-256          IAVL binary tree          │
  │  地址派生          SHA-256(PubKey)  截取前 20 bytes            │
  │  TLS              TLS 1.3          P2P 节点间通信             │
  │                                                               │
  └───────────────────────────────────────────────────────────────┘
```

## 13.3 密钥生命周期

```
  密钥生命周期管理

  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  生成    │──→│  分割    │──→│  使用    │──→│  轮换    │
  │ Ed25519  │   │ Shamir   │   │ 签名/验证│   │ 定期更换 │
  │ KeyPair  │   │ (3,5)    │   │         │   │ 旧密钥   │
  └──────────┘   └──────────┘   └──────────┘   └────┬─────┘
                                                     │
                      ┌──────────────────────────────┘
                      ▼
               ┌──────────┐
               │  销毁    │
               │ 安全擦除 │
               │ 分片销毁 │
               └──────────┘

  Shamir (3,5) 分片分布:
    Share 1 → 节点本地 HSM / Secure Enclave
    Share 2 → 运营团队安全保险柜
    Share 3 → 第三方托管机构
    Share 4 → 冷存储（离线 USB）
    Share 5 → 灾备数据中心
    
    任意 3 片可恢复完整密钥
```

```go
// pkg/crypto/shamir.go — Shamir 秘密分享示例接口

package crypto

type ShamirScheme struct {
    Threshold int // 恢复所需最少分片数 (k=3)
    Total     int // 总分片数 (n=5)
}

type Share struct {
    Index byte   // 分片索引 1..n
    Data  []byte // 分片数据
}

func NewShamirScheme(threshold, total int) *ShamirScheme {
    return &ShamirScheme{Threshold: threshold, Total: total}
}

func (s *ShamirScheme) Split(secret []byte) ([]Share, error) {
    if s.Threshold > s.Total {
        return nil, ErrInvalidThreshold
    }
    shares := make([]Share, s.Total)
    for i := range shares {
        shares[i] = Share{
            Index: byte(i + 1),
            Data:  computeShareGF256(secret, byte(i+1), s.Threshold),
        }
    }
    return shares, nil
}

func (s *ShamirScheme) Reconstruct(shares []Share) ([]byte, error) {
    if len(shares) < s.Threshold {
        return nil, ErrNotEnoughShares
    }
    return lagrangeInterpolateGF256(shares[:s.Threshold]), nil
}
```

## 13.4 前端提交-揭示方案（Anti Front-Running）

AMX 撮合中防止矿工/验证者利用交易排序获利：

```
  Commit-Reveal 防抢跑

  Phase 1: Commit (区块 N)
  ┌───────────────────────────────────────┐
  │  Consumer 提交:                        │
  │  commitment = SHA-256(                 │
  │    request_payload || salt || nonce    │
  │  )                                    │
  │  链上只存 commitment hash              │
  │  验证者无法看到 request 内容            │
  └───────────────────────────────────────┘

  Phase 2: Reveal (区块 N+1 ~ N+3)
  ┌───────────────────────────────────────┐
  │  Consumer 提交:                        │
  │  {request_payload, salt, nonce}        │
  │  链上验证:                             │
  │    SHA-256(payload||salt||nonce)       │
  │    == stored commitment                │
  │  匹配后 request 进入撮合队列           │
  └───────────────────────────────────────┘

  超时规则:
    若 Reveal 在 N+3 之后仍未提交 → commitment 作废
    Consumer 损失 gas，无其他处罚
```

## 13.5 通信安全

```
  节点间通信安全层

  ┌─────────────────────────────────────────────┐
  │          TLS 1.3 + 节点身份绑定              │
  │                                             │
  │  Node A                        Node B       │
  │    │                             │          │
  │    │── TLS ClientHello ────────→│          │
  │    │   (with Ed25519 pubkey     │          │
  │    │    in SNI extension)       │          │
  │    │                             │          │
  │    │←─ TLS ServerHello ─────────│          │
  │    │   (with Ed25519 pubkey)    │          │
  │    │                             │          │
  │    │── Certificate Verify ─────→│          │
  │    │   (Ed25519 签名 TLS 握手)   │          │
  │    │                             │          │
  │    │←─ Certificate Verify ──────│          │
  │    │                             │          │
  │    │══ Encrypted Channel ═══════│          │
  │    │   (AES-256-GCM)           │          │
  │    │                             │          │
  │    │  每条消息额外携带:           │          │
  │    │  • sender Ed25519 签名     │          │
  │    │  • 消息序号 (防重放)        │          │
  │    │  • 时间戳 (±30s 容差)      │          │
  │                                             │
  └─────────────────────────────────────────────┘
```

## 13.6 安全审计清单

| # | 审计项 | 方法 | 频率 |
|---|-------|------|------|
| A1 | 共识安全 | CometBFT 模糊测试 + 形式化验证 | 每版本 |
| A2 | ABCI 状态机 | 属性测试 + 不变量检查 | 每次提交 |
| A3 | 密码学实现 | 第三方安全审计 | 半年一次 |
| A4 | P2P 协议 | 网络模糊测试 + 渗透测试 | 季度 |
| A5 | 法币网关 | PCI DSS 合规审计 | 年度 |
| A6 | 智能合约 / 链上逻辑 | 形式化验证 (TLA+) | 每版本 |
| A7 | 依赖供应链 | SBOM + CVE 扫描 | 每日 CI |
| A8 | 密钥管理 | 红队演练 | 半年 |

---

# 14. 技术栈与实现路线

## 14.1 技术选型全景

```
┌────────────────────────────────────────────────────────────────────┐
│                     AACP 技术栈全景图                                │
│                                                                    │
│  ┌─── 应用层 ───────────────────────────────────────────────────┐  │
│  │  Agent SDK (Go/Python/TS)  │  CLI (cobra)  │  Dashboard (Vue) │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌─── 协议层 ───────────────────────────────────────────────────┐  │
│  │  AMX │ AAP │ WEAVE │ Cap-UTXO │ REP │ ARB │ AFD │ FIAT      │  │
│  │  ─────────── 全部 Go 1.23+ 实现 ──────────────────           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌─── 共识层 ───────────────────────────────────────────────────┐  │
│  │  CometBFT 1.x  │  ABCI 2.0 (FinalizeBlock)  │  IAVL v2     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌─── 存储层 ───────────────────────────────────────────────────┐  │
│  │  IAVL (状态树)  │  BadgerDB (WAL)  │  IPFS (大对象)          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌─── 网络层 ───────────────────────────────────────────────────┐  │
│  │  libp2p (P2P)  │  gRPC (RPC)  │  Protobuf (序列化)          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌─── 密码学 ───────────────────────────────────────────────────┐  │
│  │  Ed25519  │  SHA-256  │  AES-256-GCM  │  Shamir (3,5)       │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
```

## 14.2 Go 工程结构

```bash
aacp/
├── cmd/
│   ├── aacpd/              # 主节点二进制
│   │   └── main.go
│   ├── aacp-cli/           # 命令行工具
│   │   └── main.go
│   └── aacp-relay/         # 轻量级 Relay 二进制
│       └── main.go
├── pkg/
│   ├── amx/                # AMX 市场引擎
│   │   ├── engine.go       # 撮合引擎接口 + 实现
│   │   ├── listing.go      # 挂单管理
│   │   ├── order.go        # 订单状态机
│   │   └── scoring.go      # 撮合评分算法
│   ├── aap/                # AAP 行动协议
│   │   ├── task.go         # 任务状态机
│   │   ├── sla.go          # SLA 管理
│   │   ├── evidence.go     # 证据链
│   │   └── heartbeat.go    # 心跳管理
│   ├── weave/              # WEAVE 协调引擎
│   │   ├── dag.go          # DAG 定义与调度
│   │   ├── scheduler.go    # 拓扑排序调度器
│   │   └── dataref.go      # 数据传递
│   ├── caputxo/            # Capability-UTXO
│   │   ├── utxo.go         # UTXO 核心逻辑
│   │   ├── validate.go     # 派生验证
│   │   └── store.go        # IAVL 存储层
│   ├── rep/                # 信誉系统
│   │   ├── score.go        # 评分计算
│   │   └── decay.go        # 衰减逻辑
│   ├── arb/                # 争议仲裁
│   │   ├── dispute.go      # 争议状态机
│   │   ├── rules.go        # 自动仲裁规则引擎
│   │   └── committee.go    # 委员会投票
│   ├── afd/                # 反欺诈
│   │   ├── sybil.go        # Sybil 检测
│   │   └── anomaly.go      # 异常检测
│   ├── fiat/               # 法币结算
│   │   ├── escrow.go       # Escrow 托管
│   │   ├── commission.go   # 佣金计算与分配
│   │   └── deposit.go      # 保证金管理
│   ├── node/               # 节点管理
│   │   ├── registry.go     # 节点注册
│   │   └── election.go     # Validator 选举
│   └── crypto/             # 密码学工具
│       ├── ed25519.go
│       ├── shamir.go
│       └── aes_gcm.go
├── internal/
│   ├── abci/               # ABCI 2.0 应用实现
│   │   ├── app.go          # Application 主入口
│   │   ├── finalize.go     # FinalizeBlock 处理
│   │   └── query.go        # Query 处理
│   ├── state/              # 状态管理
│   │   ├── iavl.go         # IAVL 树封装
│   │   └── snapshot.go     # 状态快照
│   └── p2p/                # P2P 网络
│       ├── host.go         # libp2p Host
│       └── relay.go        # Relay 协议
├── proto/
│   └── aacp/v1/            # Protobuf 定义
│       ├── amx.proto
│       ├── aap.proto
│       ├── weave.proto
│       ├── caputxo.proto
│       ├── rep.proto
│       ├── arb.proto
│       ├── fiat.proto
│       └── node.proto
├── api/
│   ├── grpc/               # gRPC 服务定义
│   └── rest/               # REST 网关 (grpc-gateway)
├── config/
│   ├── node.toml           # 节点配置
│   └── genesis.json        # 创世配置
├── scripts/
│   ├── init_testnet.sh     # 测试网初始化
│   └── benchmark.sh        # 性能基准测试
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
└── README.md
```

## 14.3 ABCI 2.0 应用入口

```go
// internal/abci/app.go

package abci

import (
    abcitypes "github.com/cometbft/cometbft/abci/types"
    "github.com/cosmos/iavl"

    "aacp/pkg/amx"
    "aacp/pkg/aap"
    "aacp/pkg/weave"
    "aacp/pkg/caputxo"
    "aacp/pkg/rep"
    "aacp/pkg/arb"
    "aacp/pkg/fiat"
)

type AACPApp struct {
    tree *iavl.MutableTree

    amxEngine   amx.Engine
    aapManager  aap.Manager
    weaveScheduler weave.Scheduler
    capStore    caputxo.Store
    repService  rep.Service
    arbService  arb.Service
    fiatService fiat.Service
}

func (app *AACPApp) FinalizeBlock(
    req *abcitypes.FinalizeBlockRequest,
) (*abcitypes.FinalizeBlockResponse, error) {
    results := make([]*abcitypes.ExecTxResult, len(req.Txs))

    for i, txBytes := range req.Txs {
        envelope, err := DecodeTxEnvelope(txBytes)
        if err != nil {
            results[i] = &abcitypes.ExecTxResult{Code: 1, Log: err.Error()}
            continue
        }

        if err := VerifySignature(envelope); err != nil {
            results[i] = &abcitypes.ExecTxResult{Code: 2, Log: "invalid signature"}
            continue
        }

        result := app.routeAndExecute(envelope)
        results[i] = result
    }

    app.endBlockHousekeeping(req.Height)

    hash, _, err := app.tree.SaveVersion()
    if err != nil {
        return nil, err
    }

    return &abcitypes.FinalizeBlockResponse{
        TxResults: results,
        AppHash:   hash,
    }, nil
}

func (app *AACPApp) routeAndExecute(env *TxEnvelope) *abcitypes.ExecTxResult {
    switch env.Module {
    case "amx":
        return app.amxEngine.Execute(env.Action, env.Payload, env.Sender)
    case "aap":
        return app.aapManager.Execute(env.Action, env.Payload, env.Sender)
    case "weave":
        return app.weaveScheduler.Execute(env.Action, env.Payload, env.Sender)
    case "caputxo":
        return app.capStore.Execute(env.Action, env.Payload, env.Sender)
    case "rep":
        return app.repService.Execute(env.Action, env.Payload, env.Sender)
    case "arb":
        return app.arbService.Execute(env.Action, env.Payload, env.Sender)
    case "fiat":
        return app.fiatService.Execute(env.Action, env.Payload, env.Sender)
    default:
        return &abcitypes.ExecTxResult{Code: 3, Log: "unknown module"}
    }
}
```

## 14.4 构建与运行

```bash
# 环境要求
# Go 1.23+, protoc 25+, CometBFT 1.x

# 编译
make build           # → bin/aacpd, bin/aacp-cli, bin/aacp-relay

# 初始化单节点测试网
./bin/aacpd init --moniker "my-node" --chain-id aacp-testnet-1

# 配置创世 Validator
./bin/aacpd genesis add-validator \
    --pubkey $(./bin/aacpd tendermint show-validator) \
    --deposit 500000 \
    --tier T0_VALIDATOR

# 启动节点
./bin/aacpd start \
    --rpc.laddr tcp://0.0.0.0:26657 \
    --grpc.address 0.0.0.0:9090 \
    --p2p.seeds "seed1@10.0.1.1:26656,seed2@10.0.1.2:26656"

# Docker 一键启动 4 节点测试网
docker compose -f deploy/docker-compose.testnet.yml up -d
```

```dockerfile
# Dockerfile — 多阶段构建
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o /aacpd ./cmd/aacpd

FROM alpine:3.20
RUN apk add --no-cache ca-certificates
COPY --from=builder /aacpd /usr/local/bin/aacpd
EXPOSE 26656 26657 9090
ENTRYPOINT ["aacpd"]
CMD ["start"]
```

## 14.5 性能基线

| 基准测试项 | 目标 | 实测环境 |
|-----------|------|---------|
| AMX 撮合 TPS | ≥ 2,000 | 4 × T0 Validator, 16C/64GB |
| AAP 状态转换延迟 | ≤ 50ms | 单笔 Tx 端到端 |
| WEAVE DAG 调度（10节点） | ≤ 200ms | 拓扑排序 + 任务下发 |
| Cap-UTXO 花费验证 | ≤ 5ms | 含 IAVL 读取 + 签名验证 |
| IAVL Commit | ≤ 100ms | 10M key 状态树 |
| 区块终局 (Finality) | ≤ 3s | 21 Validator BFT |
| P2P 消息广播 (1000节点) | ≤ 500ms | Gossip 到 95% 节点 |

---

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

# 16. 路线图

```
  AACP 里程碑路线图
  
  2026 Q2                    2026 Q3                    2026 Q4
  ──────────────────────── ──────────────────────── ────────────────────────
  
  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
  │ M1: Foundation      │  │ M2: Testnet Alpha   │  │ M3: Testnet Beta    │
  │                     │  │                     │  │                     │
  │ • CometBFT 集成     │  │ • 4节点测试网上线    │  │ • 公开测试网         │
  │ • ABCI 2.0 骨架     │  │ • AMX 基础撮合      │  │ • AMX 完整撮合       │
  │ • IAVL 状态存储     │  │ • AAP 核心状态机    │  │ • WEAVE DAG 引擎     │
  │ • Ed25519 + Protobuf│  │ • Cap-UTXO 基本流程 │  │ • 信誉系统 v1        │
  │ • P2P 基础连通      │  │ • 单币种法币 Escrow │  │ • 仲裁阶段 1+2       │
  │ • CI/CD 流水线      │  │ • CLI + 基础 SDK    │  │ • 双币种(CNY+USD)    │
  └─────────────────────┘  └─────────────────────┘  │ • T0-T3 节点上线     │
                                                    │ • 漏洞赏金计划       │
                                                    └─────────────────────┘
  
  2027 Q1                    2027 Q2                    2027 Q3+
  ──────────────────────── ──────────────────────── ────────────────────────
  
  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
  │ M4: Security Audit  │  │ M5: Mainnet Launch  │  │ M6: Ecosystem       │
  │                     │  │                     │  │                     │
  │ • 第三方安全审计     │  │ • 主网创世启动       │  │ • Agent SDK 多语言   │
  │ • 形式化验证 (TLA+) │  │ • 21 Validator 就位 │  │ • 开发者市场         │
  │ • 红队渗透测试      │  │ • 法币网关对接       │  │ • 企业级 SLA 模板    │
  │ • 性能压测 & 调优   │  │ • 治理模块激活       │  │ • 跨链互操作         │
  │ • 反欺诈引擎 v1     │  │ • T4/T5 边缘节点    │  │ • AI Agent 生态基金  │
  │ • 文档 & API 参考   │  │ • 保险池初始注资     │  │ • 行业垂直解决方案   │
  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘
```

---

# 17. 结语

AACP 不试图做"又一条公链"或"又一个 AI 平台"。我们聚焦一个明确的结构性缺口：

> **AI Agent 需要一个交易市场来被定价和交易，需要一个责任网络来被审计和问责。**

三大原则指导了每一个设计决策：

- **Marketplace-First**：AMX 让 Agent 服务像商品一样被发现、比较、购买。
- **Fiat-Native**：无代币、法币结算，让协议服务于真实商业而非投机。
- **Edge-First**：六级节点网络让 Agent 在数据产生的地方执行，而非全部回传云端。

这份白皮书描述了协议的完整设计——从撮合引擎到证据链，从能力令牌到三阶段仲裁。对应的技术规范文档（AACP_TECHNICAL_SPEC.md）将提供工程实现级别的详细定义。

我们相信 Agent 经济的基础设施层才刚刚开始，AACP 将是其中关键的一块拼图。

---

> **AACP — Agent Accountability & Coordination Protocol**
>
> 让每一个 AI Agent 都可信、可交易、可问责。

---

# 附录 A：白皮书自检报告

## A.1 术语一致性自检

| 术语 | 首次出现 | 全文统一性 | 状态 |
|------|---------|-----------|------|
| AACP (Agent Accountability & Coordination Protocol) | §1 | 全文一致 | ✅ |
| AMX (Agent Marketplace Exchange) | §4 | §4/§5/§9/§12 一致 | ✅ |
| AAP (Agent Action Protocol) | §4 | §4/§6/§7/§9/§12 一致 | ✅ |
| WEAVE (Workflow Engine for Agent Versatile Execution) | §4 → §7 展开 | 全文一致 | ✅ |
| Capability-UTXO / Cap-UTXO | §4 | §4/§5/§6/§7/§8 一致 | ✅ |
| REP (信誉系统) | §4 | §4/§11 一致 | ✅ |
| ARB (争议仲裁) | §4 | §4/§12 一致 | ✅ |
| AFD (反欺诈) | §4 | §4/§12 一致 | ✅ |
| FIAT (法币结算) | §4 | §4/§9 一致 | ✅ |
| CometBFT | §1 (技术栈) | §4/§10/§14 一致 | ✅ |
| ABCI 2.0 / FinalizeBlock | §4 | §14 一致 | ✅ |
| IAVL | §4 | §6/§8/§14 一致 | ✅ |
| Ed25519 | §4 | §4/§8/§13 一致，均为 32B key | ✅ |
| Shamir (3,5) | §13 | §14 一致 | ✅ |

## A.2 数值一致性自检

| 数值 | 定义位置 | 引用位置 | 一致性 | 状态 |
|------|---------|---------|--------|------|
| 佣金率 8%–15% | §1, §3.2 | §9.3, §11.5, §15.3 | commission_min_bps=800, max=1500 | ✅ |
| 佣金分配 40/20/20/20 | §3.2 | §9.4, §10.2 | 验证者40%/Relay20%/保险池20%/运营20% | ✅ |
| 心跳间隔 30s | §6.5 | §15.3 (heartbeat_interval_sec=30) | 一致 | ✅ |
| Validator 数量 ≤21 | §3.3, §10.1 | §9.4, §10.5, §12.1, §15.3 | max_validators=21 | ✅ |
| 争议仲裁 ≤72h | §1 | §6.2, §12.1, §15.3 (full_review_timeout_hr=72) | 一致 | ✅ |
| 仲裁委员会 5 人 | §12.1 | §15.3 (committee_size=5) | 一致 | ✅ |
| 保险池安全线 200% | §9.7 | §15.3 (insurance_safety_ratio=200) | 一致 | ✅ |
| 衰减触发 4 Epoch | §11.4 | §15.3 (decay_epochs=4) | 一致 | ✅ |
| 衰减率 2% | §11.4 | §15.3 (decay_rate_bps=200) | 一致 | ✅ |
| Epoch 7 天 | §10.5 | §15.3 (epoch_duration_days=7) | 一致 | ✅ |
| Unbonding 7 天 | §10.5 | §15.3 (unbonding_days=7) | 一致 | ✅ |
| Finality ≤3s | §1 | §14.5 | 一致 | ✅ |
| TPS ≥2000 | §1 | §14.5 | 一致 | ✅ |
| AAP 最大重试 3 次 | §6.2 | §7.6, §12.3, §15.3 (max_retries=3) | 一致 | ✅ |

## A.3 状态机一致性自检

| 状态机 | 定义位置 | 状态数 | 终态 | 与其他模块衔接 | 状态 |
|--------|---------|-------|------|---------------|------|
| AMX 订单状态机 | §5.4 | 8 | COMPLETED/DISPUTED/CANCELED/FAILED/REJECTED | ACTIVE→AAP 接管; DISPUTED→ARB | ✅ |
| AAP 任务状态机 | §6.2 | 12 | COMPLETED/RESOLVED | FAILED→DISPUTED→ARB; VERIFIED→SETTLING→FIAT | ✅ |
| WEAVE DAG 状态 | §7.2 | 5 (DAG) + 6 (Node) | DAG_COMPLETED/DAG_FAILED | Node→AAP sub_task; 失败→重试/备选 | ✅ |
| Cap-UTXO 状态 | §8.2 | 4 | SPENT/EXPIRED/REVOKED | AAP 使用→SPENT; 级联撤销→REVOKED | ✅ |
| Escrow 状态机 | §9.6 | 7 | RELEASED/REFUNDED/SLASHED | AAP 完成→RELEASING; 争议→LOCKED→ARB | ✅ |
| 仲裁状态机 | §12.2 | 7 | AUTO/COMM/FINAL_RESOLVED | 裁决→FIAT 赔付 + REP 扣分 | ✅ |

## A.4 修正记录

> 本次自检未发现不一致项。所有术语、数值、状态机在全文中保持一致。

---

# 附录 B：Nano Babana 生图提示词

> 目标：将白皮书关键机制转成可复用视觉素材（官网、路演 Deck、社媒长图）。
>
> 适用模型：**Nano Babana**（若平台写作 Nano Banana，可直接复用）。
>
> 使用方式：先使用 B.1 的基线前后缀，再叠加 B.2 对应场景提示词。

## B.1 基线前后缀（建议统一复用）

**基线前缀（Style Prefix）**

```text
clean futuristic fintech infographic, trustworthy enterprise tone, white and deep navy with teal accents, subtle gold highlights, cinematic soft lighting, high detail, crisp edges, geometric layout, strong visual hierarchy, minimal clutter, premium product render, 8k, sharp focus
```

**统一负面词（Negative Prompt）**

```text
blurry, lowres, noisy background, watermark, random logo, unreadable text, typo text, distorted perspective, crowded composition, oversaturated neon, heavy cyberpunk purple, childish cartoon style, messy UI
```

## B.2 关键内容生图提示词（按章节）

### Prompt 01 · 协议一句话定位（对应 §1 执行摘要）

```text
AACP protocol hero image, AI agent marketplace plus accountability network, two-sided ecosystem with provider agents on left and enterprise consumers on right, central trust layer glowing, fiat settlement symbols CNY USD, no platform token symbolism, modern enterprise tech keynote visual, panoramic composition, 16:9
```

### Prompt 02 · AI Agent 经济断裂带（对应 §2.1 问题与机遇）

```text
conceptual infographic of fragmented AI agent economy, scattered developers open-source teams and private APIs on left, confused buyers enterprises and users on right, broken bridge in the middle with labels discovery pricing accountability settlement collaboration gaps, dramatic but professional, clean data-viz style, 16:9
```

### Prompt 03 · 三大原则对照图（对应 §3 设计哲学）

```text
triple-panel comparison poster for Marketplace-First Fiat-Native Edge-First, each panel shows old model vs AACP model, clear icons for market pricing fiat payment edge execution, consistent enterprise infographic style, balanced typography placeholders, high contrast, presentation-ready, 16:9
```

### Prompt 04 · 四层协议栈总览（对应 §4 协议架构）

```text
exploded architecture diagram of AACP four-layer stack, top application layer AMX AAP WEAVE, middle capability-utxo and reputation arbitration modules, consensus and network layer with CometBFT, infrastructure layer with edge nodes and fiat gateway, clean isometric blocks, technical blueprint aesthetic, white background, 16:9
```

### Prompt 05 · AMX 交易市场流程（对应 §5 AMX）

```text
agent marketplace exchange process visualization, listing discovery bidding matching escrow settlement feedback loop, multiple agent cards with price and SLA badges, real-time order flow arrows, transparent fee display 8 to 15 percent, fintech dashboard style, dynamic yet tidy, 16:9
```

### Prompt 06 · AAP 问责状态机（对应 §6 AAP）

```text
state machine infographic for agent accountability protocol, task lifecycle from created assigned running verifying settling completed with dispute branch, evidence chain snapshots heartbeat monitor and retry logic, formal process diagram, precise arrows and status nodes, compliance-grade visual style, 16:9
```

### Prompt 07 · WEAVE 多 Agent DAG 协作（对应 §7 WEAVE）

```text
multi-agent orchestration DAG graph for ecommerce product listing workflow, nodes for title generation image optimization pricing compliance publishing, dependencies and parallel branches, scheduler selecting best path, resilient retry and fallback paths highlighted, modern graph visualization, clear depth, 16:9
```

### Prompt 08 · Capability-UTXO 生命周期（对应 §8 Cap-UTXO）

```text
tokenized capability lifecycle diagram without cryptocurrency vibe, mint split delegate consume revoke expire states, capability cards flowing through secure channels, cryptographic ownership proof symbols, low-latency permission transfer, technical yet business-friendly infographic, 16:9
```

### Prompt 09 · 法币原生资金流与佣金分配（对应 §9 经济模型）

```text
fiat-native settlement flowchart for AI agent transaction, user payment in CNY USD enters escrow then releases to provider and commission buckets, explicit commission split 40 validator 20 relay 20 insurance 20 operations, banking-grade trust visual, anti-volatility no token speculation motif, 16:9
```

### Prompt 10 · 六级节点拓扑（对应 §10 节点网络）

```text
network topology illustration with six tiers T0 to T5, small validator core ring, full nodes, archive and relay layers, edge execution clusters and micro nodes on devices, latency heatmap from cloud to edge showing major reduction, world map subtle background, high clarity, 21:9 wide
```

### Prompt 11 · 信誉系统与衰减机制（对应 §11 信誉系统）

```text
reputation scoring dashboard for agents and nodes, multi-dimensional radar chart reliability quality timeliness dispute rate, decay over epochs line chart, tier badges linked to rights and staking limits, transparent scoring formula panel, clean analytical UI visual, 16:9
```

### Prompt 12 · 三阶段仲裁与反欺诈（对应 §12 仲裁反欺诈）

```text
three-stage arbitration pipeline image, auto resolve layer committee review layer final appeal layer, fraud detection engine monitoring sybil collusion abnormal behavior, risk alerts and evidence snapshots, justice and security tone without legal cliché, professional governance infographic, 16:9
```

### Prompt 13 · 安全架构总览（对应 §13 安全架构）

```text
security architecture poster for decentralized agent protocol, threat model matrix cryptography stack key lifecycle secure communication anti front-running commit reveal, layered defense shield around core modules, red team checklist style accents, high-trust enterprise cyber visual, 16:9
```


---

<!-- AACP_WHITEPAPER.md 全文完成 -->
