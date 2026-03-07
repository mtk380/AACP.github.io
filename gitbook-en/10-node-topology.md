# 10. Node Tiers and Network Topology

## 10.1 Six-Tier Node System

AACP defines six tiers:

- `T0` Validator (consensus core, max `21`)
- `T1` Full Node
- `T2` Archive Node
- `T3` Relay Node
- `T4` Edge Execution Node
- `T5` Micro Node

## 10.2 Resource and Role Matrix

The whitepaper specifies CPU/RAM/storage/network baselines, deposit requirements, and protocol responsibilities for each tier.

## 10.3 Registration and Discovery

Node registration includes:

- Ed25519 pubkey
- Tier and endpoint
- Region and hardware declaration
- Deposit proof
- Signature

## 10.4 Regional Routing

Routing policy prioritizes nearest relay and local edge capacity, then falls back to cross-region relay routing.

## 10.5 T0 Election

Validator election score:

`score = deposit_weight*0.5 + reputation_weight*0.3 + uptime_weight*0.2`

Runs per epoch (default 7 days), selecting top candidates and applying unbonding cooldown for exits.

## 10.6 Configuration Example

A T3 relay `node.toml` sample is provided for operator onboarding.
