# security-scan — Reusable Security Scan Workflow

**File:** `.github/workflows/security-scan.yml`

---

## Purpose

Runs automated security analysis using GitHub CodeQL and dependency review. Identifies common vulnerabilities including SQL injection, XSS, path traversal, insecure deserialization, and known vulnerable dependencies. Results are published to the repository's Security tab.

---

## When To Use

Use this workflow on any repository to:

- Detect security vulnerabilities in source code (CodeQL)
- Block pull requests that introduce known-vulnerable dependencies (Dependency Review)
- Meet security compliance requirements for production services

Recommended to run on every pull request targeting `main` or `develop`.

---

## Required Project Setup

| Requirement | Details |
|---|---|
| GitHub Advanced Security | Required for CodeQL on private repositories (included free for public repos) |
| Security tab enabled | Must be enabled in repository settings |
| Dependency graph enabled | Required for Dependency Review (`Settings → Code security → Dependency graph`) |

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `language` | string | `'javascript'` | CodeQL language(s): `javascript`, `csharp`, `java`, `python` |
| `working-directory` | string | `'.'` | Project root |
| `fail-on-severity` | string | `'high'` | Minimum severity to fail on: `low`, `moderate`, `high`, `critical` |

---

## Example Caller Workflow

```yaml
name: Security Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly scan every Monday at 02:00 UTC

jobs:
  security:
    uses: EWBHoldings/EWB-devops/.github/workflows/security-scan.yml@main
    with:
      language: 'javascript'
      fail-on-severity: 'high'
```

For a .NET project:

```yaml
jobs:
  security:
    uses: EWBHoldings/EWB-devops/.github/workflows/security-scan.yml@main
    with:
      language: 'csharp'
```

For multiple languages in a full-stack project:

```yaml
jobs:
  security:
    uses: EWBHoldings/EWB-devops/.github/workflows/security-scan.yml@main
    with:
      language: 'javascript,csharp'
```

---

## Pipeline Steps

1. Checkout source code
2. Initialize CodeQL for the specified language(s)
3. Autobuild the project
4. Perform CodeQL static analysis and publish results to Security tab
5. Run Dependency Review (pull requests only) — fails if new dependencies with vulnerabilities at or above `fail-on-severity` are introduced

---

## Viewing Results

- CodeQL findings: **Repository → Security → Code scanning alerts**
- Dependency alerts: **Repository → Security → Dependabot alerts**
- Dependency Review results: **Pull request → Checks tab**

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| CodeQL autobuild fails | Project requires custom build steps | Replace the `Autobuild` step with explicit build commands in a fork of this workflow |
| Dependency Review step skipped | Workflow not triggered by a pull request | Expected behaviour — Dependency Review only runs on PR events |
| `Advanced Security is not enabled` error | Private repo without GitHub Advanced Security licence | Enable GitHub Advanced Security or restrict CodeQL to public repositories |
| False positive alerts | CodeQL flagging intentional patterns | Dismiss with a reason in the Security tab; do not suppress in code unless unavoidable |
