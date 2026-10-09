# ADR-0004: Money representation

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

Floating point cannot represent 0.10 exactly. Currencies have different numbers of decimal places (USD 2, JPY 0, KWD 3). Stripe's API uses integer minor units.

## Options considered

1. `double` — rejected, rounding errors.
2. `long` minor units everywhere — exact and fast, but every developer must remember the currency's exponent; easy to mix cents and dollars.
3. `BigDecimal` + currency in a `Money` value object, `NUMERIC(19,4)` in PostgreSQL, conversion to minor units only inside the Stripe adapter.

## Decision

Option 3.

- `Money` is an immutable value object (a Java `record`) holding `BigDecimal amount` and `java.util.Currency currency`.
- Arithmetic between different currencies throws.
- Amounts are normalized to the currency's default fraction digits on creation; rounding mode is `HALF_EVEN` (banker's rounding) and is never implicit — every rounding call names the mode.
- Compare with `compareTo`, never `equals` (`1.0` and `1.00` are not `equals`).
- Columns: `amount NUMERIC(19,4) NOT NULL`, `currency CHAR(3) NOT NULL`.
- JSON: amounts are serialized as **strings** (`"12.50"`) to avoid JavaScript float parsing.

## Consequences

- Positive: exact, readable, supports any ISO currency.
- Negative: `BigDecimal` is slower and verbose; developers must use `compareTo`. A unit test suite for `Money` (EW-012) is mandatory.
