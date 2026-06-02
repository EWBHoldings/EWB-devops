# QA Standards

This document defines the quality standards that all EWB software projects are expected to meet before code is merged into `main` or `develop`.

---

## Overview

Quality is a shared responsibility. These standards apply to all pull requests across all technology stacks. The QA pipeline enforced by EWB-devops automates as many of these checks as possible, but some require human judgement during code review.

---

## Pipeline Gates

Every pull request must pass the automated QA pipeline before merging. A failing pipeline blocks the merge.

| Gate | Required | Notes |
|---|---|---|
| Build passes | Yes | No compilation or bundling errors |
| Unit tests pass | Yes | All tests must be green |
| Lint passes | Yes (if configured) | No lint errors; warnings reviewed case by case |
| No critical security alerts | Yes | CodeQL / dependency scan (where configured) |

---

## Code Quality

### General

- Code must compile or build without errors
- No commented-out code blocks committed to shared branches
- No hardcoded credentials, secrets, or environment-specific values
- Environment configuration via environment variables or secrets management

### Naming

- Follow the naming conventions of the language and framework in use
- Variable, function, and class names must be descriptive and unambiguous

### Complexity

- Functions and methods should have a single clear responsibility
- Avoid deeply nested conditionals — prefer early returns or extracted methods
- Files exceeding 400 lines are a signal to review structure

---

## Testing Standards

### Unit Tests

Unit tests are **required** for all non-trivial logic.

| Requirement | Detail |
|---|---|
| Coverage target | Aim for meaningful coverage of business logic |
| Test naming | Describe what is being tested and the expected outcome |
| Independence | Tests must not depend on execution order or external state |
| Speed | Unit tests must complete in seconds, not minutes |

### Integration Tests

Integration tests are **strongly encouraged** for:

- Database interactions
- External API calls
- Authentication and authorisation flows
- Message queue producers and consumers

### What Not to Test

- Framework internals (e.g. ASP.NET routing, React rendering)
- Auto-generated code
- Simple getters and setters with no logic

---

## Code Review Standards

### Reviewer Responsibilities

- Verify the change does what the PR description claims
- Check for logical errors, not just style
- Confirm tests cover the new or changed behaviour
- Flag any security concerns
- Ensure no breaking changes are undocumented

### Author Responsibilities

- PR description clearly explains the change and the reason for it
- PR is scoped to one logical change — avoid mixing features and refactors
- All checklist items in the PR template are completed
- Reviewer comments are addressed before requesting re-review

### Review Turnaround

- Authors should respond to review comments within **1 business day**
- Reviewers should complete reviews within **1 business day** of being requested

---

## Security Standards

- No secrets or credentials committed to any branch
- Dependencies must not have known critical vulnerabilities (CVSS score ≥ 9.0)
- Input validation applied at all system boundaries
- Authentication and authorisation verified on all protected routes
- Sensitive data must not appear in logs

---

## Documentation Standards

- Public APIs and exported functions must have summary documentation
- Non-obvious decisions must include an inline comment explaining the why
- New configuration options must be documented in the project README or relevant doc file
- Architecture changes must be reflected in `docs/architecture.md`

---

## Definition of Done

A task or feature is considered **done** when all of the following are true:

- [ ] Implementation is complete and matches the acceptance criteria
- [ ] Unit tests written and passing
- [ ] Integration tests written where appropriate
- [ ] QA pipeline passes (build, lint, test)
- [ ] Pull request reviewed and approved by at least one reviewer
- [ ] No known defects introduced
- [ ] Relevant documentation updated
- [ ] Merged to `develop` (or `main` for hotfixes)
