# ewb-pipeline — Centralized Pipeline Orchestrator

**File:** `.github/workflows/ewb-pipeline.yml`

---

## Purpose

`ewb-pipeline.yml` is the single entry point for all EWB CI/CD pipelines. Instead of calling multiple individual workflows, consuming projects call this one workflow and use **boolean feature flags** to control exactly which pipeline stages run.

This gives each project team full authority to enable or disable any feature — QA, security scanning, SonarQube, Docker build, and releases — from a single place in their repository.

---

## How It Works

```
Consumer repo: .github/workflows/qa.yml
│
└── uses: EWB-devops/.github/workflows/ewb-pipeline.yml
        │
        ├── stack: 'dotnet'             → runs dotnet-qa.yml
        │
        ├── enable-security-scan: true  → runs security-scan.yml (parallel)
        │
        ├── enable-sonarqube: false     → SKIPPED
        │
        ├── enable-docker-build: true   → runs docker-build.yml (after QA)
        │
        └── enable-release: false       → SKIPPED
```

- **QA always runs** — determined by the `stack` input
- **Security scan runs in parallel** with QA when enabled
- **Docker, SonarQube, and Release run after QA passes** — they are skipped automatically if QA fails

---

## Required Input

| Input | Type | Required | Description |
|---|---|---|---|
| `stack` | string | **Yes** | `react`, `node`, `dotnet`, or `java` |

---

## Feature Flags

All flags default to `false`. Enable only the stages your project needs.

| Flag | Default | What it enables |
|---|---|---|
| `enable-security-scan` | `false` | CodeQL security analysis |
| `enable-sonarqube` | `false` | SonarQube analysis + Quality Gate |
| `enable-docker-build` | `false` | Docker image build |
| `enable-docker-push` | `false` | Push image to registry (requires `enable-docker-build: true`) |
| `enable-release` | `false` | GitHub Release creation from a version tag |

---

## All Inputs

### Runtime Versions

| Input | Default | Description |
|---|---|---|
| `node-version` | `'20'` | Node.js version (react / node stacks) |
| `dotnet-version` | `'8.0.x'` | .NET SDK version (dotnet stack) |
| `java-version` | `'21'` | Java version (java stack) |
| `java-distribution` | `'temurin'` | JDK distribution (java stack) |

### Project Configuration

| Input | Default | Description |
|---|---|---|
| `working-directory` | `'.'` | Project root — use for monorepos |
| `solution-file` | `''` | Path to `.sln` or `.csproj` (dotnet only) |

### SonarQube

| Input | Default | Description |
|---|---|---|
| `sonar-project-key` | `''` | SonarQube project key (required if SonarQube enabled) |
| `sonar-project-name` | `''` | Display name (defaults to project key) |

### Docker

| Input | Default | Description |
|---|---|---|
| `docker-image-name` | `''` | Image name without registry prefix |
| `docker-registry` | `'ghcr.io'` | Container registry URL |
| `docker-image-tag` | `'latest'` | Image tag |
| `dockerfile-path` | `'Dockerfile'` | Path to Dockerfile |

### Release

| Input | Default | Description |
|---|---|---|
| `release-tag` | `''` | Tag name (inferred from git ref if triggered by tag push) |
| `release-draft` | `false` | Create as draft |
| `release-prerelease` | `false` | Mark as pre-release |

---

## Required Secrets

Pass only the secrets required by the features you enable:

| Secret | Required when |
|---|---|
| `SONAR_TOKEN` | `enable-sonarqube: true` |
| `SONAR_HOST_URL` | `enable-sonarqube: true` |
| `REGISTRY_USERNAME` | `enable-docker-push: true` |
| `REGISTRY_PASSWORD` | `enable-docker-push: true` |

---

## Required Permissions

Add these to your calling workflow based on the features you enable:

| Feature | Required permission |
|---|---|
| `enable-security-scan` | `security-events: write`, `actions: read` |
| `enable-release` | `contents: write` |

If you enable neither of these, you can omit the `permissions:` block entirely.

---

## Example Caller Workflows

### Minimal — QA only

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]

jobs:
  pipeline:
    uses: EWBHoldings/EWB-devops/.github/workflows/ewb-pipeline.yml@main
    with:
      stack: 'react'
```

---

### Standard — QA + Security Scan

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

permissions:
  security-events: write
  actions: read
  contents: read

jobs:
  pipeline:
    uses: EWBHoldings/EWB-devops/.github/workflows/ewb-pipeline.yml@main
    with:
      stack: 'dotnet'
      enable-security-scan: true
```

---

### Full — QA + Security + SonarQube + Docker

```yaml
name: Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

permissions:
  security-events: write
  actions: read
  contents: read

jobs:
  pipeline:
    uses: EWBHoldings/EWB-devops/.github/workflows/ewb-pipeline.yml@main
    with:
      stack: 'dotnet'
      enable-security-scan: true
      enable-sonarqube: true
      sonar-project-key: 'my-project-key'
      enable-docker-build: true
      enable-docker-push: false
      docker-image-name: 'my-api'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

---

### Release — push Docker image and create GitHub Release on tag

```yaml
name: Release Pipeline

on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  security-events: write
  actions: read
  contents: write

jobs:
  pipeline:
    uses: EWBHoldings/EWB-devops/.github/workflows/ewb-pipeline.yml@main
    with:
      stack: 'dotnet'
      enable-security-scan: true
      enable-docker-build: true
      enable-docker-push: true
      docker-image-name: 'my-api'
      docker-image-tag: ${{ github.ref_name }}
      enable-release: true
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

---

## CodeQL Language Mapping

The `stack` input is automatically mapped to the correct CodeQL language — no manual configuration needed:

| Stack | CodeQL Language |
|---|---|
| `react` | `javascript` |
| `node` | `javascript` |
| `dotnet` | `csharp` |
| `java` | `java` |

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| All jobs skipped except QA | Feature flags default to `false` | Set the desired flags to `true` |
| Security scan fails with permission error | `security-events: write` not granted | Add `permissions: security-events: write` to the caller workflow |
| Docker push fails | Registry credentials missing | Add `REGISTRY_USERNAME` and `REGISTRY_PASSWORD` secrets |
| SonarQube fails with `project not found` | Missing `sonar-project-key` | Set `sonar-project-key` to match the key in your SonarQube server |
| Release fails | No tag available | Trigger via a tag push (`git tag v1.0.0 && git push origin v1.0.0`) |
