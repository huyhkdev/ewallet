# ADR-0005: Git flow branching and Conventional Commits

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

The target employer's job description asks for Git flow explicitly. Many product companies prefer trunk-based development. We want to practice what the job uses while understanding the alternative.

## Decision

- Git flow: `main` (released), `develop` (integration), `feature/EW-123-short-name`, `release/x.y.0`, `hotfix/x.y.z`.
- Feature branches merge into `develop` through a PR with `--no-ff` (merge commit) so each ticket is visible in history.
- A release branch is cut at the end of each phase, tagged `v0.<phase>.0`, merged into `main` and back into `develop`.
- Commit messages follow Conventional Commits: `feat(wallet): add transfer limits [EW-031]`.

## Consequences

- Positive: matches the JD, gives a clean release history per phase.
- Negative: more ceremony than trunk-based; long-lived branches cause merge pain. Mitigation: keep feature branches under one week.
- Revisit in Phase 4: write a short comparison with trunk-based development for interviews.
