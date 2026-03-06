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
