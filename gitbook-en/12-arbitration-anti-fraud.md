# 12. Dispute Arbitration and Anti-Fraud

![Figure 12: Three-Layer Arbitration Pipeline](../gitbook-zh/images/fig-12-arbitration-pipeline.png)

*Figure 12: Escalation pipeline from automatic checks to full validator review.*

## 12.1 Three-Stage Arbitration

AACP uses progressive escalation:

1. **Auto arbitration** (majority of cases, target `<= 10 min`)
2. **Committee review** (5 high-reputation validators, target `<= 24h`)
3. **Full validator review** (`2/3` supermajority, target `<= 72h`)

## 12.2 Dispute State Machine

Canonical flow: `FILED -> AUTO_REVIEW -> COMMITTEE_REVIEW -> FULL_REVIEW -> RESOLVED`.

Verdict payload includes winner, refund/slash amounts, reputation penalty, and optional ban.

## 12.3 Rule Engine (Auto Phase)

Examples of deterministic rules:

- SLA timeout breaches
- Heartbeat loss thresholds
- Verification failure after retries
- Unjustified refusal to release payment

## 12.4 AFD: Anti-Fraud Detection

Three layers:

- On-chain real-time rules
- Off-chain epoch analytics
- Deep investigation for complex coordinated attacks

## 12.5 Sybil Defense

Heuristics include per-IP and per-hardware registration clustering, with risk scoring and alert generation.

## 12.6 Penalty Ladder

Penalty gradient ranges from minor timeout penalties to permanent bans and full collateral slashing for systemic fraud.
