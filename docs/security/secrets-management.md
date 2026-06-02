# Secrets Management

This document defines how credentials, API keys, connection strings, and other sensitive values must be managed across all EWB software projects.

---

## The Rule

**No secret, credential, API key, connection string, password, or token may ever be committed to source control** — not in code files, configuration files, `.env` files, comments, or documentation.

This applies to all branches, including feature branches and personal forks.

---

## What Counts as a Secret

| Category | Examples |
|---|---|
| Database credentials | Connection strings, passwords, usernames |
| API keys | Third-party service tokens, webhook secrets |
| Authentication | JWT signing keys, OAuth client secrets, certificate private keys |
| Cloud credentials | AWS access keys, Azure service principal credentials, GCP service account keys |
| Infrastructure | SSH private keys, VPN credentials, internal service passwords |
| Encryption keys | Application encryption keys, KMS key IDs |

---

## Where Secrets Live

### GitHub Actions Secrets

For CI/CD pipelines, secrets are stored in GitHub and injected as environment variables at runtime.

**Repository secrets** — available to workflows in a single repository:
- `Settings → Secrets and variables → Actions → New repository secret`

**Organisation secrets** — available to all (or selected) repositories in the organisation:
- Organisation `Settings → Secrets and variables → Actions → New organisation secret`

Reference in workflow:

```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.THIRD_PARTY_API_KEY }}
```

### Application Runtime Secrets

For deployed applications, secrets are injected via environment variables from the deployment platform:

| Platform | Mechanism |
|---|---|
| Azure App Service | Application Settings |
| Azure Container Apps | Secrets + environment variable references |
| Kubernetes | Kubernetes Secrets (mounted as env vars) |
| AWS ECS | AWS Secrets Manager or Parameter Store |
| Docker Compose (local dev) | `.env` file — never committed, documented via `.env.example` |

### Local Development

Provide a `.env.example` file that documents required variables with placeholder values:

```
# .env.example — copy to .env and fill in values for local development
DATABASE_URL=postgresql://localhost:5432/myapp
EXTERNAL_API_KEY=your-api-key-here
JWT_SECRET=replace-with-a-secure-random-string
```

The actual `.env` file must be listed in `.gitignore`.

---

## .gitignore Requirements

All project repositories must gitignore the following:

```gitignore
# Environment and secrets
.env
.env.local
.env.*.local
*.pem
*.key
*.p12
*.pfx
secrets.json
appsettings.Development.json    # May contain local dev secrets
appsettings.Local.json
```

---

## What To Do If a Secret Is Committed

If a secret is accidentally committed:

1. **Rotate the secret immediately.** Assume it is compromised from the moment it was committed, even if the commit was made to a private repository.
2. Remove the secret from the commit history using `git filter-repo` or by contacting the DevOps team.
3. Report the incident internally — the team must assess whether the secret was accessed.
4. Add the file or pattern to `.gitignore` to prevent recurrence.

Removing the secret from git history does not guarantee safety — GitHub caches commit content and the secret may have been cloned or observed before removal.

---

## Secret Rotation

Secrets must be rotated:

- Immediately if suspected of compromise
- When a team member with access leaves the organisation
- Periodically per the organisation's security policy

Document rotation procedures in the project's runbook or operational documentation.

---

## Pre-commit Prevention (Optional but Recommended)

Use [gitleaks](https://github.com/gitleaks/gitleaks) or [truffleHog](https://github.com/trufflesecurity/trufflehog) as a pre-commit hook to detect secrets before they are committed:

```bash
# Install gitleaks (macOS)
brew install gitleaks

# Run a scan on the current repository
gitleaks detect --source . --verbose
```

This can also be added as a step in the security-scan workflow.
