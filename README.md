# Wallet — a digital wallet & payments platform

A production-style digital wallet built with Java 21 and Spring Boot 3. Users top up with a card (Stripe, test mode), send money to each other, and see a full statement. Every money movement is recorded in an immutable **double-entry ledger**.

> Status: Phase 1 (foundation). See [backlog/phase-1.md](backlog/phase-1.md).

## Why this project exists

This repository is run like a real fintech product team: a PRD, architecture decision records, a ticketed backlog, Git flow, code review, CI. It grows in four phases from a well-structured modular monolith to an event-driven system deployed on AWS.

## Features (target v1)

- Registration, login (JWT), simple KYC tiers with limits
- Wallet per user and currency, balance and statement
- Card top-up through Stripe PaymentIntents, confirmed by signed webhooks
- Peer-to-peer transfers with idempotency keys
- Fees posted to a revenue account
- Notifications (email) on money received
- Daily reconciliation against Stripe, admin adjustments with maker-checker approval
- Audit log of every privileged action

## Tech stack

| Area | Choice | Phase |
|---|---|---|
| Language / framework | Java 21, Spring Boot 3, Spring Data JPA (Hibernate) | 1 |
| Database | PostgreSQL 16, Flyway migrations | 1 |
| API | REST, OpenAPI 3, RFC 9457 Problem Details | 2 |
| Testing | JUnit 5, AssertJ, Mockito, Testcontainers, ArchUnit | 2 |
| CI | GitHub Actions | 2 |
| Payments | Stripe (test mode) | 2–3 |
| Messaging | Kafka with transactional outbox | 3 |
| Resilience / cache | Resilience4j, Redis | 3 |
| Observability | Micrometer, Prometheus, Grafana, OpenTelemetry | 3 |
| Runtime | Docker, Kubernetes, AWS (RDS, EKS) | 3 |

## Documentation map

| Document | What it answers |
|---|---|
| [PRD](docs/product/PRD.md) | What we build, for whom, and what we explicitly do not build |
| [Domain primer](docs/architecture/domain-primer.md) | How double-entry bookkeeping works for a wallet |
| [Architecture overview](docs/architecture/overview.md) | Modules, boundaries, and how the system evolves |
| [ADRs](docs/adr/) | Decisions and their trade-offs |
| [Conventions](docs/conventions/) | Code, Git, API, testing, review rules |
| [Backlog](backlog/) | Epics and tickets by phase |

## Running locally

```bash
docker compose -f infra/docker-compose.yml up -d   # PostgreSQL + Mailpit
./mvnw spring-boot:run
```

Stripe keys go in environment variables (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`), never in the repository.

## Contributing

Read [CONTRIBUTING.md](CONTRIBUTING.md). Short version: branch from `develop`, one ticket per branch, Conventional Commits, PR with the template, green CI, one approval.
