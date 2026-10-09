# Architecture Decision Records

| ADR | Title | Status |
|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted |
| [0002](0002-modular-monolith-first.md) | Modular monolith first | Accepted |
| [0003](0003-double-entry-ledger.md) | Double-entry ledger is the source of truth | Accepted |
| [0004](0004-money-representation.md) | Money representation | Accepted |
| [0005](0005-git-flow-and-conventional-commits.md) | Git flow and Conventional Commits | Accepted |
| [0006](0006-postgresql-and-flyway.md) | PostgreSQL, Flyway, schema per module | Accepted |

## To be written by the developer

Each is attached to a ticket. Use [0000-template.md](0000-template.md) and compare at least two options.

| ADR | Topic | Ticket |
|---|---|---|
| 0007 | Ledger schema: posting direction vs signed amount, cached balance | EW-010 |
| 0008 | Concurrency control for balance updates (pessimistic vs optimistic vs conditional update, deadlock avoidance) | EW-014 |
| 0009 | Idempotency key storage and semantics | EW-016 |
| 0010 | Authentication: JWT access + refresh token strategy | EW-008 |
| 0011 | API error model and versioning | EW-020 |
| 0012 | Payment provider abstraction (Strategy + Adapter) | EW-024 |
| 0013 | Kafka vs RabbitMQ for domain events | EW-040 |
| 0014 | Transactional outbox implementation | EW-041 |
| 0015 | Top-up saga: choreography vs orchestration | EW-043 |
| 0016 | Deployment target on AWS | EW-052 |
