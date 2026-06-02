# Architecture — Centralized QA & DevOps Framework

## Overview

EWB-devops implements a **centralized, reusable workflow architecture** for GitHub Actions. Rather than each project repository owning and maintaining its own CI/CD pipelines, all workflow definitions live in a single authoritative repository and are consumed by downstream projects via GitHub's `workflow_call` mechanism.

---

## Design Principles

| Principle | Description |
|---|---|
| **Single source of truth** | Workflow logic lives in one place; all consumers reference it |
| **Loose coupling** | Consuming projects declare which workflow to use, not how it works |
| **Parameterisation** | Workflows accept inputs so consumers can customise runtime, directory, and options |
| **Fail-fast** | All pipeline steps fail the run immediately on error |
| **Graceful degradation** | Optional scripts (lint, test, build) are skipped if not present rather than failing |

---

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        EWB-devops                               │
│                                                                 │
│   .github/workflows/                                            │
│   ├── react-qa.yml      ◄─── Reusable workflow (workflow_call)  │
│   ├── node-qa.yml       ◄─── Reusable workflow (workflow_call)  │
│   ├── dotnet-qa.yml     ◄─── Reusable workflow (workflow_call)  │
│   └── java-qa.yml       ◄─── Reusable workflow (workflow_call)  │
│                                                                 │
└──────────────────────────────┬──────────────────────────────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
    ┌───────▼──────┐  ┌────────▼──────┐  ┌───────▼───────┐
    │  project-a   │  │  project-b    │  │   project-c   │
    │  React app   │  │  Node.js API  │  │  .NET service │
    │              │  │               │  │               │
    │ qa.yml:      │  │ qa.yml:       │  │ qa.yml:       │
    │ uses:        │  │ uses:         │  │ uses:         │
    │ react-qa.yml │  │ node-qa.yml   │  │ dotnet-qa.yml │
    └──────────────┘  └───────────────┘  └───────────────┘
```

---

## Workflow Consumption Model

Consuming repositories use GitHub's `workflow_call` trigger. The workflow reference format is:

```
{org}/{repo}/.github/workflows/{file}.yml@{ref}
```

Example:

```yaml
jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/react-qa.yml@main
```

The `@main` ref means consumers always run the latest approved version of the workflow. To pin to a specific version, use a commit SHA or tag:

```yaml
uses: EWBHoldings/EWB-devops/.github/workflows/react-qa.yml@v1.2.0
```

---

## Workflow Inputs

Each workflow exposes typed inputs to allow project-level customisation without forking the workflow:

| Input | Type | Default | Purpose |
|---|---|---|---|
| `node-version` | string | `'20'` | Override the Node.js version |
| `dotnet-version` | string | `'8.0.x'` | Override the .NET SDK version |
| `java-version` | string | `'21'` | Override the Java version |
| `java-distribution` | string | `'temurin'` | Choose JDK distribution |
| `working-directory` | string | `'.'` | Support monorepo project roots |
| `solution-file` | string | `''` | Pin .NET build to a specific solution |

---

## Dependency Caching

All workflows implement dependency caching to reduce pipeline execution time:

| Workflow | Cache Location | Cache Key |
|---|---|---|
| React / Node.js | `~/.npm` | Hash of `package-lock.json` |
| .NET | NuGet packages (managed by SDK) | Automatic |
| Java (Maven) | `~/.m2/repository` | Hash of `pom.xml` |
| Java (Gradle) | `~/.gradle/caches`, `~/.gradle/wrapper` | Hash of `*.gradle*` files |

---

## Repository Access Model

For `workflow_call` to work across repositories, EWB-devops must be set to **Internal** (for GitHub Enterprise) or **Public** visibility. All repositories within the organisation can then reference its workflows.

---

## Future Extensions

The architecture is designed for incremental growth. Adding a new reusable workflow requires only:

1. Creating a new `.yml` file in `.github/workflows/`
2. Defining `on: workflow_call` with appropriate inputs
3. Documenting the workflow in `README.md` and `onboarding.md`

No changes to consuming repositories are required until they opt in to the new workflow.
