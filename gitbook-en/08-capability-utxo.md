# 8. Capability-UTXO Model

## 8.1 Motivation

AACP models capability rights as UTXO-like objects to get deterministic ownership transfer, delegation, and revocation semantics.

## 8.2 UTXO Structure

A capability UTXO captures:

- Owner identity
- Capability scope
- Constraints/policy
- Validity window
- Parent lineage

## 8.3 Lifecycle

Typical lifecycle includes mint, split/derive, delegate, consume, expire, and revoke states.

## 8.4 Derivation and Delegation

Rules enforce constrained delegation:

- Child scope must not exceed parent authority.
- Delegation must satisfy policy and time bounds.
- Single-spend semantics prevent unauthorized replay.

## 8.5 UTXO Set Management

State tracking uses verifiable key-value storage (IAVL) with deterministic transitions for validation and audit.
