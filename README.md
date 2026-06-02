# EWB Engineering Standards & DevOps Platform

> The centralized engineering standards, QA pipelines, security guidelines, coding standards, and DevOps tooling for all EWB software projects.

EWB-devops is the single source of truth for how software is built, tested, secured, and shipped across the EWB organization. It provides reusable GitHub Actions workflows, documentation standards, and project starter templates that any team can adopt in minutes.

---

## Platform Objectives

| Objective | Description |
|---|---|
| **Standardization** | One pipeline definition, one set of standards, applied consistently |
| **Reusability** | Update a workflow once — all consumers benefit automatically |
| **Security** | Automated security scanning and dependency review on every PR |
| **Onboarding speed** | New projects adopt full CI/CD in under 15 minutes |
| **Quality enforcement** | Build, lint, test, and security gates on every pull request |

---

## Architecture

```
                 ┌────────────────────────────────────────────────┐
                 │            EWB-devops (this repository)         │
                 │                                                │
                 │  ewb-pipeline.yml  ◄── Orchestrator (use this) │
                 │       │                                        │
                 │       ├── react-qa.yml      (stack: react)     │
                 │       ├── node-qa.yml       (stack: node)      │
                 │       ├── dotnet-qa.yml     (stack: dotnet)    │
                 │       ├── java-qa.yml       (stack: java)      │
                 │       ├── security-scan.yml (feature flag)     │
                 │       ├── sonarqube.yml     (feature flag)     │
                 │       ├── docker-build.yml  (feature flag)     │
                 │       └── release.yml       (feature flag)     │
                 └──────────────────┬─────────────────────────────┘
                                    │  workflow_call
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
┌───────▼──────────┐    ┌───────────▼──────────┐    ┌──────────▼──────────┐
│    project-a     │    │      project-b        │    │      project-c      │
│    React SPA     │    │      Node.js API      │    │    .NET service     │
│                  │    │                       │    │                     │
│ stack: react     │    │ stack: node           │    │ stack: dotnet       │
│ security: true   │    │ security: true        │    │ security: true      │
│ sonarqube: false │    │ sonarqube: true       │    │ docker-build: true  │
└──────────────────┘    └───────────────────────┘    └─────────────────────┘
```

---

## Repository Structure

```
EWB-devops/
├── README.md
│
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows/                  ← Reusable GitHub Actions workflows
│       ├── react-qa.yml
│       ├── node-qa.yml
│       ├── dotnet-qa.yml
│       ├── java-qa.yml
│       ├── security-scan.yml
│       ├── sonarqube.yml
│       ├── docker-build.yml
│       └── release.yml
│
├── workflows/                      ← Documentation for each workflow
│   ├── react-qa/README.md
│   ├── node-qa/README.md
│   ├── dotnet-qa/README.md
│   ├── java-qa/README.md
│   ├── security-scan/README.md
│   ├── sonarqube/README.md
│   ├── docker-build/README.md
│   └── release/README.md
│
├── docs/
│   ├── adr/                        ← Architecture Decision Records
│   │   ├── README.md
│   │   ├── adr-template.md
│   │   └── 0001-use-centralized-github-actions.md
│   ├── architecture/               ← System design documentation
│   │   ├── README.md
│   │   ├── reusable-workflow-architecture.md
│   │   └── repository-standards.md
│   ├── security/                   ← Security standards and guidelines
│   │   ├── README.md
│   │   ├── secure-coding-guidelines.md
│   │   ├── secrets-management.md
│   │   └── dependency-scanning.md
│   ├── coding-standards/           ← Language-specific coding standards
│   │   ├── README.md
│   │   ├── dotnet-coding-standards.md
│   │   ├── react-coding-standards.md
│   │   ├── node-coding-standards.md
│   │   ├── java-coding-standards.md
│   │   └── sql-coding-standards.md
│   └── onboarding/                 ← Guides for new projects and teams
│       ├── README.md
│       ├── how-to-connect-project.md
│       ├── new-repository-checklist.md
│       └── branch-protection-setup.md
│
└── templates/                      ← Starter templates for new projects
    ├── react-app/
    ├── dotnet-api/
    └── microservice/
```

---

## Available Workflows

### Orchestrator (recommended entry point)

| Workflow | File | Purpose |
|---|---|---|
| **EWB Pipeline** | `ewb-pipeline.yml` | Single entry point — select stack + toggle features with boolean flags |

### Individual Workflows (called by the orchestrator)

| Workflow | File | Purpose |
|---|---|---|
| React QA | `react-qa.yml` | Install, lint, test, build for React apps |
| Node.js QA | `node-qa.yml` | Install, lint, test, build for Node.js projects |
| .NET QA | `dotnet-qa.yml` | Restore, build, test for .NET 8 projects |
| Java QA | `java-qa.yml` | Maven or Gradle test for Java 21 projects |
| Security Scan | `security-scan.yml` | CodeQL analysis + dependency review |
| SonarQube | `sonarqube.yml` | SonarQube analysis and Quality Gate check |
| Docker Build | `docker-build.yml` | Build (and optionally push) Docker images |
| Release | `release.yml` | Create GitHub releases from semver tags |

> Each workflow has detailed documentation in the `workflows/` directory.

---

## How To Use

### Recommended: EWB Pipeline Orchestrator

Call one workflow and control everything with feature flags. Create `.github/workflows/qa.yml` in your project:

```yaml
name: Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

permissions:
  security-events: write   # remove if not using security scan
  actions: read
  contents: read

jobs:
  pipeline:
    uses: EWBHoldings/EWB-devops/.github/workflows/ewb-pipeline.yml@main
    with:
      stack: 'dotnet'            # react | node | dotnet | java

      # Toggle features on/off — all default to false
      enable-security-scan: true
      enable-sonarqube:     false
      enable-docker-build:  false
      enable-release:       false
```

See [ewb-pipeline documentation](workflows/ewb-pipeline/README.md) for all inputs, flags, and examples.

---

### Individual Workflows (direct usage)

You can also call individual workflows directly if you need fine-grained control. Each accepts the same `workflow_call` interface.

> **Requirement:** EWB-devops must be set to **Internal** or **Public** visibility in the GitHub organisation.

---

### React Project

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      node-version: '20'
```

---

### Node.js Project

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/node-qa.yml@main
    with:
      node-version: '20'
```

---

### .NET Project

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/dotnet-qa.yml@main
    with:
      dotnet-version: '8.0.x'
```

---

### Java Project

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/java-qa.yml@main
    with:
      java-version: '21'
      java-distribution: 'temurin'
```

---

### Docker Build

```yaml
jobs:
  docker:
    uses: EWBHoldings/EWB-devops/.github/workflows/docker-build.yml@main
    with:
      image-name: 'my-api'
      push-image: false        # true on merge to main, with secrets
```

---

### Security Scan

```yaml
jobs:
  security:
    uses: EWBHoldings/EWB-devops/.github/workflows/security-scan.yml@main
    with:
      language: 'javascript'   # or: csharp, java, python
      fail-on-severity: 'high'
    permissions:
      security-events: write
      actions: read
      contents: read
```

---

### SonarQube

```yaml
jobs:
  sonarqube:
    uses: EWBHoldings/EWB-devops/.github/workflows/sonarqube.yml@main
    with:
      project-key: 'my-project-key'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

---

### Release

```yaml
name: Release

on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  contents: write

jobs:
  release:
    uses: EWBHoldings/EWB-devops/.github/workflows/release.yml@main
    with:
      generate-notes: true
```

---

## Project Templates

Starter templates for common project types are available in `templates/`:

| Template | Location | Pipelines Included |
|---|---|---|
| React application | `templates/react-app/` | QA |
| .NET Web API | `templates/dotnet-api/` | QA |
| .NET Microservice | `templates/microservice/` | QA + Docker Build + Security Scan |

Each template includes a ready-to-use `.github/workflows/qa.yml` — replace `EWBHoldings` with your organisation name and you are ready to run.

---

## Branch Protection Recommendations

Apply the following to `main` and `develop` in all project repositories:

| Rule | Setting |
|---|---|
| Require pull request before merging | Enabled — 1 required approval |
| Dismiss stale reviews on new commits | Enabled |
| Require status checks to pass | Enabled — add the QA pipeline job |
| Require branch to be up to date | Enabled |
| Block force pushes | Enabled |
| Restrict deletions | Enabled |
| Allow bypass | Disabled |

See [branch-protection-setup.md](docs/onboarding/branch-protection-setup.md) for step-by-step instructions.

---

## Pull Request Process

1. Branch from `develop` using `feature/TICKET-description` or `bugfix/TICKET-description`
2. Implement changes following the relevant [coding standards](docs/coding-standards/)
3. Open a pull request against `develop`
4. QA and security pipelines run automatically
5. At least one reviewer approves
6. All checks pass
7. Merge — the branch is deleted automatically

For the complete branching model, see [branch-strategy.md](docs/branch-strategy.md).

---

## Documentation Index

| Section | Description |
|---|---|
| [ADRs](docs/adr/) | Architecture Decision Records |
| [Architecture](docs/architecture/) | System design and repository standards |
| [Security](docs/security/) | Secure coding, secrets management, dependency scanning |
| [Coding Standards](docs/coding-standards/) | .NET, React, Node.js, Java, SQL |
| [Onboarding](docs/onboarding/) | New project setup, checklists, branch protection |
| [Workflow Docs](workflows/) | Per-workflow purpose, inputs, and usage examples |

---

## Future Roadmap

| Planned Workflow | Purpose |
|---|---|
| `k8s-deploy.yml` | Kubernetes deployment pipeline |
| `azure-deploy.yml` | Azure App Service / Container Apps deployment |
| `aws-deploy.yml` | AWS ECS / Lambda deployment |
| `slack-notify.yml` | Slack deployment and failure notifications |
| `teams-notify.yml` | Microsoft Teams notifications |
| `dependency-scan.yml` | OWASP full dependency vulnerability report |
| `performance-test.yml` | k6 / Artillery load testing pipeline |
| `db-migration.yml` | Automated database migration execution |

---

## Contributing to EWB-devops

Changes to this repository affect all consuming projects. Follow this process:

1. Open an issue using the Bug Report or Feature Request template
2. Create a branch: `feature/DEVOPS-description`
3. Implement and validate the change
4. Open a pull request — a DevOps team member must review and approve
5. Merge only after all pipeline checks pass

---

*EWB-devops is maintained by the EWB DevOps team.*
