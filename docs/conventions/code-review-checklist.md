# Code review checklist

Use this as the author before requesting review, and as the reviewer. Comments say **why**, and are prefixed: `blocker:`, `suggestion:`, `question:`, `nit:`.

## Correctness
- [ ] Does it do what the ticket asks, including the unhappy paths?
- [ ] Are transaction boundaries right? Any self-invocation, checked exception or swallowed exception that breaks rollback (day-1 traps)?
- [ ] Concurrency: what happens if this runs twice at the same time? If it is retried?

## Money (`money` label)
- [ ] `Money` everywhere, no `double`, explicit rounding.
- [ ] Every movement goes through `LedgerService.post`, balanced, append-only.
- [ ] Idempotent under retries and duplicate events.
- [ ] Invariant check in the tests.

## Design
- [ ] Module boundaries respected; domain free of framework code.
- [ ] Names say what things are; no "Manager"/"Helper"/"Util" dumping grounds.
- [ ] Simplest design that meets the requirement; no speculative abstraction.

## Security
- [ ] Authorization checked on the server for every resource (no trusting ids from the client).
- [ ] Input validated; no PII, secrets or tokens in logs or error bodies.

## Operability
- [ ] Logs and metrics would let you debug this at 2 a.m.
- [ ] Migration is safe on a table with data (no long locks, backwards compatible).

## Tests and docs
- [ ] Tests would fail if the feature broke. Edge cases covered.
- [ ] ADR / README / OpenAPI updated where needed.
