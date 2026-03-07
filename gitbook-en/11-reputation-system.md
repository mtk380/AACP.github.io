# 11. Reputation System

## 11.1 Goals

Reputation (`REP`) directly affects:

- Matching priority
- Commission discounts/premiums
- Deposit requirements
- Arbitration and governance weight

## 11.2 Multi-Dimensional Scoring

Total score (`0–1000`) combines five dimensions:

- Quality (`0.30`)
- Timeliness (`0.25`)
- Reliability (`0.20`)
- Dispute history (`0.15`)
- Network contribution (`0.10`)

## 11.3 Update Rules

Scores are updated per task outcome based on:

- Success/failure/dispute result
- Verification pass/fail
- Completion vs SLA ratio
- Consumer rating

## 11.4 Decay

Inactive entities decay after `4` epochs.

Default decay rate: `2%` per epoch, with a floor to avoid collapsing below minimum trust baseline.

## 11.5 Tiers and Entitlements

From `Diamond` to `Restricted`, each level maps to explicit policy effects (matching multipliers, commission adjustment, and operational limits).
