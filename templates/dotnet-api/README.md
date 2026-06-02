# .NET API — Project Template

This template provides the recommended starting structure for a .NET 8 Web API within the EWB ecosystem. It follows Clean Architecture principles and includes a pre-configured QA pipeline.

---

## Using This Template

1. Copy the contents of this directory into your new repository
2. Replace `your-org` in `.github/workflows/qa.yml` with your GitHub organisation name
3. Rename `MyApp` throughout the project to your application name
4. Update `README.md` with project-specific information
5. Run `dotnet restore` and `dotnet build` locally to verify the setup
6. Push to GitHub and confirm the QA pipeline executes on the first pull request

---

## Recommended Project Structure

```
MyApp/
├── src/
│   ├── MyApp.API/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Program.cs
│   │   └── MyApp.API.csproj
│   ├── MyApp.Application/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── Interfaces/
│   │   └── MyApp.Application.csproj
│   ├── MyApp.Domain/
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Exceptions/
│   │   └── MyApp.Domain.csproj
│   └── MyApp.Infrastructure/
│       ├── Persistence/
│       ├── Repositories/
│       └── MyApp.Infrastructure.csproj
├── tests/
│   ├── MyApp.UnitTests/
│   │   └── MyApp.UnitTests.csproj
│   └── MyApp.IntegrationTests/
│       └── MyApp.IntegrationTests.csproj
├── .github/
│   └── workflows/
│       └── qa.yml
├── .gitignore
├── README.md
└── MyApp.sln
```

---

## appsettings Guidance

Keep environment-specific values out of `appsettings.json`. Use the following pattern:

**appsettings.json** — committed, contains non-sensitive defaults:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

**appsettings.Development.json** — gitignored, contains local developer overrides:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;Trusted_Connection=true;"
  }
}
```

**Environment variables in production** — override via `ConnectionStrings__DefaultConnection`, `JwtSettings__Secret`, etc. (double underscore maps to nested JSON keys).

Add `appsettings.Development.json` and `appsettings.Local.json` to `.gitignore`.

---

## Coding Standards

See [.NET Coding Standards](../../docs/coding-standards/dotnet-coding-standards.md).

---

## QA Pipeline

The `.github/workflows/qa.yml` in this template calls `dotnet-qa.yml` from EWB-devops, which runs:

1. `dotnet restore`
2. `dotnet build --configuration Release`
3. `dotnet test --configuration Release`
4. Uploads `.trx` test results as pipeline artifacts
