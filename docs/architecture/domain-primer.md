# Domain primer: double-entry ledger for a wallet

Read this before touching any money code. Interviewers for fintech roles ask about it, and every design decision in this repo follows from it.

## 1. The core idea

A wallet company holds customers' money. From the **company's** point of view:

- Money a customer has in their wallet is a **liability**: the company owes it to them.
- Money sitting at Stripe or in a bank is an **asset**: the company has it.
- Fees the company earns are **revenue**; fees it pays (e.g. to Stripe) are **expenses**.

Double-entry bookkeeping records every movement as a **journal entry** made of two or more **postings**. Each posting hits one ledger account on the **debit** side or the **credit** side. For every journal entry:

```
sum(debit postings) = sum(credit postings)
```

So money is never created or destroyed by a bug; it can only move between accounts. If the sums ever disagree, you have a bug, and you can find it.

## 2. Debit and credit are sides, not "plus" and "minus"

| Account type | Increases with | Decreases with | Normal balance |
|---|---|---|---|
| Asset | Debit | Credit | Debit |
| Expense | Debit | Credit | Debit |
| Liability | Credit | Debit | Credit |
| Revenue | Credit | Debit | Credit |
| Equity | Credit | Debit | Credit |

A customer wallet is a liability, so **crediting** a wallet increases its balance.

## 3. Chart of accounts (v1)

| Code | Name | Type | One per |
|---|---|---|---|
| `ASSET:STRIPE_CLEARING:USD` | Money held at Stripe | Asset | currency |
| `LIABILITY:WALLET:{walletId}` | Customer wallet | Liability | wallet |
| `REVENUE:TOPUP_FEES:USD` | Top-up fee income | Revenue | currency |
| `EXPENSE:PROCESSING_FEES:USD` | Stripe processing fees | Expense | currency |
| `EQUITY:ADJUSTMENTS:USD` | Manual adjustments offset | Equity | currency |

System accounts (non-wallet) are created by a Flyway migration, not at runtime.

## 4. Worked examples

**Top-up of 100.00 USD.** Our fee is 2.9% of 100.00 + 0.30 = 3.20, so the card is charged 103.20.

| Account | Debit | Credit |
|---|---|---|
| ASSET:STRIPE_CLEARING:USD | 103.20 | |
| LIABILITY:WALLET:alice | | 100.00 |
| REVENUE:TOPUP_FEES:USD | | 3.20 |
| **Total** | **103.20** | **103.20** |

**Stripe charges us its processing fee of 3.29** (known later, from the Stripe balance transaction; posted by reconciliation in Phase 3).

| Account | Debit | Credit |
|---|---|---|
| EXPENSE:PROCESSING_FEES:USD | 3.29 | |
| ASSET:STRIPE_CLEARING:USD | | 3.29 |

**Alice sends 25.00 to Bob.**

| Account | Debit | Credit |
|---|---|---|
| LIABILITY:WALLET:alice | 25.00 | |
| LIABILITY:WALLET:bob | | 25.00 |

Note that a P2P transfer does not touch any asset: the company's total cash is unchanged, only who it owes changed.

**The transfer was a mistake and support reverses it** (via an approved adjustment). We never edit or delete the original entry. We post a new entry with the sides swapped and a reference to the original:

| Account | Debit | Credit |
|---|---|---|
| LIABILITY:WALLET:bob | 25.00 | |
| LIABILITY:WALLET:alice | | 25.00 |

## 5. Invariants the code must guarantee

1. Every journal entry balances, per currency.
2. A journal entry has at least two postings, all in the same currency (FX in v2 uses a pair of entries through an FX account).
3. Postings are append-only. No `UPDATE`, no `DELETE`. Consider enforcing this in the database too (permissions or a trigger) and write down why in an ADR.
4. A customer wallet balance never goes below zero. System accounts may.
5. The balance of any account equals the sum of its postings. If you also store a cached balance for speed (you probably will), a check must be able to prove the cache matches.
6. One business event produces exactly one journal entry, even under retries (idempotency).
7. Sum of all customer wallet balances = what the company owes customers. Reconciliation compares the asset side with Stripe.

## 6. Design questions you must answer yourself (ticket EW-010)

These are deliberately left open. Answer each in the ERD and in an ADR, with the trade-off.

- Do postings store a positive amount plus a `direction` (DEBIT/CREDIT), or a signed amount? What does each make easy or error-prone?
- Do you store a balance on the account row, compute it from postings, or both? If both, how do you keep them consistent under concurrent transfers?
- How do you prevent two concurrent transfers from both seeing 100 and both spending 80? (Pessimistic `SELECT ... FOR UPDATE`, optimistic `@Version`, or a single `UPDATE ... WHERE balance >= :amount`?) What about deadlocks when Alice pays Bob while Bob pays Alice?
- What is the primary key type of a journal entry, and what is its business key for idempotency?
- `NUMERIC(19,4)` or minor units in `BIGINT`? (ADR-0004 gives the project default; you may challenge it with a new ADR.)

## 7. Further reading

- Martin Fowler, "Accounting Patterns" (Account, Accounting Entry, Accounting Transaction).
- Modern Treasury, "Accounting for Developers" (parts 1–3).
- Square / Stripe / Uber engineering blogs on their ledgers ("Ledger", "LedgerStore").
