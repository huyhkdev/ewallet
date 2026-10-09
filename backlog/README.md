# Backlog

Tickets are grouped by phase. IDs are permanent (`EW-NNN`); gaps are reserved for tickets added later, including change requests from the PO.

| File | Months | Release | Theme | Estimate |
|---|---|---|---|---|
| [phase-1.md](phase-1.md) | 1–3 | `v0.1.0` | Foundation, ledger, transfers | ~65h |
| [phase-2.md](phase-2.md) | 4–6 | `v0.2.0` | Production-grade API, tests, Stripe top-up, back office | ~62h |
| [phase-3.md](phase-3.md) | 7–9 | `v0.3.0` | Events, services, resilience, observability, cloud | ~87h |
| [phase-4.md](phase-4.md) | 10–12 | `v1.0.0` | Portfolio, design doc, interview kit | ~30h + stretch |

Capacity: about 6h per week on the project, 2-week sprints of ~12h. Estimates include learning time for a 2-year developer.

## Ticket format

```
### EW-NNN · Title
Type · Epic · Sprint · Estimate · Labels
Learning goal: which roadmap topic this practices
Description: why and what
Acceptance criteria: checklist; the ticket is done when every box is ticked and the Definition of Done is met
```

Types: `story` (user-visible), `task` (technical), `spike` (research, output is a document), `chore`.
Labels: `adr` (the PR must include an ADR), `money` (extra review rules apply), `security`, `infra`.

## Workflow states

`Backlog → Ready → In progress → In review → Done`

A ticket is **Ready** when its acceptance criteria are clear to you. If not, ask the PO before starting. See [Definition of Ready / Done](../docs/conventions/definition-of-done.md).

## When GitHub is set up

Each ticket becomes a GitHub Issue with the same ID in the title, grouped in a GitHub Project board with the columns above and a milestone per phase.
