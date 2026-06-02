# dotnet-qa — Reusable .NET QA Workflow

**File:** `.github/workflows/dotnet-qa.yml`

---

## Purpose

Runs a full QA pipeline for .NET 8 applications. Covers NuGet package restore, Release-mode build, test execution with result upload, and supports Clean Architecture, minimal API, and microservice project layouts.

---

## When To Use

Use this workflow for any .NET 8 project including:

- ASP.NET Core Web APIs
- Minimal APIs
- Background services / Worker services
- Class libraries with unit tests
- Clean Architecture solutions

---

## Required Project Setup

| Requirement | Details |
|---|---|
| `.NET 8 SDK` | Default — overridable via `dotnet-version` input |
| At least one `.csproj` or `.sln` | Must be discoverable from the working directory |
| Test project present | `dotnet test` is called unconditionally — it will pass with no test projects found |

Recommended solution structure:

```
MyApp/
├── src/
│   ├── MyApp.API/
│   └── MyApp.Application/
├── tests/
│   ├── MyApp.UnitTests/
│   └── MyApp.IntegrationTests/
└── MyApp.sln
```

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `dotnet-version` | string | `'8.0.x'` | .NET SDK version |
| `working-directory` | string | `'.'` | Project root |
| `solution-file` | string | `''` | Path to a specific `.sln` or `.csproj` (auto-detected if omitted) |

---

## Example Caller Workflow

```yaml
name: QA Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/dotnet-qa.yml@main
    with:
      dotnet-version: '8.0.x'
```

Targeting a specific solution file:

```yaml
jobs:
  qa:
    uses: EWBHoldings/EWB-devops/.github/workflows/dotnet-qa.yml@main
    with:
      dotnet-version: '8.0.x'
      solution-file: 'src/MyApp.sln'
```

---

## Pipeline Steps

1. Checkout source code
2. Setup .NET 8 SDK
3. Restore NuGet dependencies (`dotnet restore`)
4. Build solution in Release configuration (`dotnet build --no-restore`)
5. Run tests with TRX logger (`dotnet test --no-build`)
6. Upload `.trx` test results as a pipeline artifact (retained 7 days)

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| Restore fails — package not found | Private NuGet feed requires authentication | Configure `NUGET_AUTH_TOKEN` secret and add a `nuget.config` to the repo |
| Multiple solution files found | Auto-detection is ambiguous | Pass the `solution-file` input explicitly |
| Test results not appearing | Tests project not referencing correct test SDK | Ensure test projects reference `Microsoft.NET.Test.Sdk` |
| Build passes but tests fail | Missing appsettings or connection strings | Use `appsettings.Testing.json` and environment variable overrides in CI |
