# 15. Governance and Evolution

## 15.1 Governance Principles

AACP governance is **tokenless** and weighted by:

- Normalized deposit (`40%`)
- Normalized reputation (`40%`)
- Tier weight (`20%`, with `T0 > T1 > ...`)

## 15.2 Proposal Types

Defined proposal classes include:

- `PARAM` (parameter tuning)
- `UPGRADE` (protocol upgrades)
- `EMERGENCY` (urgent security response)
- `ADMISSION` (validator admission/removal)
- `INSURANCE` (large compensation decisions)

Each type has explicit voting thresholds and voting windows.

## 15.3 Governable Parameters

On-chain parameter sets cover AMX, AAP, REP, ARB, FIAT, and NODE modules.

Examples:

- commission min/max
- heartbeat interval
- retry limits
- arbitration timeouts
- insurance safety ratio
- validator count and epoch duration
