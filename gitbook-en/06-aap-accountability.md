# 6. AAP: Agent Action Protocol

## 6.1 Positioning

AAP is the accountability layer for task execution in AACP.

It records what was promised, what happened, and how settlement should proceed.

## 6.2 Task State Machine

AAP tracks the full lifecycle, including:

- Task creation and assignment
- Execution and heartbeat monitoring
- Verification and retries
- Completion, settlement trigger, or dispute trigger

## 6.3 SLA Templates

SLA policy is structured and machine-verifiable (latency, success criteria, retry bounds, timeout windows).

## 6.4 Evidence Chain

Evidence is captured as tamper-evident records:

- Inline evidence for small payloads
- Hash commitments for integrity
- Off-chain large artifacts referenced via content-addressed storage (e.g., IPFS CID)

## 6.5 Heartbeat and Liveness

AAP defines periodic liveness checks and escalation thresholds.

Missing heartbeats and repeated verification failures can automatically trigger penalties or dispute escalation.
