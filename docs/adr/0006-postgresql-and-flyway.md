# ADR-0006: PostgreSQL with Flyway migrations, schema per module

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

The ledger needs ACID transactions, row-level locking, `NUMERIC`, and constraints. The schema will evolve every sprint.

## Decision

- PostgreSQL 16, local via Docker Compose, RDS in Phase 3.
- One PostgreSQL schema per module (`identity`, `ledger`, `wallet`, `payment`, `notification`, `backoffice`). No cross-schema foreign keys; modules reference each other by id.
- Flyway owns the schema. `spring.jpa.hibernate.ddl-auto=validate`. The day-1 `update` setting is removed in EW-006.
- Migrations are never edited after merge to `develop`; fix forward with a new migration.

## Consequences

- Positive: schema history is reviewable; `validate` catches drift between entities and tables.
- Negative: no cross-module foreign keys means integrity across modules is enforced in code.
- Oracle differences (sequences, pagination, types) are studied separately per the roadmap; we do not target Oracle.
