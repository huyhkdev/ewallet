# Architecture overview

## 1. Guiding principles

1. **Correct before fast before distributed.** The ledger is right first; then it is fast; only then do we split services.
2. **Modular monolith first** (ADR-0002). Module boundaries are enforced from day one so that extraction in Phase 3 is a deployment change, not a rewrite.
3. **Hexagonal inside each module.** Domain code has no Spring, JPA or Stripe imports. Adapters live at the edges.
4. **The ledger is the source of truth** (ADR-0003). Every other number (balances, reports) is derived from it.

## 2. Modules (bounded contexts)

| Module | Owns | Exposes | Depends on |
|---|---|---|---|
| `identity` | Users, credentials, KYC tier, tokens | `UserLookup` port, auth filter | — |
| `ledger` | Ledger accounts, journal entries, postings, balances | `LedgerService.post(JournalEntryRequest)`, balance queries | — |
| `wallet` | Wallets, limits, transfers, idempotency records | REST `/api/v1/wallets`, `/api/v1/transfers` | `identity`, `ledger` |
| `payment` | Top-ups, provider integration, webhooks | REST `/api/v1/topups`, `/webhooks/stripe` | `wallet` (via events), `ledger` |
| `notification` | Templates, delivery attempts | Listens to domain events | — (events only) |
| `backoffice` | Freezes, adjustments, approvals, audit log, reconciliation | REST `/api/v1/admin/**` | `wallet`, `ledger`, `payment` |

Rules (enforced by ArchUnit or Spring Modulith tests, ticket EW-021):
- A module may call another only through that module's public API package (`..module.api`).
- No module reads another module's tables.
- `ledger` depends on nothing. It does not know what a "transfer" or "top-up" is; it only knows balanced entries.

## 3. Package layout inside a module

```
com.huyhk.wallet.<module>
├── api/            # public interfaces and DTOs other modules may use
├── domain/         # entities, value objects (Money), domain services, domain events — no framework imports
├── application/    # use cases, @Transactional boundaries, ports (interfaces)
└── infrastructure/ # JPA repositories, REST controllers, Stripe client, Kafka — adapters
```

## 4. Key flows

**P2P transfer (Phase 1–2, in-process)**

```
Client ──POST /transfers (Idempotency-Key)──> wallet.TransferController
  wallet.TransferUseCase  [one DB transaction]
    1. idempotency check (insert key, or return stored response)
    2. load both wallets, check frozen/limits
    3. ledger.post(entry: debit sender, credit receiver)   <- locks + balance check
    4. save transfer record, publish TransferCompleted (after commit)
notification listens to TransferCompleted -> sends email (async, outside the transaction)
```

**Card top-up (Phase 2–3)**

```
Client ──POST /topups──> payment: create Topup(PENDING) + Stripe PaymentIntent (idempotency key = topupId)
Client confirms the card with Stripe.js (card data never touches our servers)
Stripe ──webhook payment_intent.succeeded──> payment: verify signature, dedupe by event id,
        Topup PENDING -> SUCCEEDED, ledger.post(top-up entry), publish TopupSucceeded
```

## 5. Evolution by phase

| Phase | Shape | What changes |
|---|---|---|
| 1 (months 1–3) | Modular monolith, one PostgreSQL schema per module | Ledger, wallets, transfers, Flyway, Git flow |
| 2 (months 4–6) | Same, production-grade | Clean REST + OpenAPI, Problem Details, Testcontainers, ArchUnit, CI, Stripe top-up via `PaymentProvider` strategy |
| 3 (months 7–9) | `notification` and `payment` extracted as services; Kafka between them | Transactional outbox, idempotent consumers, saga for top-up, Resilience4j, Redis, OpenTelemetry, Docker Compose → Kubernetes → AWS |
| 4 (months 10–12) | Polished | Design doc, final ADRs, load test results, demo video |

The `ledger` and `wallet` modules stay together on purpose: a transfer must be atomic, and splitting them would force a distributed transaction for no business benefit. Defend this in an interview.

## 6. Diagram to draw yourself

Ticket EW-011 asks you to draw a C4 context and container diagram (draw.io, Excalidraw or Mermaid) and commit it to `docs/architecture/`.
