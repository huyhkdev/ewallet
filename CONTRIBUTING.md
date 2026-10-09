# Contributing

## Workflow (Git flow, ADR-0005)

1. Pick a **Ready** ticket from the sprint. Move it to *In progress*.
2. Branch from `develop`: `git switch develop && git pull && git switch -c feature/EW-016-p2p-transfer`.
3. Commit small, meaningful steps with [Conventional Commits](https://www.conventionalcommits.org/):
   ```
   feat(wallet): reject transfers to self [EW-016]
   test(ledger): reproduce lost update under concurrency [EW-014]
   fix(payment): ignore failed event after success [EW-026]
   docs(adr): add ADR-0008 concurrency control [EW-014]
   ```
   Types: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`, `ci`, `perf`, `build`.
4. Rebase on `develop` before opening the PR if it moved (`git rebase develop`), never after review has started on shared commits.
5. Open a PR to `develop` using the template. Title: `EW-016: P2P transfer with idempotency`.
6. CI green + one approval → merge with a merge commit (`--no-ff`). Delete the branch.
7. Releases: see EW-019.

Hotfix: branch `hotfix/0.1.1` from `main`, fix, merge to `main` (tag) and `develop`.

## Before requesting review

Go through [code-review-checklist.md](docs/conventions/code-review-checklist.md) and the [Definition of Done](docs/conventions/definition-of-done.md) yourself.

## Conventions

- [Coding](docs/conventions/coding.md)
- [REST API](docs/conventions/api-guidelines.md)
- [Testing](docs/conventions/testing.md)
