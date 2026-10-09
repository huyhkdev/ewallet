# Phase 1 — Foundation, ledger, transfers (months 1–3, sprints 1–6)

**Phase goal:** a modular monolith where customers can register, get a USD wallet, and send money to each other through a correct, concurrent-safe double-entry ledger. Released as `v0.1.0`.

**Suggested sprint plan**

| Sprint | Tickets |
|---|---|
| 1 | EW-001, EW-002, EW-003, EW-004, EW-005, EW-006 |
| 2 | EW-007, EW-010, EW-011, EW-012 |
| 3 | EW-008, EW-009 |
| 4 | EW-013 |
| 5 | EW-014, EW-015 |
| 6 | EW-016, EW-017, EW-018, EW-019 |

---

## Epic E0 — Project foundation

### EW-001 · Create the repository and protect the main branches
chore · E0 · Sprint 1 · 1h · infra
**Learning goal:** Git flow (Phase 1).
**Description:** Create the GitHub repository `ewallet`, push the day-1 project and these docs. Set up `main` and `develop` as protected branches.
**Acceptance criteria**
- [ ] Repository contains the Spring Boot project, `docs/`, `backlog/`, `.github/` templates.
- [ ] `main` and `develop` require a PR to merge; direct pushes blocked.
- [ ] Default branch is `develop`.
- [ ] Repository description and topics set (java, spring-boot, fintech, ledger).

### EW-002 · Bootstrap the application
task · E0 · Sprint 1 · 2h
**Learning goal:** Spring Boot auto-configuration basics.
**Description:** The day-1 lesson created `com.huyhk:wallet`. Make it a proper base: Maven wrapper committed, Java 21 toolchain enforced, `spring-boot-starter-actuator` with `/actuator/health`, `application.yml` split into `default`, `local` and `test` profiles.
**Acceptance criteria**
- [ ] `./mvnw verify` passes on a clean clone.
- [ ] `GET /actuator/health` returns `UP` with DB status.
- [ ] No credentials in `application.yml`; local values come from the `local` profile or env vars.
- [ ] `.editorconfig` and `.gitignore` present.

### EW-003 · Local infrastructure with Docker Compose
task · E0 · Sprint 1 · 1h · infra
**Description:** Use `infra/docker-compose.yml` (PostgreSQL + Mailpit) instead of the `docker run` from day 1.
**Acceptance criteria**
- [ ] `docker compose -f infra/docker-compose.yml up -d` starts PostgreSQL 16 and Mailpit with health checks.
- [ ] The day-1 `wallet-db` container is stopped and removed first (same port 5432); credentials in the `local` profile updated to match Compose.
- [ ] README "Running locally" works exactly as written.

### EW-004 · Module package skeleton
task · E0 · Sprint 1 · 1h
**Learning goal:** Hexagonal architecture (preview of Phase 2).
**Description:** Create the module packages from the [architecture overview](../docs/architecture/overview.md#3-package-layout-inside-a-module): `identity`, `ledger`, `wallet`, `payment`, `notification`, `backoffice`, each with `api/domain/application/infrastructure`. Add a `package-info.java` in each module root describing its responsibility in one sentence.
**Acceptance criteria**
- [ ] Packages exist and compile.
- [ ] Each module has a one-line responsibility in `package-info.java`.

### EW-005 · Move the day-1 lab code out of the product
chore · E0 · Sprint 1 · 1h
**Description:** The `Account`/`TransferService` from the `@Transactional` lesson are experiments, not product code. Move them to `com.huyhk.wallet.labs` (and their tests). Future lessons that need throwaway code also go into `labs`.
**Acceptance criteria**
- [ ] No product module depends on `labs`.
- [ ] Lab tests still run and still demonstrate the three traps.

### EW-006 · Flyway and schema per module
task · E0 · Sprint 1 · 3h · adr-reviewed (ADR-0006)
**Learning goal:** DB migrations, DB design (Phase 1).
**Description:** Replace `ddl-auto: update` with Flyway. One schema per module.
**Acceptance criteria**
- [ ] `V1__create_schemas.sql` creates the six module schemas.
- [ ] `spring.jpa.hibernate.ddl-auto=validate` in all profiles.
- [ ] App fails fast on startup if an entity does not match the table.
- [ ] You can explain in the PR description why editing a merged migration is forbidden.

### EW-007 · Continuous integration
task · E0 · Sprint 2 · 2h · infra
**Learning goal:** CI/CD with GitHub Actions (Phase 3 preview).
**Description:** A GitHub Actions workflow that runs `./mvnw -B verify` on every PR to `develop` and `main`.
**Acceptance criteria**
- [ ] Workflow runs on PR and push, caches Maven dependencies.
- [ ] Branch protection requires the check to pass.
- [ ] A deliberately failing test blocks the merge (prove it once, then revert).

---

## Epic E1 — Identity & KYC

### EW-008 · Registration and login with JWT
story · E1 · Sprint 3 · 8h · adr, security
**Learning goal:** Spring Security filter chain, JWT pitfalls.
**Description:** As a customer I can register with email and password and log in to receive an access token, so that only I can use my wallet. Write ADR-0010 on the token strategy (lifetime, signing key, refresh token storage, logout).
**Acceptance criteria**
- [ ] `POST /api/v1/auth/register`, `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`.
- [ ] Passwords hashed with BCrypt or Argon2; never logged.
- [ ] Email unique case-insensitively (DB constraint, not only code).
- [ ] Every other endpoint requires a valid token; the current user comes from the token, never from a request parameter.
- [ ] Integration test: a user cannot read another user's wallet (403/404, explain which and why in the ADR).
- [ ] ADR-0010 merged.

### EW-009 · KYC tier upgrade
story · E1 · Sprint 3 · 3h
**Description:** As a customer I can submit my full name, date of birth and ID number to move from `TIER_0` to `TIER_1`. Approval is simulated: ID numbers ending in `0` are rejected, others approved.
**Acceptance criteria**
- [ ] `POST /api/v1/me/kyc` stores the submission and the decision.
- [ ] Customers under 18 are rejected.
- [ ] ID number is masked in API responses and logs (`*****1234`).
- [ ] Tier change publishes a `KycTierChanged` domain event (used by limits later).

---

## Epic E2 — Ledger core

### EW-010 · Design the ledger schema (ERD + ADR-0007)
spike · E2 · Sprint 2 · 4h · adr, money
**Learning goal:** DB design for money (Phase 1 "Thiết kế DB" exercise).
**Description:** Read the [domain primer](../docs/architecture/domain-primer.md). Design the tables for ledger accounts, journal entries and postings, and how wallets reference ledger accounts. Answer every question in section 6 of the primer. **Design only; no code.** Get the design reviewed before starting EW-013.
**Acceptance criteria**
- [ ] ERD committed to `docs/architecture/ledger-erd.(png|md)` with every column, type, constraint and index.
- [ ] ADR-0007 compares at least two options for posting representation and for balance storage.
- [ ] The ERD shows how invariants 1–6 of the primer are enforced (DB constraint, code, or test).
- [ ] Reviewer approved.

### EW-011 · C4 diagrams
task · E2 · Sprint 2 · 2h
**Description:** Draw a C4 Context and Container diagram for v1 (actors: customer, ops, finance, Stripe, email).
**Acceptance criteria**
- [ ] Diagrams committed in `docs/architecture/` and linked from the overview.

### EW-012 · Money value object
task · E2 · Sprint 2 · 3h · money
**Learning goal:** Java records, `BigDecimal`, immutability (Effective Java).
**Description:** Implement `Money` per ADR-0004 in a shared `common` package (no framework imports).
**Acceptance criteria**
- [ ] `of(String, Currency)`, `plus`, `minus`, `negate`, `isNegative`, `isGreaterThanOrEqual`, `percentage(BigDecimal, RoundingMode)`.
- [ ] Mixing currencies throws a specific exception.
- [ ] Tests cover: `0.1 + 0.2 = 0.3`, scale normalization (`1.0` vs `1.00`), JPY with 0 decimals, rounding of 2.9% of 0.35, `null` inputs.
- [ ] JSON serialization as a string amount + currency, tested.

### EW-013 · Post balanced journal entries
story · E2 · Sprint 4 · 8h · money
**Learning goal:** JPA mapping, transactions, constraints.
**Description:** Implement the ledger module from your approved ERD. Expose `LedgerService.post(JournalEntryRequest)` in `ledger.api`. Create system accounts (primer section 3) with a Flyway migration.
**Acceptance criteria**
- [ ] Unbalanced, single-posting, mixed-currency, and zero-amount entries are rejected with distinct exceptions.
- [ ] An entry that would make a customer wallet account negative is rejected and nothing is written.
- [ ] Postings cannot be updated or deleted through the code (no setters, no delete method) **and** the decision for DB-level protection is documented.
- [ ] Balance query for an account is correct after 1,000 random balanced postings (property-style test).
- [ ] The `ledger` package imports nothing from `wallet`, `payment` or `identity`.

---

## Epic E3 — Wallets & transfers

### EW-014 · Concurrency control for balances (ADR-0008)
spike + task · E3 · Sprint 5 · 6h · adr, money
**Learning goal:** isolation levels, optimistic vs pessimistic locking, deadlocks (Phase 1 JPA/PostgreSQL).
**Description:** The day-1 lesson ended with an open bug: two concurrent transfers can both read 100. Solve it in the ledger.
**Acceptance criteria**
- [ ] A test starts 50 threads that each move 10 out of a wallet holding 300; exactly 30 succeed, final balance is 0, no posting is lost.
- [ ] A test where A→B and B→A run concurrently 100 times finishes without a deadlock error (or retries it correctly, and the ADR says why).
- [ ] ADR-0008 compares at least: `SELECT ... FOR UPDATE`, `@Version` optimistic locking with retry, conditional `UPDATE ... WHERE balance >= ?`. States the chosen isolation level.
- [ ] Tests run against real PostgreSQL (Testcontainers or the Compose DB), not H2. Explain why in the PR.

### EW-015 · Wallet creation, balance and statement
story · E3 · Sprint 5 · 5h
**Description:** As a customer I get a USD wallet automatically when I register, and I can see my balance and a statement.
**Acceptance criteria**
- [ ] Wallet + its ledger account created in reaction to user registration (no direct call from `identity` into `wallet` internals).
- [ ] `GET /api/v1/wallets/me` returns balance as a string amount + currency.
- [ ] `GET /api/v1/wallets/me/statement?from&to&cursor&size` returns entries newest first with running balance, cursor pagination, max size 100.
- [ ] No N+1 queries on the statement endpoint (prove with SQL log or Hibernate statistics in a test).

### EW-016 · Peer-to-peer transfer with idempotency (ADR-0009)
story · E3 · Sprint 6 · 8h · adr, money
**Learning goal:** idempotency keys, transactional boundaries, `@Transactional` traps from day 1.
**Description:** As a customer I can send money to another customer by email. Clients must send `Idempotency-Key` so that retries never double-send.
**Acceptance criteria**
- [ ] `POST /api/v1/transfers` with `{ "toEmail", "amount", "currency", "note" }` → `201` with the transfer resource.
- [ ] Missing `Idempotency-Key` → `400`. Same key + same body → original response (same status and body). Same key + different body → `422`. Keys are scoped per user and expire after 24h.
- [ ] Two simultaneous requests with the same new key result in exactly one transfer.
- [ ] All rejection rules in PRD 3.1 are implemented and each has a test.
- [ ] Transfer record, idempotency record and ledger entry commit in one transaction; the `TransferCompleted` event is published only after commit.
- [ ] ADR-0009 merged.

### EW-017 · Limits by KYC tier
story · E3 · Sprint 6 · 4h · money
**Learning goal:** Strategy pattern preview, configuration properties.
**Description:** Enforce PRD limits (max balance, daily outgoing, single transfer). Limits are configured, not hard-coded.
**Acceptance criteria**
- [ ] Limits bound from `application.yml` with `@ConfigurationProperties` and validated on startup.
- [ ] Daily outgoing is computed for the UTC day and includes the current transfer.
- [ ] Receiver's max balance is checked too.
- [ ] Error responses say which limit was hit, without leaking other users' data.

### EW-018 · Ledger invariant check
task · E3 · Sprint 6 · 2h · money
**Description:** A query and a test helper that verify, across the whole database: every journal entry balances; every cached balance (if any) equals the sum of its postings; no customer account is negative.
**Acceptance criteria**
- [ ] Helper is called at the end of every integration test that moves money.
- [ ] Exposed as an admin-only actuator endpoint or scheduled job that logs an ERROR on violation.

### EW-019 · Release v0.1.0
chore · Sprint 6 · 1h
**Acceptance criteria**
- [ ] `release/0.1.0` cut from `develop`, version bumped, `CHANGELOG.md` updated, merged into `main` and back into `develop`, tagged `v0.1.0`.
- [ ] Sprint 6 retro includes a short English paragraph: "What I would design differently in the ledger now".
