# Branch Strategy

This document defines the Git branching conventions used across all EWB software projects.

---

## Branch Model

EWB projects follow a **trunk-based development with short-lived feature branches** model, derived from GitFlow principles but simplified for modern CI/CD workflows.

```
main
 │
 ├── hotfix/TICKET-critical-fix          (emergency production fix)
 │
develop
 │
 ├── feature/TICKET-user-authentication  (new feature)
 ├── feature/TICKET-dashboard-redesign   (new feature)
 ├── bugfix/TICKET-login-redirect-error  (non-critical bug)
 └── bugfix/TICKET-pagination-off-by-one (non-critical bug)
```

---

## Branch Definitions

### `main`

| Property | Value |
|---|---|
| **Purpose** | Production-ready code |
| **Deployable** | Always |
| **Direct commits** | Never |
| **Merges from** | `develop` (releases), `hotfix/*` (emergencies) |
| **Protection** | Required reviews + passing pipeline |

`main` always reflects what is deployed or ready to be deployed to production. Every commit on `main` should be deployable without additional work.

---

### `develop`

| Property | Value |
|---|---|
| **Purpose** | Integration branch for active development |
| **Deployable** | Should be — staging environment target |
| **Direct commits** | Never |
| **Merges from** | `feature/*`, `bugfix/*` |
| **Protection** | Required reviews + passing pipeline |

All completed features are merged into `develop`. This is the base branch for all feature and bugfix branches.

---

### `feature/*`

| Property | Value |
|---|---|
| **Purpose** | New functionality |
| **Base branch** | `develop` |
| **Merge target** | `develop` |
| **Lifetime** | Short-lived — deleted after merge |

**Naming convention:**

```
feature/TICKET-short-description
```

Examples:

```
feature/EWB-42-user-authentication
feature/EWB-87-export-to-csv
feature/EWB-101-multi-tenant-support
```

---

### `bugfix/*`

| Property | Value |
|---|---|
| **Purpose** | Fix a defect in `develop` or pre-release |
| **Base branch** | `develop` |
| **Merge target** | `develop` |
| **Lifetime** | Short-lived — deleted after merge |

**Naming convention:**

```
bugfix/TICKET-short-description
```

Examples:

```
bugfix/EWB-55-login-redirect-loop
bugfix/EWB-63-null-reference-on-export
```

---

### `hotfix/*`

| Property | Value |
|---|---|
| **Purpose** | Emergency fix for a production defect |
| **Base branch** | `main` |
| **Merge target** | `main` AND `develop` |
| **Lifetime** | Short-lived — deleted after both merges |

**Naming convention:**

```
hotfix/TICKET-short-description
```

Examples:

```
hotfix/EWB-99-payment-calculation-error
hotfix/EWB-112-data-loss-on-save
```

Hotfixes must be merged into both `main` and `develop` to keep branches in sync.

---

## Pull Request Requirements

All merges must go through a pull request. Direct commits to `main` or `develop` are not permitted.

| Requirement | main | develop |
|---|---|---|
| Minimum approvals | 1 | 1 |
| Passing QA pipeline | Required | Required |
| Up to date with target | Required | Required |
| Squash or merge commit | Recommended | Recommended |

---

## Commit Message Convention

Use the following format for all commits:

```
type(scope): short description

Optional longer explanation if needed.
```

Types:

| Type | When to use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Tooling, build, dependencies |
| `docs` | Documentation only |
| `refactor` | Code change without feature or fix |
| `test` | Adding or updating tests |
| `ci` | CI/CD pipeline changes |

Examples:

```
feat(auth): add JWT refresh token support
fix(dashboard): correct pagination offset calculation
chore(deps): upgrade Node.js to 20.x
ci(pipelines): add .NET QA reusable workflow
```

---

## Release Process

1. All features for a release are merged into `develop`
2. QA validation is performed on `develop`
3. A pull request is raised from `develop` to `main`
4. The PR is reviewed and approved
5. The QA pipeline passes on the merge commit
6. Merge into `main`
7. Tag the release: `v{major}.{minor}.{patch}`

```
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```
