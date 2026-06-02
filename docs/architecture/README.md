# Architecture Documentation

This directory contains architectural documentation for the EWB-devops platform and the engineering principles that guide all EWB software projects.

---

## Contents

| Document | Description |
|---|---|
| [reusable-workflow-architecture.md](reusable-workflow-architecture.md) | Design and mechanics of the centralized GitHub Actions workflow platform |
| [repository-standards.md](repository-standards.md) | Standards for how all EWB project repositories should be structured and configured |

---

## Relationship to ADRs

Architecture documents describe the current state of the system — how it is designed and why things work the way they do. Architecture Decision Records (ADRs) document the specific decisions that led to the current state.

- For the history of a design decision, see [`docs/adr/`](../adr/)
- For the current authoritative design description, see this directory

---

## Keeping Documentation Current

Architecture documentation becomes a liability when it drifts from reality. When making a significant change to the platform:

1. Update the relevant document in this directory
2. If the change represents a significant architectural decision, add a new ADR
3. Include the documentation update in the same pull request as the code change
