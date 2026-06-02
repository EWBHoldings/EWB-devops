# Reusable Workflow Architecture

## Overview

EWB-devops implements a **hub-and-spoke** CI/CD architecture using GitHub Actions' `workflow_call` mechanism. EWB-devops is the hub — all project repositories are spokes that call into it.

```
                    ┌─────────────────────────────────────┐
                    │           EWB-devops (hub)           │
                    │                                     │
                    │  .github/workflows/                 │
                    │  ├── react-qa.yml                   │
                    │  ├── node-qa.yml                    │
                    │  ├── dotnet-qa.yml                  │
                    │  ├── java-qa.yml                    │
                    │  ├── security-scan.yml              │
                    │  ├── sonarqube.yml                  │
                    │  ├── docker-build.yml               │
                    │  └── release.yml                    │
                    └────────────────┬────────────────────┘
                                     │  workflow_call
          ┌──────────────────────────┼──────────────────────────┐
          │                          │                          │
  ┌───────▼──────┐       ┌───────────▼──────┐       ┌──────────▼──────┐
  │  project-a   │       │    project-b      │       │   project-c     │
  │  React SPA   │       │    Node.js API    │       │  .NET service   │
  │              │       │                  │       │                 │
  │ qa.yml calls │       │ qa.yml calls     │       │ qa.yml calls    │
  │ react-qa.yml │       │ node-qa.yml      │       │ dotnet-qa.yml   │
  │ security-    │       │ security-        │       │ docker-build.yml│
  │   scan.yml   │       │   scan.yml       │       │ security-       │
  └──────────────┘       └──────────────────┘       │   scan.yml      │
                                                    └─────────────────┘
```

---

## How workflow_call Works

GitHub Actions' `workflow_call` trigger allows a workflow file to be called from another repository's workflow using the `uses:` key:

```yaml
jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/react-qa.yml@main
```

Key properties:
- The **called** workflow runs in the context of the **calling** repository
- Secrets must be explicitly passed from caller to called workflow
- Inputs and outputs are declared in the called workflow's `on.workflow_call` block
- The called workflow cannot access the caller's repository files; it only receives what is passed via inputs and secrets

---

## Versioning Strategy

Each consuming project references EWB-devops workflows at a specific ref:

| Ref | Behaviour | When To Use |
|---|---|---|
| `@main` | Always uses the latest version | Development, fast-moving projects |
| `@v1.2.0` | Pinned to a specific release | Production services requiring stability |
| `@abc1234` | Pinned to a specific commit SHA | Maximum reproducibility |

For most projects, `@main` is recommended. When EWB-devops introduces breaking changes, a migration guide will be provided alongside a new major version tag.

---

## Secrets Propagation

Secrets are never inherited automatically. The calling workflow must explicitly pass secrets:

```yaml
jobs:
  sonarqube:
    uses: EWBHoldings/EWB-devops/.github/workflows/sonarqube.yml@main
    with:
      project-key: 'my-project'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

Alternatively, use `secrets: inherit` to forward all secrets from the calling workflow (use with caution in public repositories):

```yaml
jobs:
  sonarqube:
    uses: EWBHoldings/EWB-devops/.github/workflows/sonarqube.yml@main
    with:
      project-key: 'my-project'
    secrets: inherit
```

---

## Permissions Model

Reusable workflows inherit the permissions of the calling workflow job, not of EWB-devops. If a called workflow requires elevated permissions (e.g. `security-events: write` for CodeQL, `contents: write` for releases), the calling workflow must grant them:

```yaml
jobs:
  security:
    permissions:
      security-events: write
      actions: read
      contents: read
    uses: EWBHoldings/EWB-devops/.github/workflows/security-scan.yml@main
```

---

## Dependency Caching

All workflows implement caching at the tool level:

| Workflow | Cache Type | Cache Key Strategy |
|---|---|---|
| react-qa, node-qa | npm (`~/.npm`) | Hash of `package-lock.json` |
| dotnet-qa | NuGet (managed by SDK action) | Automatic |
| java-qa (Maven) | Maven local repo (`~/.m2`) | Hash of `pom.xml` |
| java-qa (Gradle) | Gradle caches and wrapper | Hash of all `*.gradle*` files |
| docker-build | Docker layer cache (GHA cache) | GitHub Actions cache backend |

---

## Adding a New Reusable Workflow

1. Create `.github/workflows/<name>.yml` in EWB-devops
2. Set `on: workflow_call` with appropriate inputs and secrets
3. Add documentation in `workflows/<name>/README.md`
4. Add an example caller workflow in the appropriate template under `templates/`
5. Update the main `README.md` workflow table
6. Open a pull request — a DevOps team member must review before merge
