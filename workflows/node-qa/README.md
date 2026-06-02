# node-qa — Reusable Node.js QA Workflow

**File:** `.github/workflows/node-qa.yml`

---

## Purpose

Runs a quality assurance pipeline for Node.js applications and APIs. Covers dependency installation, linting, unit testing, and optional build validation. Designed for Express APIs, serverless functions, CLI tools, and general-purpose Node.js services.

---

## When To Use

Use this workflow for any Node.js project that:

- Uses npm for package management
- Has a linter (ESLint or similar) configured
- Has a test suite (Jest, Mocha, Vitest, or similar)
- Is not a React frontend (use `react-qa.yml` instead for frontend apps)

---

## Required Project Setup

| Requirement | Details |
|---|---|
| `package.json` present | Must be in the working directory |
| `package-lock.json` committed | Enables `npm ci` and dependency caching |
| Node.js 20 compatible | Default runtime — overridable via `node-version` input |

Recommended scripts in `package.json`:

```json
{
  "scripts": {
    "lint": "eslint src --ext .js,.ts",
    "test": "jest",
    "build": "tsc"
  }
}
```

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `node-version` | string | `'20'` | Node.js version to use |
| `working-directory` | string | `'.'` | Project root (useful in monorepos) |

---

## Example Caller Workflow

Create `.github/workflows/qa.yml` in your project repository:

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

## Pipeline Steps

1. Checkout source code
2. Setup Node.js with npm cache
3. Install dependencies (`npm ci` if lockfile present, else `npm install`)
4. Run lint (`npm run lint`) — skipped if no `lint` script
5. Run unit tests (`npm test`) — skipped if no `test` script
6. Validate build (`npm run build`) — skipped if no `build` script

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `package-lock.json` not found | Lockfile not committed | Commit `package-lock.json` to the repository |
| Tests fail in CI but pass locally | Missing environment variables | Provide required env vars via caller workflow `env:` block or repository secrets |
| TypeScript build fails | `typescript` not in dependencies | Add `typescript` as a dev dependency |
| npm cache not hitting | `working-directory` differs from lockfile location | Set `cache-dependency-path` or ensure lockfile is at the project root |
