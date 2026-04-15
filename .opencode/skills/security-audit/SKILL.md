---
name: security-audit
description: Scan for security vulnerabilities including secrets, injection, auth issues, and dependency risks
---

## What I do

I perform security-focused code audits:

- **Secrets exposure**: API keys, passwords, tokens, private keys in source code
- **Injection attacks**: SQL injection, XSS, command injection, path traversal, LDAP injection
- **Authentication & authorization**: Bypass opportunities, missing auth checks, privilege escalation
- **Input validation**: Missing validation, type confusion, prototype pollution, unsafe deserialization
- **Dependency risks**: Known vulnerable packages, outdated dependencies with CVEs
- **Configuration**: Insecure defaults, CORS misconfiguration, missing rate limiting

## When to use me

Use this skill when:
- Before deploying to production
- When adding authentication, authorization, or payment features
- After adding new API endpoints or user input handling
- Periodically for security health checks
- When incorporating new dependencies

## How I work

1. **Identify attack surface** — Find all entry points: API routes, CLI args, file inputs, user forms, webhooks.
2. **Trace untrusted data** — Follow user input from entry to storage/rendering. Check validation at each step.
3. **Check auth patterns** — Verify every sensitive endpoint has auth. Check for IDOR (insecure direct object references). Verify session handling.
4. **Scan for secrets** — Search for hardcoded credentials, API keys, private keys. Check .gitignore covers .env and credential files.
5. **Review dependencies** — Check package manifest for known-vulnerable versions. Suggest updates.
6. **Report findings** — Classify by severity: CRITICAL (immediate exploit risk), HIGH (exploitable with effort), MEDIUM (requires specific conditions), LOW (defense in depth).

## Output format

```
[SEVERITY] category — short description
  file:line
  Attack vector: how this could be exploited
  Remediation: specific fix
```

## Guidelines

- Always flag hardcoded credentials as CRITICAL
- Assume all user input is malicious
- Don't just find issues — provide actionable fixes
- Consider the full attack chain, not just individual weaknesses