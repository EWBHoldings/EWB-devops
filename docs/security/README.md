# Security Documentation

This directory contains security standards, guidelines, and operational practices for all EWB software projects.

---

## Contents

| Document | Description |
|---|---|
| [secure-coding-guidelines.md](secure-coding-guidelines.md) | OWASP-aligned secure coding practices for all technology stacks |
| [secrets-management.md](secrets-management.md) | How to manage credentials, API keys, and sensitive configuration |
| [dependency-scanning.md](dependency-scanning.md) | Tooling and processes for identifying vulnerable dependencies |

---

## Security Principles

All EWB software is built on the following security principles:

**Defence in depth** — No single control is relied upon. Multiple layers of security controls reduce the impact of any single failure.

**Least privilege** — Services, users, and processes are granted only the permissions they need to function. Nothing more.

**Secure by default** — New projects and services are configured securely out of the box. Security is not an afterthought.

**Fail securely** — When something fails, it should fail in a way that does not expose sensitive data or grant unintended access.

**Zero trust** — No network location or service is implicitly trusted. All requests are authenticated and authorised.

---

## Reporting a Security Issue

Do not open public GitHub issues for security vulnerabilities. Report security concerns to the DevOps or security team through the internal reporting channel.

Provide:
- A description of the vulnerability
- Steps to reproduce
- Potential impact assessment
- Suggested remediation (if known)
