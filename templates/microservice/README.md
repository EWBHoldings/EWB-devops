# Microservice — Project Template

This template provides the recommended starting structure for a .NET 8 microservice within the EWB ecosystem. It includes a QA pipeline, Docker build pipeline, and security scan — all calling EWB-devops reusable workflows.

---

## Using This Template

1. Copy the contents of this directory into your new repository
2. Replace `EWBHoldings` with your GitHub organisation name throughout
3. Replace `my-microservice` in the workflow file with your image name
4. Rename `MyService` throughout the project to your service name
5. Update the `Dockerfile` `ENTRYPOINT` to match your project's DLL name
6. Run `docker build .` locally to verify the Dockerfile
7. Push to GitHub and confirm all three pipeline jobs run on the first pull request

---

## Recommended Structure

```
my-microservice/
├── src/
│   └── MyService/
│       ├── Controllers/
│       ├── Services/
│       ├── Models/
│       ├── Program.cs
│       └── MyService.csproj
├── tests/
│   └── MyService.Tests/
│       └── MyService.Tests.csproj
├── .github/
│   └── workflows/
│       └── qa.yml              # QA + Docker build + Security scan
├── Dockerfile
├── .dockerignore
├── .gitignore
├── README.md
└── MyService.sln
```

---

## Pipeline Overview

The QA pipeline for this template runs three parallel job groups:

```
pull_request
      │
      ├─── qa           (dotnet-qa.yml — restore, build, test)
      │
      ├─── security     (security-scan.yml — CodeQL for C#)
      │
      └─── docker       (docker-build.yml — build only, no push)
                │
         (on push to main only)
                └─── push to registry
```

---

## Environment Variables

Document all required environment variables in `.env.example`:

```
ASPNETCORE_ENVIRONMENT=Development
ConnectionStrings__DefaultConnection=Server=localhost;Database=MyService;Trusted_Connection=true;
ServiceBus__ConnectionString=your-service-bus-connection-string
ExternalApi__BaseUrl=https://api.example.com
ExternalApi__ApiKey=your-api-key
```

In Kubernetes, inject these as environment variables from Secrets and ConfigMaps. Never commit actual values.

---

## Coding Standards

See [.NET Coding Standards](../../docs/coding-standards/dotnet-coding-standards.md).

---

## Security Standards

See [Secure Coding Guidelines](../../docs/security/secure-coding-guidelines.md) and [Secrets Management](../../docs/security/secrets-management.md).
