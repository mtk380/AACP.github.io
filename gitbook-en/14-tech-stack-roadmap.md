# 14. Tech Stack and Implementation Path

## 14.1 Technology Choices

AACP implementation anchors on:

- `Go` runtime and tooling
- `CometBFT` consensus
- `ABCI 2.0` app/consensus interface
- `IAVL` versioned state tree
- `Protobuf` for protocol contracts

## 14.2 Go Project Layout

The whitepaper maps modules to package domains (`amx`, `aap`, `weave`, `cap`, `rep`, `arb`, `fiat`, `node`, `gov`) to keep code ownership and review boundaries explicit.

## 14.3 ABCI Entry Points

Core transaction and state flow follows `CheckTx`, `DeliverTx/FinalizeBlock`, `Commit`, and query paths with deterministic state commitments.

## 14.4 Build and Run

It includes local and Docker-based flows for:

- single-node initialization
- genesis validator setup
- multi-node testnet bootstrap

## 14.5 Performance Baseline Targets

- AMX matching throughput: `>= 2,000 TPS`
- AAP state transition latency: `<= 50ms`
- WEAVE DAG scheduling (10 nodes): `<= 200ms`
- Cap-UTXO spend verification: `<= 5ms`
- IAVL commit: `<= 100ms`
- Finality: `<= 3s`
- P2P broadcast to 95% of 1000 nodes: `<= 500ms`
