# Phase 2 — Production-grade API, tests, Stripe top-up, back office (months 4–6, sprints 7–12)

**Phase goal:** code another team would happily maintain. Clean REST API with a documented error model, real-database tests, enforced module boundaries, card top-up through Stripe test mode, and the first back-office features. From this phase on, PR descriptions and commit messages are in English only. Released as `v0.2.0`.

| Sprint | Tickets |
|---|---|
| 7 | EW-020, EW-021, EW-022 |
| 8 | EW-023, EW-024 |
| 9 | EW-025, EW-026 (start) |
| 10 | EW-026, EW-027 |
| 11 | EW-028, EW-029, EW-030 |
| 12 | EW-031, EW-032, EW-033, EW-034 |

---

## Epic E4 — API & code quality

### EW-020 · Error model and API versioning (ADR-0011)
task · E4 · Sprint 7 · 4h · adr
**Learning goal:** REST design, `ProblemDetail` (Spring 6), RFC 9457.
**Acceptance criteria**
- [ ] All errors are `application/problem+json` with `type`, `title`, `status`, `detail`, `instance`, plus a stable `code` (e.g. `INSUFFICIENT_FUNDS`) and `traceId`.
- [ ] Validation errors list every invalid field.
- [ ] Unknown exceptions return 500 without stack traces or SQL in the body.
- [ ] ADR-0011 covers error codes and versioning (URI vs header) with the chosen approach.

### EW-021 · Enforce module boundaries
task · E4 · Sprint 7 · 3h
**Learning goal:** ArchUnit, Spring Modulith, dependency rules.
**Acceptance criteria**
- [ ] Tests fail the build if a module uses another module outside its `api` package, if `domain` imports Spring/JPA, or if `ledger` depends on any other module.
- [ ] Introduce one violation on a branch, show the failing build in the PR, then remove it.

### EW-022 · OpenAPI documentation
task · E4 · Sprint 7 · 3h
**Acceptance criteria**
- [ ] springdoc-openapi, Swagger UI at `/swagger-ui.html` in `local` profile only.
- [ ] Every endpoint has a summary, request/response examples, and documented error codes.
- [ ] `Idempotency-Key` header documented on money endpoints. Security scheme configured so Swagger "Authorize" works.

### EW-023 · Test strategy with Testcontainers
task · E4 · Sprint 8 · 5h
**Learning goal:** JUnit 5, test slices, Testcontainers (Phase 2 Testing).
**Acceptance criteria**
- [ ] `docs/conventions/testing.md` updated with what each layer tests (unit / `@DataJpaTest` / `@WebMvcTest` / full `@SpringBootTest`).
- [ ] One shared PostgreSQL container for the integration suite (singleton container pattern); suite runs in CI.
- [ ] No H2 anywhere.
- [ ] Coverage report (JaCoCo) published as a CI artifact; `ledger` and `wallet.domain` ≥ 90% line coverage.

---

## Epic E5 — Card top-up (Stripe test mode)

### EW-024 · Payment provider abstraction (ADR-0012)
task · E5 · Sprint 8 · 6h · adr
**Learning goal:** Strategy and Adapter patterns, Dependency Inversion (SOLID).
**Description:** Define a `PaymentProvider` port in `payment.application` and a `StripePaymentProvider` adapter in `payment.infrastructure`. Add a `FakePaymentProvider` for tests and local runs without Stripe keys.
**Acceptance criteria**
- [ ] No Stripe class is visible outside `payment.infrastructure` (ArchUnit rule).
- [ ] Provider chosen by configuration; adding a second provider requires no change in application code.
- [ ] Money converted to minor units only in the Stripe adapter, with tests for USD and JPY.
- [ ] ADR-0012 explains the abstraction and what you deliberately did *not* abstract.

### EW-025 · Create a top-up
story · E5 · Sprint 9 · 5h · money
**Description:** As a customer I start a top-up for an amount and get a client secret to confirm the card payment.
**Acceptance criteria**
- [ ] `POST /api/v1/topups` (with `Idempotency-Key`) creates `Topup(PENDING)` and a Stripe PaymentIntent for amount + fee (PRD 3.1). Stripe idempotency key = topup id.
- [ ] Response contains topup id, amount, fee, total charged, client secret.
- [ ] Tier limits checked before contacting Stripe.
- [ ] If Stripe call fails, the top-up is marked `FAILED` and the client gets a clear error; no ledger entry exists.

### EW-026 · Handle Stripe webhooks safely
story · E5 · Sprints 9–10 · 8h · money, security
**Learning goal:** webhooks, signature verification, idempotent processing, out-of-order events (Phase 3 Fintech preview).
**Acceptance criteria**
- [ ] `POST /webhooks/stripe` verifies the `Stripe-Signature` header; invalid signature → 400 and nothing processed.
- [ ] Each Stripe event id is processed at most once (stored, unique constraint).
- [ ] `payment_intent.succeeded` → top-up `SUCCEEDED` and the top-up journal entry from the primer is posted, in one transaction.
- [ ] `payment_intent.payment_failed` → `FAILED`, no ledger entry.
- [ ] A `failed` event arriving after `succeeded` does not change a succeeded top-up (state machine test).
- [ ] Tested with Stripe CLI (`stripe listen`, `stripe trigger`) and documented in the README.
- [ ] Webhook endpoint returns 2xx quickly; explain in the PR what happens if processing is slow.

### EW-027 · Demo page for card top-up
task · E5 · Sprint 10 · 3h
**Acceptance criteria**
- [ ] One static page using Stripe Elements confirms a PaymentIntent with test card `4242 4242 4242 4242`.
- [ ] Card data goes directly to Stripe; our backend never receives it (explain PCI DSS scope in 3 sentences in the PR).

---

## Epic E6 — Notifications (in-process)

### EW-028 · Email notifications
story · E6 · Sprint 11 · 4h
**Learning goal:** Observer pattern via Spring events, `@TransactionalEventListener`, `@Async`.
**Acceptance criteria**
- [ ] Receiver gets an email on `TransferCompleted`; customer on `TopupSucceeded`. Visible in Mailpit.
- [ ] Listener runs after commit; an email failure never rolls back money.
- [ ] Explain in the PR what is lost if the app crashes between commit and sending (this motivates the outbox in Phase 3).

---

## Epic E7 — Back office

### EW-029 · Roles and wallet freeze
story · E7 · Sprint 11 · 4h · security
**Acceptance criteria**
- [ ] Roles `CUSTOMER`, `OPS`, `FINANCE`; admin endpoints under `/api/v1/admin/**` with method security.
- [ ] `OPS` can freeze/unfreeze a wallet with a reason. Frozen wallets cannot send (PRD open question 2 resolved and documented).

### EW-030 · Audit log
task · E7 · Sprint 11 · 4h
**Learning goal:** AOP / Decorator, Template Method.
**Acceptance criteria**
- [ ] Every admin action writes an append-only audit record: actor, action, target, before, after, timestamp, trace id.
- [ ] Implemented once (aspect or decorator), not copy-pasted per endpoint.
- [ ] `FINANCE` can query the audit log with filters.

### EW-031 · Manual adjustment with maker-checker
story · E7 · Sprint 12 · 6h · money
**Acceptance criteria**
- [ ] `OPS` creates an adjustment (credit/debit a wallet against `EQUITY:ADJUSTMENTS`) → `PENDING_APPROVAL`.
- [ ] A different user with `FINANCE` approves or rejects. The same person cannot do both (test).
- [ ] Only approval posts the journal entry. Reversal of a past transfer is an adjustment referencing it.
- [ ] Audited.

### EW-032 · Code quality gates in CI
chore · Sprint 12 · 3h
**Acceptance criteria**
- [ ] Formatter check (Spotless) and static analysis (SpotBugs or Error Prone) run in CI.
- [ ] Dependency vulnerability scan (OWASP dependency-check or GitHub Dependabot) enabled.

### EW-033 · SOLID self-review
spike · Sprint 12 · 3h
**Description:** Re-read your own Phase 1 code. Find one violation of each SOLID principle (or argue none exists), refactor at least two.
**Acceptance criteria**
- [ ] `docs/notes/solid-review.md` in English, with before/after snippets. This becomes interview material.

### EW-034 · Release v0.2.0
chore · Sprint 12 · 1h
Same steps as EW-019. Retro question: "Which test saved me from a real bug this phase?"
