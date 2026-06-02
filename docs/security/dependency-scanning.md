# Dependency Scanning

This document describes how EWB projects identify, track, and remediate vulnerable third-party dependencies.

---

## Why Dependency Scanning Matters

Third-party dependencies are one of the most common attack vectors in modern software. A single vulnerable package in your dependency tree can expose the application to known exploits — regardless of how carefully you write your own code.

EWB projects are expected to maintain dependencies free of known high and critical severity vulnerabilities.

---

## Scanning in CI

The `security-scan.yml` reusable workflow includes an automated dependency review step that runs on pull requests. It blocks merges that introduce dependencies with vulnerabilities at or above the configured severity threshold.

See [security-scan workflow documentation](../../workflows/security-scan/README.md) for setup details.

---

## Automated Scanning Tools by Stack

### JavaScript / TypeScript (npm)

**GitHub Dependabot** — enable in repository settings:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: '/'
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
```

**Manual audit:**

```bash
npm audit
npm audit --audit-level=high   # Show only high and critical
npm audit fix                  # Auto-fix compatible vulnerabilities
```

---

### .NET (NuGet)

**GitHub Dependabot:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: nuget
    directory: '/'
    schedule:
      interval: weekly
```

**Manual audit:**

```bash
dotnet list package --vulnerable
dotnet list package --vulnerable --include-transitive
```

---

### Java (Maven)

**GitHub Dependabot:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: maven
    directory: '/'
    schedule:
      interval: weekly
```

**Manual scan with OWASP Dependency Check:**

```bash
mvn org.owasp:dependency-check-maven:check
```

---

### Java (Gradle)

**Dependency Check Gradle plugin:**

```groovy
// build.gradle
plugins {
    id 'org.owasp.dependencycheck' version '9.0.0'
}
```

```bash
./gradlew dependencyCheckAnalyze
```

---

## Recommended .github/dependabot.yml

Enable Dependabot for all EWB repositories. A single file can cover multiple ecosystems:

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: '/'
    schedule:
      interval: weekly
    open-pull-requests-limit: 5

  - package-ecosystem: github-actions
    directory: '/'
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
```

Always include the `github-actions` ecosystem to keep action versions current.

---

## Severity Classifications

| Severity | CVSS Score | Required Action |
|---|---|---|
| Critical | 9.0 – 10.0 | Remediate within 24 hours |
| High | 7.0 – 8.9 | Remediate within 1 week |
| Moderate | 4.0 – 6.9 | Remediate within 1 sprint |
| Low | 0.1 – 3.9 | Remediate in scheduled dependency updates |

---

## Remediation Process

1. Receive alert (Dependabot PR, CI failure, or manual audit)
2. Assess the severity and whether the vulnerable code path is actually reachable
3. Update the dependency to the patched version
4. Run the test suite to confirm no regressions
5. Merge the fix through the normal PR process
6. For critical vulnerabilities, consider deploying a hotfix immediately

If a patch is not yet available, assess whether a workaround is possible (e.g. disabling the vulnerable feature, applying a WAF rule). Document the decision in a GitHub issue.

---

## False Positives

If a vulnerability scanner flags a dependency that is not actually exploitable in your context (e.g. a server-side vulnerability in a client-only package), document the reason in the relevant issue or Dependabot alert and dismiss with a written rationale.
