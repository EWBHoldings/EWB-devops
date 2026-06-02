# How To Connect a Project to EWB-devops

This guide explains how to connect any existing or new repository to the EWB-devops centralized QA and DevOps platform.

---

## Prerequisites

- [ ] Your repository is in the same GitHub organisation as `EWB-devops`
- [ ] `EWB-devops` is set to **Internal** or **Public** visibility
- [ ] You have write access to your project repository

---

## Step 1 — Choose Your Workflow(s)

Select the reusable workflows that match your project:

| Project Type | Workflow |
|---|---|
| React application | `react-qa.yml` |
| Node.js API or service | `node-qa.yml` |
| .NET 8 application | `dotnet-qa.yml` |
| Java application | `java-qa.yml` |
| Any project with a Dockerfile | `docker-build.yml` |
| Any project needing security scanning | `security-scan.yml` |
| SonarQube analysis | `sonarqube.yml` |
| Tag-based releases | `release.yml` |

---

## Step 2 — Create the Workflow File

In your repository, create `.github/workflows/qa.yml`. Use the template for your stack.

### React

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
    with:
      node-version: '20'
```

### Node.js

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

### .NET

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

### Java

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
```

---

## Step 3 — Add Additional Pipelines (Optional)

Combine multiple workflows in the same file. Use `needs:` to sequence jobs:

```yaml
name: Full Pipeline

on:
  pull_request:
    branches: [main, develop]

jobs:
  qa:
    uses: your-org/EWB-devops/.github/workflows/dotnet-qa.yml@main

  security:
    uses: your-org/EWB-devops/.github/workflows/security-scan.yml@main
    with:
      language: 'csharp'
    permissions:
      security-events: write
      actions: read
      contents: read

  docker:
    needs: qa
    uses: your-org/EWB-devops/.github/workflows/docker-build.yml@main
    with:
      image-name: 'my-api'
      push-image: false
```

---

## Step 4 — Configure Branch Protection

After the pipeline is running, configure branch protection to enforce it on pull requests.

See [branch-protection-setup.md](branch-protection-setup.md) for the detailed setup guide.

---

## Step 5 — Verify

1. Push the `qa.yml` file to your repository
2. Open a pull request (even a trivial change)
3. Navigate to the **Checks** tab on the pull request
4. Confirm the QA pipeline appears and all steps complete successfully

---

## Passing Secrets to Workflows

Secrets are never inherited automatically. Pass them explicitly:

```yaml
jobs:
  sonarqube:
    uses: your-org/EWB-devops/.github/workflows/sonarqube.yml@main
    with:
      project-key: 'my-project'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

Or pass all secrets from the calling workflow using `secrets: inherit` (use with care on public repositories).

---

## Monorepo Support

All workflows accept a `working-directory` input. Use this for monorepos with multiple applications:

```yaml
jobs:
  qa-frontend:
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      working-directory: 'apps/frontend'

  qa-api:
    uses: your-org/EWB-devops/.github/workflows/node-qa.yml@main
    with:
      working-directory: 'apps/api'
```
