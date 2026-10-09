# ADR-0003: Double-entry ledger is the source of truth for money

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

The simplest wallet stores a `balance` column and does `balance = balance - amount`. That design (used in the day-1 lab) cannot answer "why is this balance 42.10?", hides bugs that create or destroy money, and gives auditors nothing to check.

## Decision

All money movement is recorded as balanced journal entries in the `ledger` module (see [domain primer](../architecture/domain-primer.md)). Postings are append-only. Corrections are reversal entries. Any stored balance is a cache that must be provably equal to the sum of postings.

The `ledger` module exposes one write operation: post a balanced journal entry. It rejects unbalanced entries, mixed currencies, and entries that would make a customer wallet negative.

## Consequences

- Positive: full audit trail, invariants testable automatically, reconciliation becomes possible, strong interview story.
- Negative: more tables and more writes per transfer; balance reads need a cached balance or an aggregate query (decided in EW-010).
- The physical schema (signed vs direction, cached balance, locking) is decided by the developer in ADR-0007 and ADR-0008.
