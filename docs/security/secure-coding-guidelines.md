# Secure Coding Guidelines

These guidelines apply to all EWB software projects regardless of technology stack. They are aligned with the OWASP Top 10 and Microsoft Secure Coding Guidelines.

---

## Input Validation

All data entering the system from external sources — HTTP requests, file uploads, message queues, user input — must be validated before use.

**Do:**
- Validate type, format, length, and range
- Use allowlists (accepted values) rather than denylists (rejected values) where possible
- Reject invalid input at the earliest point — do not attempt to sanitize and continue

**Do not:**
- Trust data from any external source, including other internal services
- Rely on client-side validation as the only validation layer
- Pass unvalidated input to SQL queries, file paths, shell commands, or serialization routines

---

## SQL Injection Prevention

Never concatenate user input into SQL query strings.

**Wrong:**

```csharp
var query = "SELECT * FROM Users WHERE Username = '" + username + "'";
```

**Correct — use parameterized queries:**

```csharp
var user = await context.Users
    .Where(u => u.Username == username)
    .FirstOrDefaultAsync();
```

```java
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE username = ?");
stmt.setString(1, username);
```

```javascript
const user = await db.query('SELECT * FROM users WHERE username = $1', [username]);
```

---

## Cross-Site Scripting (XSS) Prevention

Never render user-supplied data in HTML without encoding.

- In React: JSX escapes values by default — do not use `dangerouslySetInnerHTML` with user content
- In .NET Razor: `@variable` escapes HTML — do not use `@Html.Raw()` with user content
- For rich text input: use a well-maintained sanitization library (e.g. DOMPurify for JavaScript)

---

## Authentication and Authorisation

- Use established authentication libraries — do not implement custom authentication mechanisms
- Enforce authorisation on every protected route and resource — never rely solely on UI-level hiding
- Validate authorisation server-side on every request, not just on login
- Use short-lived access tokens; implement refresh token rotation
- Log all authentication events (success and failure) with sufficient context

---

## Sensitive Data Handling

- Never log passwords, tokens, credit card numbers, national identity numbers, or health data
- Mask or truncate sensitive values before logging (e.g. last 4 digits only)
- Encrypt sensitive data at rest using approved algorithms (AES-256 or stronger)
- Use HTTPS for all communications — do not allow HTTP fallback in production
- Set appropriate HTTP security headers: `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`

---

## Dependency Management

- Keep dependencies up to date — outdated packages are a leading source of vulnerabilities
- Review dependency changes in pull requests
- Run automated dependency scanning in CI (see [dependency-scanning.md](dependency-scanning.md))
- Do not add dependencies without understanding their purpose and provenance

---

## Secrets and Credentials

Never commit secrets to source control. See [secrets-management.md](secrets-management.md) for the full policy.

---

## Error Handling

- Return generic error messages to clients — do not expose stack traces, internal paths, or database errors
- Log the full error detail internally for debugging purposes
- Use structured logging with correlation IDs to trace errors across services
- Handle all exceptions — unhandled exceptions can cause information disclosure or denial of service

---

## File Handling

- Validate file type by content (magic bytes) not by extension
- Restrict file upload size and type at the application layer
- Do not execute uploaded files
- Store uploaded files outside the web root
- Use unpredictable identifiers for stored files to prevent direct enumeration

---

## Cryptography

- Use `BCrypt`, `Argon2`, or `PBKDF2` for password hashing — never MD5, SHA1, or unsalted SHA256
- Use `AES-256-GCM` or `ChaCha20-Poly1305` for symmetric encryption
- Use `RSA-2048` or higher, or `ECDSA P-256`, for asymmetric operations
- Never implement cryptographic algorithms from scratch
- Store cryptographic keys outside application code — use a key management service

---

## Code Review Security Checklist

Use this checklist during pull request review:

- [ ] No secrets or credentials in the diff
- [ ] User input is validated before use
- [ ] SQL queries use parameterized statements
- [ ] Authorisation checked on all protected operations
- [ ] No sensitive data in log statements
- [ ] Error responses do not expose internal details
- [ ] No new critical dependency vulnerabilities introduced
