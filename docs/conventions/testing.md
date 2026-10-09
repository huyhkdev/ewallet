# Testing conventions

| Layer | Tool | What it proves | Speed |
|---|---|---|---|
| Domain unit | JUnit 5 + AssertJ | Rules and invariants (`Money`, limits, ledger balancing) | ms |
| Application | JUnit + Mockito or in-memory fakes | Use case orchestration, error paths | ms |
| Persistence | `@DataJpaTest` + Testcontainers PostgreSQL | Mappings, queries, constraints, locking | s |
| Web | `@WebMvcTest` | Validation, status codes, error format, security rules | s |
| Integration | `@SpringBootTest` + Testcontainers | Whole flows: transfer, top-up webhook | s |
| Architecture | ArchUnit / Spring Modulith | Module boundaries | ms |
| Concurrency | Real PostgreSQL, `ExecutorService` | No lost updates, no deadlocks | s |
| Contract (Phase 3) | Spring Cloud Contract / Pact | Event and API compatibility between services | s |

Rules:
- Test names describe behaviour: `rejectsTransferWhenBalanceIsInsufficient`.
- Arrange data through the public API or builders, not by inserting rows that bypass invariants.
- Never use H2 for anything that touches the ledger: locking and `NUMERIC` behave differently.
- Every integration test that moves money ends with the ledger invariant check (EW-018).
- A bug fix starts with a failing test that reproduces it.
