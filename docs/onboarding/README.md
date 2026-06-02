# Onboarding Documentation

This directory contains guides for setting up new projects within the EWB engineering ecosystem.

---

## Contents

| Document | Description |
|---|---|
| [how-to-connect-project.md](how-to-connect-project.md) | Connect an existing or new project to EWB-devops reusable workflows |
| [new-repository-checklist.md](new-repository-checklist.md) | Checklist of all steps required when creating a new EWB repository |
| [branch-protection-setup.md](branch-protection-setup.md) | Step-by-step guide to configuring GitHub branch protection rules |

---

## Where To Start

If you are setting up a **brand new repository**, start with the [new-repository-checklist.md](new-repository-checklist.md) — it covers everything from creating the repo to connecting the CI pipeline.

If you are **connecting an existing project** to the EWB-devops pipelines, go to [how-to-connect-project.md](how-to-connect-project.md).

If you need to **configure branch protection rules** on an existing repository, see [branch-protection-setup.md](branch-protection-setup.md).

---

## Project Templates

For common project types, starter templates are available in the `templates/` directory at the root of EWB-devops:

| Template | Location |
|---|---|
| React application | `templates/react-app/` |
| .NET API | `templates/dotnet-api/` |
| .NET Microservice | `templates/microservice/` |

Each template includes a pre-configured QA workflow that calls the appropriate EWB-devops workflow.
