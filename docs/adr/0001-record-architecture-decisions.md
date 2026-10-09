# ADR-0001: Record architecture decisions

- Status: Accepted
- Date: 2026-10-08
- Deciders: huyhk, Claude

## Context

Decisions made today will be questioned in six months by reviewers, interviewers and our future selves. Without a record, the reasoning is lost and decisions get re-argued.

## Decision

We record every significant decision as a short Markdown ADR in `docs/adr/`, numbered sequentially, using [0000-template.md](0000-template.md). "Significant" means: hard to reverse, affects more than one module, or a reasonable engineer would have chosen differently.

ADRs are immutable once Accepted. To change a decision, write a new ADR that supersedes the old one and update the old one's status line only.

## Consequences

- Every ticket labelled `adr` produces an ADR in the same PR as the code.
- ADRs are written in English and reviewed like code.
