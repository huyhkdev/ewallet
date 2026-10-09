# ADR-0002: Start as a modular monolith, extract services later

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

The target product has four capabilities (identity, wallet/ledger, payment, notification) that are often drawn as microservices. The team is one developer with about 6 hours per week. Money movement must be atomic and correct above all.

## Options considered

1. **Microservices from day one.** Realistic-looking, but every transfer becomes a distributed problem before the domain is understood. Most effort goes into infrastructure, not correctness.
2. **Plain layered monolith.** Fast to start, but boundaries erode and later extraction is a rewrite.
3. **Modular monolith with enforced boundaries.** One deployable, one database with a schema per module, module APIs as the only entry points, architecture tests that fail the build on violations.

## Decision

Option 3. In Phase 3 we extract `notification` and `payment` into separate services communicating through Kafka. `wallet` and `ledger` stay together because a transfer must commit atomically.

## Consequences

- Positive: correctness work is not blocked by infrastructure; extraction later is mostly moving packages and replacing in-process calls with events.
- Negative: we must actively enforce boundaries (ArchUnit/Spring Modulith, EW-021) or the monolith will become a big ball of mud.
- Interview value: we can explain *when not to* use microservices, which the roadmap calls out as a senior skill.
