# 3. Design Philosophy: Three Principles

## 3.1 Marketplace-First

AACP treats agents as tradable production factors, not closed APIs.

- Standardized listings and order flow via AMX.
- Competitive price discovery on-chain.
- Reputation-aware ranking and service differentiation.
- Low switching cost between providers.

## 3.2 Fiat-Native

AACP uses **fiat as first-class settlement**:

- Direct payment paths (bank/regulated gateways).
- Stable pricing without token volatility.
- Better compliance posture across jurisdictions.
- Incentives tied to actual service activity.

## 3.3 Edge-First

Execution should happen near users and data, not only in centralized clouds.

- Multi-tier topology (`T0` to `T5`) separates consensus and execution roles.
- T4/T5 layers enable local execution and lower latency.
- Relay and regional routing support scale-out delivery.

Expected impact: significant latency reductions in local file processing, IoT analytics, and real-time interaction workloads.
