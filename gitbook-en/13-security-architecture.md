# 13. Security Architecture

## 13.1 Threat Model

Coverage includes consensus, P2P, marketplace abuse, task falsification, capability abuse, fiat gateway risk, and key compromise.

## 13.2 Cryptography Stack

- Identity signatures: `Ed25519`
- Hashing: `SHA-256`
- Symmetric encryption: `AES-256-GCM`
- Secret sharing: `Shamir (3,5)`
- Transport: `TLS 1.3`

## 13.3 Key Lifecycle

Lifecycle stages: generation, splitting, usage, rotation, destruction.

Key shares are distributed across independent custody locations for fault tolerance and insider-risk reduction.

## 13.4 Anti Front-Running

AMX uses a commit-reveal pattern:

- Commit hash first
- Reveal payload in a bounded block window
- Expire non-revealed commitments

## 13.5 Communication Security

Node-to-node links bind transport and identity, with replay protection via sequence numbers and timestamp tolerance.

## 13.6 Security Audit Program

The whitepaper defines recurring audits for consensus logic, state machines, cryptography, network hardening, fiat compliance, dependency supply chain, and key-management drills.
