# react-qa — Reusable React QA Workflow

**File:** `.github/workflows/react-qa.yml`

---

## Purpose

Runs a complete quality assurance pipeline for React.js applications. Covers dependency installation, linting, unit testing, and production build verification. Steps that are not defined in the project's `package.json` are skipped gracefully rather than failing.

---

## When To Use

Use this workflow for any React or Vite-based frontend application that:

- Uses Node.js and npm for package management
- Has ESLint configured for static analysis
- Has a test suite (Jest, Vitest, React Testing Library, or similar)
- Produces a production build artifact

---

## Required Project Setup

| Requirement | Details |
|---|---|
| `package.json` present | Must be in the working directory |
| `package-lock.json` committed | Enables `npm ci` and dependency caching |
| Node.js 20 compatible | Default runtime — overridable via `node-version` input |

Optional scripts in `package.json` (each step is skipped if the script is absent):

```json
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx",
    "test": "react-scripts test",
    "build": "react-scripts build"
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
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      node-version: '20'
      working-directory: '.'
```

For a monorepo where the React app lives in a subdirectory:

```yaml
jobs:
  qa:
    uses: your-org/EWB-devops/.github/workflows/react-qa.yml@main
    with:
      working-directory: 'frontend'
```

---

## Pipeline Steps

1. Checkout source code
2. Setup Node.js with npm cache
3. Install dependencies (`npm ci` if lockfile present, else `npm install`)
4. Run ESLint (`npm run lint`) — skipped if no `lint` script
5. Run unit tests (`npm test`) — skipped if no `test` script
6. Build application (`npm run build`) — skipped if no `build` script

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| Cache miss on every run | `package-lock.json` not committed | Run `npm install` locally and commit the lockfile |
| Test step times out | Jest waiting for input | Set `CI=true` in your test script or confirm it is already set in the workflow |
| Build fails but passes locally | Environment variable missing | Add required `REACT_APP_*` variables to GitHub repository secrets and pass via `env:` in your caller workflow |
| Lint not running | No `lint` script in `package.json` | Add `"lint": "eslint src"` to scripts |
