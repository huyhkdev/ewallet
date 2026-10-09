# Phase 4 — Portfolio and interview kit (months 10–12, sprints 20–26)

**Phase goal:** make the project tell your story to a hiring manager in 5 minutes and survive a 45-minute deep dive. Job applications start around now, so project time shrinks; stretch features are optional. Released as `v1.0.0`.

### EW-060 · README for strangers
task · 3h
**Acceptance criteria**
- [ ] A friend who has never seen the project runs it from a clean machine using only the README, in under 10 minutes. Fix every place they got stuck.
- [ ] README has architecture diagram, key decisions with links to ADRs, and "What I would do next".

### EW-061 · System design document
task · 6h · adr
**Acceptance criteria**
- [ ] `docs/design/system-design.md` in English: requirements, capacity estimates, architecture, data model, money flows, failure modes, trade-offs, what changes at 100× scale.
- [ ] You can present it in 40 minutes without notes (roadmap month-12 goal). Record one run.

### EW-062 · Game day and postmortem
task · 4h
**Description:** Break the system on purpose: stop Kafka during top-ups, replay 100 duplicate webhooks, kill the payment service mid-saga, fill the DB connection pool.
**Acceptance criteria**
- [ ] For each experiment: expected behaviour, observed behaviour, how you detected it (dashboards/traces).
- [ ] One blameless postmortem in English in `docs/postmortems/`.

### EW-063 · Demo video
task · 4h
**Acceptance criteria**
- [ ] 5-minute English video: the problem, a live top-up and transfer, the ledger view, one hard decision you made and why.
- [ ] Linked from README and CV.

### EW-064 · Interview kit from this project
task · 3h
**Acceptance criteria**
- [ ] `docs/notes/interview-kit.md`: 15 questions an interviewer would ask about this project (ledger, locking, idempotency, outbox, saga, why monolith first, what you would change) with your spoken-length English answers.
- [ ] 3 STAR stories drawn from this project (a bug you found, a decision you reversed, a trade-off you defended).

### EW-065 · Release v1.0.0
chore · 1h
Same steps as EW-019. Update the CV line for this project with measured results (p95, throughput, test count, invariant guarantees).

---

## Stretch (v2, only if time allows)

| Ticket | Title | Estimate | Practices |
|---|---|---|---|
| EW-070 | Withdrawal to bank via a simulated payout provider (async, can fail days later) | 8h | State machines, holds/reservations in the ledger |
| EW-071 | Multi-currency wallets and FX conversion through an FX account | 8h | Money across currencies, rounding, rate snapshots |
| EW-072 | Second payment provider behind `PaymentProvider` | 5h | Proves the Phase 2 abstraction |
| EW-073 | Read model for statements (CQRS) | 6h | When CQRS is and is not worth it |
