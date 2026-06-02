# ADR-0001 — Use Centralized GitHub Actions via EWB-devops

**Date:** 2026-06-02
**Status:** Accepted
**Deciders:** EWB DevOps Team

---

## Context

As the number of EWB software projects grows, each project has historically maintained its own GitHub Actions workflow files. This approach has resulted in:

- Duplicated CI/CD configuration across 10+ repositories
- Inconsistent quality gate enforcement (some projects run tests, others do not)
- No standard for linting, security scanning, or build validation
- High maintenance burden when updating runner versions, action versions, or pipeline logic — requiring changes in every repository
- Slow onboarding of new projects, which must build pipelines from scratch

The organisation needs a scalable approach to CI/CD that enforces consistent standards while reducing per-project maintenance overhead.

---

## Decision

We will create and maintain a single repository — **EWB-devops** — that provides reusable GitHub Actions workflows using the `workflow_call` trigger. All EWB project repositories will consume these workflows rather than defining their own.

The reusable workflows will cover:
- QA pipelines (React, Node.js, .NET, Java)
- Security scanning (CodeQL, dependency review)
- Docker build
- SonarQube analysis
- Release automation

Each workflow will be versioned via git tags on EWB-devops. Consuming repositories reference a specific tag or `@main` depending on their stability requirements.

---

## Alternatives Considered

| Alternative | Reason Not Chosen |
|---|---|
| Each project maintains its own workflows | Does not scale; inconsistent standards; high maintenance cost |
| Copy a template workflow into each project manually | Diverges over time; updates require touching every repository |
| Use a third-party CI platform (CircleCI, Jenkins) | Introduces additional tooling and cost; GitHub Actions is already in use and well-integrated |
| GitHub Actions composite actions | Suitable for single-step reuse but not full pipeline reuse; `workflow_call` is the appropriate mechanism for full pipeline sharing |

---

## Consequences

**Positive:**
- Single workflow update in EWB-devops propagates to all consuming projects
- New projects adopt standardised pipelines in under 10 minutes
- Consistent quality gates enforced across all repositories
- Centralized visibility into pipeline health and standards compliance
- Clear ownership of DevOps tooling by the DevOps team

**Negative / Trade-offs:**
- EWB-devops must be set to Internal or Public visibility — workflows cannot be reused from Private repositories
- Consuming projects are coupled to EWB-devops; a breaking change in a workflow affects all consumers (mitigated by versioned tags)
- Teams lose the ability to customise pipeline behaviour without either forking workflows or adding inputs to EWB-devops (treated as a feature, not a limitation)
- A new dependency on the EWB-devops repository being available and maintained

---

## References

- [GitHub Docs — Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [EWB-devops Architecture](../architecture/reusable-workflow-architecture.md)
