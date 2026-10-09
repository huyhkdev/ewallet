# Definition of Ready and Definition of Done

## Ready (before you start a ticket)

- [ ] Acceptance criteria are clear and testable. If not, ask the PO first.
- [ ] Dependencies (other tickets, ADRs) are done.
- [ ] You can estimate it. If it is over 8h, split it.

## Done (before a PR can merge)

- [ ] Every acceptance criterion is met and ticked in the PR description.
- [ ] Tests added: unit tests for domain logic, integration tests for persistence and API. CI is green.
- [ ] `money`-labelled tickets: the ledger invariant check runs in the new tests.
- [ ] Flyway migration added for any schema change; no edits to merged migrations.
- [ ] OpenAPI updated for any API change (from Phase 2).
- [ ] ADR merged in the same PR when the ticket is labelled `adr`.
- [ ] No TODO without a ticket id (`// TODO EW-045: ...`).
- [ ] No secrets, PII or tokens in code, logs or test fixtures.
- [ ] Reviewed and approved; all review comments resolved or answered.
- [ ] README / docs updated if behaviour or setup changed.
