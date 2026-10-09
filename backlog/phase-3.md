# Phase 3 — Events, services, resilience, observability, cloud (months 7–9, sprints 13–19)

**Phase goal:** turn the modular monolith into a small distributed system without losing correctness. Extract `notification` and `payment`, connect them with Kafka through a transactional outbox, make every failure mode observable, and deploy to AWS. Released as `v0.3.0`.

| Sprint | Tickets |
|---|---|
| 13 | EW-040, EW-041 |
| 14 | EW-042 |
| 15 | EW-043 |
| 16 | EW-044, EW-045, EW-046 |
| 17 | EW-047, EW-048, EW-050 |
| 18 | EW-049, EW-051 |
| 19 | EW-052, EW-053, EW-054 |

Before starting, re-read ADR-0002 and write down which ADRs this phase supersedes.

---

## Epic E8 — Event-driven architecture

### EW-040 · Kafka locally (ADR-0013)
spike · Sprint 13 · 4h · adr, infra
**Learning goal:** Kafka vs RabbitMQ (Phase 3).
**Acceptance criteria**
- [ ] Kafka (KRaft mode) in Docker Compose, with a UI (Kafka UI or Redpanda Console).
- [ ] ADR-0013: Kafka vs RabbitMQ for our events, covering ordering, replay, consumer groups, operational cost.
- [ ] Topic naming and partition key rules documented (e.g. partition by wallet id, and why).

### EW-041 · Transactional outbox (ADR-0014)
task · Sprint 13 · 8h · adr, money
**Learning goal:** Transactional Outbox, at-least-once delivery.
**Acceptance criteria**
- [ ] Domain events are written to an `outbox` table in the same transaction as the business change.
- [ ] A relay publishes them to Kafka (polling publisher or Debezium CDC; ADR-0014 compares both).
- [ ] Kill the app between commit and publish in a test: the event is still delivered after restart.
- [ ] Event schema versioned (`eventType`, `eventVersion`, `eventId`, `occurredAt`, payload).

### EW-042 · Extract the notification service
story · Sprint 14 · 8h
**Learning goal:** idempotent consumer, dead letter topic, separate deployable.
**Acceptance criteria**
- [ ] `notification-service` is its own Spring Boot app with its own database schema/DB.
- [ ] Consumer is idempotent by `eventId`; duplicate events send one email.
- [ ] Poison message goes to a DLT after N retries with backoff; documented how to replay it.
- [ ] The monolith no longer contains notification code.

### EW-043 · Extract the payment service and the top-up saga (ADR-0015)
story · Sprint 15 · 12h · adr, money
**Learning goal:** Saga (choreography vs orchestration), eventual consistency.
**Description:** `payment-service` owns top-ups and Stripe. The wallet is credited by the ledger in the monolith after a `TopupSucceeded` event.
**Acceptance criteria**
- [ ] Flow: top-up created in payment-service → Stripe → webhook → `TopupSucceeded` via outbox → wallet/ledger posts entry → `TopupCredited`.
- [ ] Duplicate `TopupSucceeded` never double-credits (ledger idempotency by business key).
- [ ] Failure path designed and tested: what if the ledger rejects the credit (e.g. max balance)? Document the compensation (refund via Stripe).
- [ ] ADR-0015 explains the chosen saga style and its failure handling.
- [ ] Sequence diagram committed.

### EW-044 · Contract tests between services
task · Sprint 16 · 4h
**Learning goal:** Spring Cloud Contract or Pact.
**Acceptance criteria**
- [ ] Event contracts for `TopupSucceeded` and `TransferCompleted` verified on both producer and consumer side in CI.
- [ ] Changing a field name on the producer breaks the build.

---

## Epic E9 — Resilience & performance

### EW-045 · Resilience around Stripe
task · Sprint 16 · 4h
**Learning goal:** Resilience4j (timeout, retry, circuit breaker, bulkhead).
**Acceptance criteria**
- [ ] Explicit connect/read timeouts on Stripe calls.
- [ ] Retries only on safe failures, always with the same Stripe idempotency key.
- [ ] Circuit breaker opens when Stripe is down (simulate with WireMock/Toxiproxy); top-up API fails fast with a clear error.
- [ ] Breaker state visible as a metric.

### EW-046 · Rate limiting with Redis
task · Sprint 16 · 4h · security
**Acceptance criteria**
- [ ] Per-user rate limit on `POST /transfers` and `POST /auth/login` (token bucket or sliding window), returning 429 with `Retry-After`.
- [ ] Works across two app instances (prove with two instances behind a simple proxy or two test contexts).
- [ ] PR explains why a Redis distributed lock is **not** used for balance safety.

### EW-053 · Load test
task · Sprint 19 · 4h
**Acceptance criteria**
- [ ] k6 or Gatling script: 50 req/s of transfers among 1,000 wallets for 5 minutes.
- [ ] Report p50/p95/p99, error rate, DB connections; compared with PRD targets.
- [ ] Ledger invariant check (EW-018) passes after the run.
- [ ] One bottleneck found and fixed, with before/after numbers (CV material).

---

## Epic E10 — Observability & operations

### EW-047 · Metrics, dashboards, structured logs
task · Sprint 17 · 5h
**Acceptance criteria**
- [ ] Prometheus + Grafana in Compose; dashboard with JVM, HTTP, DB pool, Kafka lag, and business metrics (transfers/min, top-up success rate).
- [ ] JSON logs with `traceId`, `userId` (not email), no PII or secrets.
- [ ] One alert rule (e.g. invariant violation or webhook failures) documented.

### EW-048 · Distributed tracing
task · Sprint 17 · 4h
**Acceptance criteria**
- [ ] OpenTelemetry across monolith, payment-service, notification-service, including through Kafka.
- [ ] Screenshot of one top-up trace spanning all three services committed to docs.

### EW-049 · Daily reconciliation with Stripe
story · Sprint 18 · 8h · money
**Learning goal:** reconciliation, fintech operations.
**Acceptance criteria**
- [ ] Scheduled job (idempotent per day, safe on multiple instances) pulls Stripe balance transactions for the previous UTC day.
- [ ] Matches each charge to a top-up and its journal entry; posts Stripe processing fees (primer example 2).
- [ ] Report of mismatches (missing in ledger, missing in Stripe, amount differs) available to `FINANCE`.
- [ ] Test with seeded mismatches of each kind.

### EW-050 · Security hardening
task · Sprint 17 · 5h · security
**Acceptance criteria**
- [ ] Refresh token rotation with reuse detection.
- [ ] OWASP Top 10 checklist filled for this app in `docs/security/owasp-review.md`.
- [ ] Secrets from environment / AWS Secrets Manager; repository scanned for secrets in CI.

### EW-051 · Containers and Kubernetes
task · Sprint 18 · 8h · infra
**Acceptance criteria**
- [ ] Multi-stage Dockerfiles, images < 250 MB, non-root user.
- [ ] Full system runs with one `docker compose up`.
- [ ] Kubernetes manifests (or Helm chart) for kind/minikube: Deployments, Services, ConfigMaps, Secrets, liveness/readiness probes, resource limits, graceful shutdown.

### EW-052 · Deploy to AWS (ADR-0016)
task · Sprint 19 · 8h · adr, infra
**Acceptance criteria**
- [ ] ADR-0016 compares EKS vs ECS Fargate vs a single EC2 for this project, including monthly cost.
- [ ] Deployed with RDS PostgreSQL; infrastructure described as code (Terraform or CDK).
- [ ] CI deploys `main` automatically (or with a manual approval step).
- [ ] Teardown script and a budget alert, so the bill stays near zero.

### EW-054 · Release v0.3.0
chore · Sprint 19 · 1h
Same steps as EW-019. Retro question: "Which failure mode surprised me, and how did I find it?"
