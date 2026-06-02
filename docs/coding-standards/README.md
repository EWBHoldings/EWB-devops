# Coding Standards

This directory contains language-specific coding standards for all EWB software projects. These standards exist to ensure code is readable, maintainable, and consistent across teams — not to enforce style for its own sake.

---

## Contents

| Document | Stack |
|---|---|
| [dotnet-coding-standards.md](dotnet-coding-standards.md) | C# / .NET 8 |
| [react-coding-standards.md](react-coding-standards.md) | React / TypeScript |
| [node-coding-standards.md](node-coding-standards.md) | Node.js / TypeScript |
| [java-coding-standards.md](java-coding-standards.md) | Java 21 |
| [sql-coding-standards.md](sql-coding-standards.md) | SQL (all databases) |

---

## General Principles

These principles apply across all languages and frameworks:

**Clarity over cleverness.** Code is read many more times than it is written. Prefer explicit, readable code over compact or clever solutions.

**Consistent naming.** Names should reveal intent. Abbreviations should be avoided unless universally understood in the domain.

**Small, focused functions.** A function or method should do one thing. If a function requires a comment to explain what it does, it is probably doing too much.

**No dead code.** Commented-out code and unreachable code must not be committed. Version control provides history.

**Tests as documentation.** Test names and structure should communicate the expected behaviour of the code under test.

---

## Tooling

Where available, coding standards are enforced automatically via linters and formatters. Do not rely on code review alone to catch style issues.

| Stack | Linter | Formatter |
|---|---|---|
| C# / .NET | Roslyn Analyzers, EditorConfig | `dotnet format` |
| React / TypeScript | ESLint + TypeScript ESLint | Prettier |
| Node.js / TypeScript | ESLint + TypeScript ESLint | Prettier |
| Java | Checkstyle | Google Java Format |
| SQL | Manual review | Manual formatting |

Configure linting and formatting in the project root. Run lint checks in the CI pipeline via the relevant EWB-devops workflow.
