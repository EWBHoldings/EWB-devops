# docker-build — Reusable Docker Build Workflow

**File:** `.github/workflows/docker-build.yml`

---

## Purpose

Builds a Docker image from a project's Dockerfile. Optionally pushes the image to any container registry (GitHub Container Registry, Azure Container Registry, Docker Hub, or similar). Uses GitHub Actions layer caching to significantly reduce build time on subsequent runs.

---

## When To Use

Use this workflow for any project that:

- Has a `Dockerfile`
- Needs to produce a container image as part of the CI pipeline
- May need to push images to a registry on merge to `main` or on tag release

---

## Required Project Setup

| Requirement | Details |
|---|---|
| `Dockerfile` present | Must be at the path specified by `dockerfile-path` input (default: `Dockerfile`) |
| `.dockerignore` present | Strongly recommended to exclude `bin/`, `obj/`, `node_modules/`, `.git/` etc. |
| Registry credentials | Required only when `push-image: true` |

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `image-name` | string | — | Image name without registry prefix (e.g. `my-api`) |
| `dockerfile-path` | string | `'Dockerfile'` | Path to the Dockerfile |
| `context` | string | `'.'` | Docker build context |
| `working-directory` | string | `'.'` | Working directory |
| `push-image` | boolean | `false` | Push image to registry after build |
| `registry` | string | `'ghcr.io'` | Container registry URL |
| `image-tag` | string | `'latest'` | Tag to apply to the built image |
| `platforms` | string | `'linux/amd64'` | Target platform(s) |

## Secrets

| Secret | Required | Description |
|---|---|---|
| `REGISTRY_USERNAME` | When pushing | Registry username or service principal |
| `REGISTRY_PASSWORD` | When pushing | Registry password, token, or client secret |

---

## Example Caller Workflow

**Build only (no push) — suitable for pull requests:**

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]

jobs:
  docker:
    uses: your-org/EWB-devops/.github/workflows/docker-build.yml@main
    with:
      image-name: 'my-api'
      push-image: false
```

**Build and push on merge to main:**

```yaml
name: Release Pipeline

on:
  push:
    branches: [main]

jobs:
  docker:
    uses: your-org/EWB-devops/.github/workflows/docker-build.yml@main
    with:
      image-name: 'my-api'
      registry: 'ghcr.io'
      image-tag: ${{ github.sha }}
      push-image: true
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

**Pushing to Azure Container Registry:**

```yaml
jobs:
  docker:
    uses: your-org/EWB-devops/.github/workflows/docker-build.yml@main
    with:
      image-name: 'my-api'
      registry: 'myacr.azurecr.io'
      image-tag: ${{ github.ref_name }}
      push-image: true
    secrets:
      REGISTRY_USERNAME: ${{ secrets.ACR_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.ACR_PASSWORD }}
```

---

## Pipeline Steps

1. Checkout source code
2. Set up Docker Buildx (multi-platform builder)
3. Log in to container registry (only when `push-image: true`)
4. Build image with GitHub Actions layer cache
5. Push image (only when `push-image: true`)
6. Write build summary to the job summary

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `Dockerfile not found` | Path mismatch | Confirm `dockerfile-path` is relative to the build `context`, not the repo root |
| Registry login fails | Incorrect credentials | Verify `REGISTRY_USERNAME` and `REGISTRY_PASSWORD` secrets are set correctly |
| Build slow on first run | Cold cache | Subsequent runs will be faster — the GHA layer cache warms up after the first build |
| Multi-arch build fails | QEMU not available for target | Use `linux/amd64` only, or set up QEMU explicitly in a custom workflow |
| Image too large | Unnecessary files included | Add a comprehensive `.dockerignore` file |
