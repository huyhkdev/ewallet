# Product Requirements Document — Wallet

| | |
|---|---|
| Owner (PO) | Claude |
| Engineering | huyhk |
| Status | Approved for Phase 1 |
| Last updated | 2026-10-08 |

## 1. Problem

Freelancers and small online sellers need a simple stored-value wallet: load money with a card, pay or send money to other users instantly and for free, and see a statement they can trust. The business must be able to prove, at any moment, where every cent is.

## 2. Users

| Persona | Needs |
|---|---|
| **Customer** | Top up, send money, see balance and history, get notified |
| **Ops agent** | Look up a customer, freeze a wallet, request a manual adjustment |
| **Finance / Compliance** | Reconcile with Stripe daily, approve adjustments, read the audit log |

## 3. Scope

### 3.1 In scope (v1)

**Identity & KYC**
- Register with email + password, log in, receive a JWT access token and refresh token.
- KYC tiers. `TIER_0` on registration. `TIER_1` after submitting name, date of birth and a (fake) ID number; approval is simulated.
- Limits per tier (configurable):

  | Tier | Max balance | Daily outgoing | Single transfer |
  |---|---|---|---|
  | TIER_0 | 200 USD | 100 USD | 50 USD |
  | TIER_1 | 10,000 USD | 2,000 USD | 1,000 USD |

**Wallets**
- One wallet per customer per currency. v1 supports USD only; the model must allow more currencies without schema changes.
- Balance = available balance. Pending top-ups are not spendable.
- Statement: paginated list of movements with running balance, filter by date range.

**Top-up (card)**
- Customer creates a top-up for an amount; the API returns a Stripe PaymentIntent client secret.
- Wallet is credited **only** when Stripe's signed `payment_intent.succeeded` webhook arrives.
- Fee: 2.9% + 0.30 USD, charged to the customer, posted to a fee revenue account.
- Duplicate, late or out-of-order webhooks must not double-credit.

**Peer-to-peer transfer**
- Send to another customer by email. Free. Instant.
- Requires an `Idempotency-Key` header. Same key + same body returns the original result; same key + different body returns 422.
- Rejected when: insufficient funds, limit exceeded, either wallet frozen, sender = receiver, amount ≤ 0 or more than 2 decimal places.

**Notifications**
- Email to the receiver on transfer received, and to the customer on top-up succeeded. Delivered asynchronously; a failed email never fails the transfer.

**Back office (API only)**
- Freeze / unfreeze a wallet.
- Manual adjustment (credit or debit) with **maker-checker**: one ops agent creates it, a different finance user approves it. Only then is it posted.
- Daily reconciliation job: compare Stripe balance transactions with ledger postings for the previous day, produce a report of mismatches.
- Audit log of every back-office action (who, what, when, before/after).

### 3.2 Later (v2, only if time allows)

- Withdrawal to a bank account (simulated payout provider).
- Multi-currency wallets and FX conversion.
- Second payment provider behind the same interface (proves the Strategy/Adapter design).
- Scheduled / recurring transfers.

### 3.3 Out of scope

- Real money, real KYC providers, real bank integrations.
- Storing card numbers or any card data (PCI DSS). Stripe Elements / test cards only.
- A production frontend. The API plus Swagger UI is the product surface. A tiny demo page for the top-up flow is allowed.
- Interest, credit, lending.

## 4. Non-functional requirements

| Category | Requirement |
|---|---|
| Correctness | For every journal entry, sum(debits) = sum(credits). A customer wallet balance never goes below zero. Enforced in code **and** checked by an automated invariant test |
| Exactly-once effect | Retrying any money-moving request or webhook never moves money twice |
| Immutability | Postings are never updated or deleted. Corrections are reversal entries |
| Auditability | Any balance can be recomputed from postings. Every back-office action is in the audit log |
| Concurrency | 50 concurrent transfers from the same wallet produce a correct final balance (load test proves it) |
| Performance | Transfer API p95 < 200 ms at 50 req/s on a laptop-sized setup |
| Security | OWASP Top 10 reviewed. Secrets from env/secret manager. No PII or tokens in logs |
| Observability | Every request has a correlation id visible in logs and traces. Business metrics: transfers/min, top-up success rate, reconciliation mismatches |
| Availability | Notification or Stripe outages degrade gracefully and never corrupt the ledger |

## 5. Success metrics (for the portfolio)

- A reviewer can clone, run `docker compose up`, and complete a top-up + transfer in under 10 minutes using the README.
- The ledger invariant test and the concurrency test are green in CI.
- ADRs explain every major decision with trade-offs.
- A 5-minute English demo video.

## 6. Open questions

Raise these with the PO when you reach them; do not guess silently.

1. Should a transfer to an unregistered email be held until the receiver signs up? (Default for v1: reject.)
2. Can a frozen wallet still **receive** money? (Default: yes, it cannot send.)
3. Which time zone defines a "day" for daily limits and reconciliation? (Default: UTC.)
