# Onboarding — Adopting EWB-devops in Your Project

This guide explains how to connect a new or existing project repository to the EWB-devops centralized QA framework.

---

## Prerequisites

Before starting, confirm the following:

- [ ] Your project is hosted in the same GitHub organisation as `EWB-devops`
- [ ] `EWB-devops` is set to **Internal** or **Public** visibility in the organisation
- [ ] You have write access to your project repository
- [ ] Your project has a `package.json`, `.sln`, or `pom.xml` / `build.gradle` at the expected location

---

## Step 1 — Identify Your Workflow

Choose the reusable workflow that matches your project type:

| Project Type | Workflow File |
|---|---|
| React.js application | `react-qa.yml` |
| Node.js API or service | `node-qa.yml` |
| .NET 8 application | `dotnet-qa.yml` |
| Java application (Maven or Gradle) | `java-qa.yml` |

---

## Step 2 — Create the Workflow File

In your project repository, create the following file:

```
.github/
└── workflows/
    └── qa.yml
```

Use the template for your project type.

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
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
```

With optional inputs:

```yaml
jobs:
  qa:
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      node-version: '20'
      working-directory: '.'
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
    uses: your-org/EWB-devops/.github/workflows/node-qa.yml@main
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
    uses: your-org/EWB-devops/.github/workflows/dotnet-qa.yml@main
    with:
      dotnet-version: '8.0.x'
```

For a specific solution file:

```yaml
jobs:
  qa:
    uses: your-org/EWB-devops/.github/workflows/dotnet-qa.yml@main
    with:
      dotnet-version: '8.0.x'
      solution-file: 'src/MyApp.sln'
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
    uses: your-org/EWB-devops/.github/workflows/java-qa.yml@main
    with:
      java-version: '21'
      java-distribution: 'temurin'
```

The workflow auto-detects Maven (`pom.xml`) or Gradle (`build.gradle` / `build.gradle.kts`).

---

## Step 3 — Configure Branch Protection

Apply these rules to `main` and `develop` in your project repository:

1. Go to **Settings → Branches → Add rule**
2. Apply to branch pattern: `main` and `develop`
3. Enable:
   - **Require status checks to pass before merging**
   - Select the QA job name (e.g. `React QA`, `Node.js QA`, `.NET QA`, `Java QA`)
   - **Require branches to be up to date before merging**
   - **Require at least 1 approving review**
   - **Dismiss stale reviews when new commits are pushed**

---

## Step 4 — Verify the Pipeline

1. Create a feature branch from `develop`
2. Push a small change
3. Open a pull request against `develop`
4. Confirm the QA pipeline appears and runs under the **Checks** tab
5. Verify all steps pass

---

## Monorepo Projects

If your repository contains multiple applications in subdirectories, use the `working-directory` input:

```yaml
jobs:
  qa-frontend:
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      working-directory: 'frontend'

  qa-api:
    uses: your-org/EWB-devops/.github/workflows/node-qa.yml@main
    with:
      working-directory: 'api'
```

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `workflow_call` not found | EWB-devops is Private | Set EWB-devops to Internal or Public |
| Cache miss on every run | `package-lock.json` not committed | Commit `package-lock.json` |
| Lint step skipped | No `lint` script in `package.json` | Add `"lint": "eslint ."` to scripts |
| `.NET` restore fails | SDK version mismatch | Set `dotnet-version` to match `global.json` |
| Java step fails with "no build tool" | Missing `pom.xml` or `build.gradle` | Confirm build file is in the working directory root |

---

## Getting Help

- Open an issue in **EWB-devops** using the Bug Report template
- Tag the DevOps team in your pull request
- Refer to [architecture.md](architecture.md) for design context
