---
name: code-review
description: Systematic code review checking for bugs, security issues, style violations, missing tests, and performance concerns
---

## What I do

I perform structured code reviews across the following dimensions:

- **Correctness**: Logic errors, off-by-one errors, null/undefined handling, race conditions
- **Security**: Injection vulnerabilities, authentication bypasses, secret exposure, input validation gaps
- **Style**: Consistency with project conventions, naming, formatting, dead code
- **Tests**: Missing test coverage for new/changed logic, edge cases, error paths
- **Performance**: Unnecessary allocations, N+1 queries, missing indexes, excessive re-renders

## When to use me

Use this skill when:
- Reviewing changes before committing
- Reviewing a pull request
- Auditing an existing module that may have issues
- After implementing a feature to catch mistakes before they ship

## How I work

1. **Read the diff or files** — Understand what changed and why
2. **Check project conventions** — Look at nearby code, linting config, existing patterns
3. **Review by dimension** — Systematically evaluate correctness, security, style, tests, performance
4. **Prioritize findings** — Critical (must fix) > Warning (should fix) > Suggestion (nice to have)
5. **Report concisely** — List findings with file:line references and actionable fixes

## Output format

For each finding:

```
[SEVERITY] category — short description
  file:line
  Explanation of the issue.
  Suggested fix (code snippet if helpful).
```

Severities: `CRITICAL`, `WARNING`, `SUGGESTION`

## Guidelines

- Focus on substantive issues, not cosmetic preferences unless they violate project config
- Always check if existing tests cover the changed code
- Never approve code that exposes secrets or has auth bypasses without flagging as CRITICAL
- When unsure if something is a bug, flag it as WARNING with your reasoning