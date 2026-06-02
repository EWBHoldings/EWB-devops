# sonarqube — Reusable SonarQube Analysis Workflow

**File:** `.github/workflows/sonarqube.yml`

---

## Purpose

Runs SonarQube static analysis and enforces the configured Quality Gate. Reports code quality metrics including code smells, bugs, vulnerabilities, code coverage, and duplications. Fails the pipeline if the Quality Gate is not passed.

---

## When To Use

Use this workflow for projects where:

- The organisation operates a SonarQube server (self-hosted or SonarCloud)
- Code quality and coverage reporting is required
- Quality Gate enforcement is part of the merge process

---

## Required Setup

### 1. SonarQube Server

You must have access to a running SonarQube instance or SonarCloud. Create a project in SonarQube and note the **project key**.

### 2. GitHub Secrets

Add the following to the repository (or organisation) secrets:

| Secret | Description |
|---|---|
| `SONAR_TOKEN` | Authentication token from SonarQube (User → My Account → Security) |
| `SONAR_HOST_URL` | Full URL to your SonarQube instance (e.g. `https://sonar.yourcompany.com`) |

### 3. sonar-project.properties (optional)

For advanced configuration, add a `sonar-project.properties` file to your project root. Any properties set there will be merged with the inputs passed to this workflow.

---

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `project-key` | string | Yes | SonarQube project key |
| `project-name` | string | No | Display name (defaults to project key) |
| `working-directory` | string | No | Project root directory |
| `sonar-host-url` | string | No | Server URL (if not using `SONAR_HOST_URL` secret) |

## Secrets

| Secret | Required | Description |
|---|---|---|
| `SONAR_TOKEN` | Yes | SonarQube authentication token |
| `SONAR_HOST_URL` | No | Server URL as a secret (takes precedence over input) |

---

## Example Caller Workflow

```yaml
name: Code Quality Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  sonarqube:
    uses: EWBHoldings/EWB-devops/.github/workflows/sonarqube.yml@main
    with:
      project-key: 'my-project-key'
      project-name: 'My Project'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

---

## Pipeline Steps

1. Checkout source code (full history — required for accurate blame and new code detection)
2. Run SonarQube scan with project key, name, and server configuration
3. Wait for and check the Quality Gate result — fails the pipeline if the gate is not passed

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `SONAR_TOKEN` unauthorized | Token expired or has insufficient permissions | Generate a new token with `Execute Analysis` permission |
| Quality Gate always fails | Coverage below configured threshold | Add test coverage reports to the scan using `sonar.javascript.lcov.reportPaths` |
| `Project not found` error | Project key mismatch | Verify the key matches exactly what is configured in SonarQube |
| Shallow clone warning | `fetch-depth` too low | The workflow sets `fetch-depth: 0` automatically — ensure no override in the caller |
| Scan times out | Large codebase or slow server | Increase the `timeout-minutes` on the Quality Gate step if needed |
