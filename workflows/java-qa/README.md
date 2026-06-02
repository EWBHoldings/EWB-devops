# java-qa — Reusable Java QA Workflow

**File:** `.github/workflows/java-qa.yml`

---

## Purpose

Runs a QA pipeline for Java applications. Automatically detects whether the project uses Maven (`pom.xml`) or Gradle (`build.gradle` / `build.gradle.kts`) and runs the appropriate test command. Uploads test reports as pipeline artifacts.

---

## When To Use

Use this workflow for any Java 21 project including:

- Spring Boot REST APIs
- Spring Batch applications
- Java microservices
- Library projects
- Both Maven and Gradle build systems

---

## Required Project Setup

| Requirement | Details |
|---|---|
| `pom.xml` or `build.gradle` | At least one must be present in the working directory |
| Java 21 compatible | Default — overridable via `java-version` input |
| Temurin distribution | Default — overridable via `java-distribution` input |

For Gradle projects, `gradlew` (the Gradle wrapper) must be committed to the repository.

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `java-version` | string | `'21'` | Java version |
| `java-distribution` | string | `'temurin'` | JDK distribution (`temurin`, `zulu`, `corretto`, `microsoft`) |
| `working-directory` | string | `'.'` | Project root |

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
    uses: your-org/EWB-devops/.github/workflows/java-qa.yml@main
    with:
      java-version: '21'
      java-distribution: 'temurin'
```

---

## Pipeline Steps

1. Checkout source code
2. Setup Java 21 (Temurin)
3. Cache Maven (`~/.m2`) or Gradle (`~/.gradle`) dependencies
4. Detect build tool and run tests:
   - Maven: `mvn clean test --batch-mode --no-transfer-progress`
   - Gradle: `./gradlew test`
5. Upload test reports as artifacts (Surefire reports for Maven, Gradle HTML reports)

---

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `gradlew` not executable | File permission not preserved | Run `git update-index --chmod=+x gradlew` and recommit |
| Neither build tool detected | `pom.xml` and `build.gradle` both absent | Confirm the `working-directory` input points to the project root |
| Maven build downloads everything | Cache key mismatch | Ensure `pom.xml` is committed and not gitignored |
| Spring Boot test fails | Application context requires a database | Use `@DataJpaTest` slices or configure an in-memory H2 database for tests |
| Java version mismatch | Project requires a different JDK | Set `java-version` to match the version in your `pom.xml` or `build.gradle` |
