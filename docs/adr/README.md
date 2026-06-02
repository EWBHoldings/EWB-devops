# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) for the EWB-devops platform and, by convention, for all EWB software projects.

---

## What Is an ADR?

An Architecture Decision Record documents a significant architectural or engineering decision made during the development of a project. It captures:

- **What** decision was made
- **Why** it was made (context and forces at play)
- **What the consequences are** (trade-offs accepted)

ADRs are immutable once accepted. If a decision is reversed, a new ADR is created superseding the old one — the original is never deleted or edited.

---

## Why We Use ADRs

Large decisions made verbally or in Slack are forgotten, misunderstood, or rediscovered through painful re-debate. ADRs create a durable record that:

- Explains the reasoning behind the current state of the system
- Helps new team members understand why things are the way they are
- Prevents relitigating settled decisions
- Provides context for future change

---

## When To Write an ADR

Write an ADR when making a decision that:

- Is difficult to reverse
- Has significant impact on multiple teams or systems
- Involves a meaningful trade-off between alternatives
- Would benefit from a written rationale visible to future maintainers

You do **not** need an ADR for every implementation detail or low-risk decision.

Examples of ADR-worthy decisions:

- Selecting a centralized CI/CD strategy
- Choosing a primary cloud provider or deployment target
- Adopting a specific architectural pattern (Clean Architecture, CQRS, Event Sourcing)
- Deciding on a programming language or framework for a new service
- Selecting an authentication mechanism

---

## ADR Lifecycle

| Status | Meaning |
|---|---|
| `Proposed` | Under discussion — not yet accepted |
| `Accepted` | Decision made and in effect |
| `Deprecated` | Was accepted but is no longer the current approach |
| `Superseded by ADR-XXXX` | Replaced by a newer decision |

---

## How To Write an ADR

1. Copy `adr-template.md` to a new file named `NNNN-short-title.md` (e.g. `0002-adopt-event-driven-architecture.md`)
2. Fill in all sections
3. Set status to `Proposed`
4. Open a pull request for team review and discussion
5. Update status to `Accepted` on merge

---

## Index

| ADR | Title | Status |
|---|---|---|
| [0001](0001-use-centralized-github-actions.md) | Use Centralized GitHub Actions via EWB-devops | Accepted |
