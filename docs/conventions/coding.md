# Coding conventions

## General

- Java 21. Prefer `record` for value objects and DTOs, `sealed` types for closed hierarchies (e.g. transfer outcomes), `var` only when the type is obvious from the right-hand side.
- Constructor injection only. No field `@Autowired`. Dependencies are `private final`.
- Domain classes (`..domain..`) have no Spring, JPA or Jackson annotations. Map to JPA entities in `infrastructure`, or accept JPA annotations on aggregates and write down that trade-off in an ADR; pick one approach and apply it everywhere.
- No `null` returns from public methods; use `Optional` for "may be absent" lookups, exceptions for violations.
- Exceptions: one base `DomainException` with a stable `code`; specific subclasses (`InsufficientFundsException`). Unchecked. Remember the day-1 lesson about rollback rules.
- `@Transactional` only on application-layer use case methods, `public`, never on controllers or repositories you write. Read-only queries use `@Transactional(readOnly = true)`.

## Money

- Always `Money` (ADR-0004). Raw `BigDecimal` for amounts is a review blocker.
- Compare with `compareTo` or `Money` methods, never `equals` on `BigDecimal`.
- Every rounding call names the `RoundingMode`.

## Naming

| Thing | Convention | Example |
|---|---|---|
| Use case | verb + noun + `UseCase` | `TransferMoneyUseCase` |
| Port (outbound) | noun + role | `PaymentProvider`, `WalletRepository` |
| Adapter | technology + port | `StripePaymentProvider`, `JpaWalletRepository` |
| Domain event | past tense | `TransferCompleted` |
| REST DTO | `...Request` / `...Response` | `CreateTransferRequest` |
| Table | snake_case plural, in module schema | `ledger.postings` |
| Migration | `V<n>__<verb>_<what>.sql` | `V7__create_postings.sql` |

## Logging

- SLF4J with placeholders, never string concatenation.
- Log ids (`walletId`, `transferId`), never emails, names, ID numbers, tokens, card data.
- `INFO` for business events, `WARN` for handled anomalies, `ERROR` only when someone should be woken up.

## Formatting

- Google Java Format via Spotless (enforced from EW-032). `.editorconfig` in the repo root.
