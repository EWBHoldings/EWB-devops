# release — Reusable GitHub Release Workflow

**File:** `.github/workflows/release.yml`

---

## Purpose

Creates a GitHub Release from a semantic version tag. Automatically generates release notes from merged pull requests and commits since the previous release. Supports draft and pre-release flags for staged rollout workflows.

---

## When To Use

Use this workflow when:

- A version tag is pushed to `main` (e.g. `v1.2.0`)
- You want to automate release creation as part of a CD pipeline
- You need consistent, auto-generated release notes

---

## Required Project Setup

| Requirement | Details |
|---|---|
| Repository uses semver tags | Tags must follow `vMAJOR.MINOR.PATCH` format (e.g. `v1.0.0`) |
| `contents: write` permission | The calling workflow must grant this permission |

Recommended tagging convention:

```bash
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `tag-name` | string | `''` | Release tag — inferred from git ref if empty and triggered by tag push |
| `generate-notes` | boolean | `true` | Auto-generate release notes from commits and PRs |
| `draft` | boolean | `false` | Create as a draft release |
| `prerelease` | boolean | `false` | Mark as a pre-release |

---

## Example Caller Workflow

**Trigger on tag push (recommended):**

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
    uses: your-org/EWB-devops/.github/workflows/release.yml@main
    with:
      generate-notes: true
```

**Create a pre-release for a release candidate:**

```yaml
jobs:
  release:
    uses: your-org/EWB-devops/.github/workflows/release.yml@main
    with:
      tag-name: 'v2.0.0-rc.1'
      prerelease: true
      generate-notes: true
```

**Create a draft release for review before publishing:**

```yaml
jobs:
  release:
    uses: your-org/EWB-devops/.github/workflows/release.yml@main
    with:
      draft: true
      generate-notes: true
```

---

## Pipeline Steps

1. Checkout source code (full history for accurate release notes)
2. Resolve tag name from input or git ref
3. Validate semantic version format (warning only — does not block)
4. Create GitHub Release using `softprops/action-gh-release`
5. Write release summary to the job summary

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `No tag name provided` error | Workflow triggered by a push, not a tag | Push a version tag: `git tag v1.0.0 && git push origin v1.0.0` |
| Release already exists | Tag was already released | Delete the existing release and re-run, or use a new tag |
| No release notes generated | No PRs or conventional commits since the last tag | Merge PRs with descriptive titles; notes are generated from PR titles |
| Permission denied creating release | `contents: write` not granted | Add `permissions: contents: write` to your caller workflow |
