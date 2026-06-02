# New Repository Checklist

Use this checklist when creating a new EWB project repository. Complete all items before the project's first sprint.

---

## Repository Creation

- [ ] Repository named following EWB naming convention (see [repository-standards.md](../architecture/repository-standards.md))
- [ ] Repository created in the EWB GitHub organisation (not a personal account)
- [ ] Visibility set correctly: **Private** for internal projects, **Internal** for shared libraries
- [ ] Repository description filled in (one sentence describing the project)
- [ ] Repository topics added (e.g. `dotnet`, `react`, `microservice`)

---

## Initial Files

- [ ] `README.md` created with project overview, setup instructions, and environment variable documentation
- [ ] `.gitignore` added and appropriate for the project type
- [ ] `.editorconfig` added for consistent editor settings
- [ ] `LICENSE` added if applicable

---

## CI/CD Pipeline

- [ ] `.github/workflows/qa.yml` created calling the appropriate EWB-devops workflow
- [ ] Pipeline tested — at least one PR has been raised and the pipeline ran successfully
- [ ] Additional pipelines added where required (security scan, Docker build, SonarQube)

---

## Branch Configuration

- [ ] `main` branch exists and is the default branch
- [ ] `develop` branch created from `main`
- [ ] Branch protection rules applied to `main` (see [branch-protection-setup.md](branch-protection-setup.md))
- [ ] Branch protection rules applied to `develop`
- [ ] Direct commits to `main` and `develop` restricted

---

## Secrets and Configuration

- [ ] Required secrets added to GitHub repository secrets or organisation secrets
- [ ] `.env.example` file committed with all required variable names (no values)
- [ ] `.env` listed in `.gitignore`
- [ ] No credentials or secrets committed to the repository

---

## Dependabot

- [ ] `.github/dependabot.yml` created and configured for the project's package ecosystem
- [ ] `github-actions` ecosystem included in dependabot configuration

---

## Repository Settings

- [ ] Issues enabled
- [ ] Pull requests configured (merge commit, squash merge, or both — disable rebase merge unless intentional)
- [ ] Branch auto-delete after merge enabled (`Settings → General → Automatically delete head branches`)
- [ ] Dependency graph enabled (`Settings → Code security → Dependency graph`)
- [ ] Dependabot security updates enabled (`Settings → Code security → Dependabot security updates`)

---

## Team Access

- [ ] Relevant team(s) granted appropriate access (typically `Write` for the development team, `Maintain` for the lead)
- [ ] No individual user access outside team assignments where avoidable

---

## Documentation

- [ ] `README.md` complete and accurate
- [ ] Environment variables documented
- [ ] Local setup instructions tested and working on a clean machine (or documented as tested)
- [ ] Architecture Decision Records created for any significant design decisions (see [docs/adr/](../adr/))

---

## Sign-Off

Before the repository is used for production code, confirm the following with the team lead or tech lead:

- [ ] All checklist items above completed
- [ ] Code review process agreed within the team
- [ ] On-call or ownership defined
