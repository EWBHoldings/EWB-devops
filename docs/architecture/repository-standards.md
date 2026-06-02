# Repository Standards

These standards apply to all EWB project repositories. They define minimum expectations for structure, configuration, and tooling.

---

## Repository Naming

| Project Type | Convention | Example |
|---|---|---|
| Frontend application | `{name}-web` or `{name}-ui` | `customer-portal-web` |
| Backend API | `{name}-api` | `payments-api` |
| Background service | `{name}-service` or `{name}-worker` | `email-worker` |
| Shared library | `{name}-lib` or `{name}-sdk` | `EWB-common-lib` |
| Infrastructure | `{name}-infra` | `payments-infra` |

Use lowercase kebab-case for all repository names.

---

## Required Files

Every EWB repository must contain the following:

| File | Purpose |
|---|---|
| `README.md` | Project overview, setup instructions, and usage |
| `.gitignore` | Appropriate gitignore for the project type |
| `.github/workflows/qa.yml` | QA pipeline calling the relevant EWB-devops workflow |
| `LICENSE` (if applicable) | Licence file for open-source or shared projects |

---

## Recommended Structure

### .NET API

```
my-api/
├── src/
│   ├── MyApi.API/              # Entry point, controllers, middleware
│   ├── MyApi.Application/      # Business logic, commands, queries
│   ├── MyApi.Domain/           # Domain models, interfaces
│   └── MyApi.Infrastructure/   # Data access, external integrations
├── tests/
│   ├── MyApi.UnitTests/
│   └── MyApi.IntegrationTests/
├── .github/
│   └── workflows/
│       └── qa.yml
├── .gitignore
├── README.md
└── MyApi.sln
```

### React Application

```
my-app/
├── public/
├── src/
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   ├── services/
│   ├── store/
│   └── types/
├── .github/
│   └── workflows/
│       └── qa.yml
├── .eslintrc.js
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

### Node.js API

```
my-service/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   ├── middleware/
│   └── routes/
├── tests/
├── .github/
│   └── workflows/
│       └── qa.yml
├── .eslintrc.js
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

---

## Branch Configuration

All repositories must have branch protection rules applied to `main` and `develop`. See [branch-protection-setup.md](../onboarding/branch-protection-setup.md) for the step-by-step configuration guide.

Minimum required rules:

- Require pull request before merging (no direct commits)
- Require at least 1 approving review
- Require status checks to pass (the QA pipeline job)
- Require branches to be up to date before merging
- Restrict force pushes

---

## Secrets Management

- No secrets, credentials, API keys, or connection strings in source code or committed files
- All secrets managed via GitHub repository secrets or organisation secrets
- Environment-specific configuration via environment variables
- See [secrets-management.md](../security/secrets-management.md) for full guidance

---

## Dependency Management

- All dependencies pinned to specific versions in the lockfile (`package-lock.json`, `packages.lock.json`, `pom.xml`, `build.gradle`)
- Lockfiles must be committed to the repository
- Dependencies reviewed regularly for vulnerabilities
- Security scan workflow configured to block PRs introducing high-severity vulnerabilities

---

## README Requirements

Every project README must include:

1. **Project name and one-line description**
2. **Prerequisites** — runtime versions, tools required
3. **Local setup** — step-by-step instructions to run locally
4. **Running tests** — how to execute the test suite
5. **Environment variables** — documented (not their values)
6. **CI/CD** — brief note on the pipeline and where to view results
7. **Contact / ownership** — team or individuals responsible
