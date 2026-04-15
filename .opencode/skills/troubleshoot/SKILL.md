---
name: troubleshoot
description: Systematic troubleshooting: read errors, check logs, isolate root cause, and apply fixes
---

## What I do

I troubleshoot issues systematically:

- Parse error messages and logs accurately
- Isolate the failing component or configuration
- Identify root cause through elimination
- Apply targeted fixes
- Verify the fix resolves the issue

## When to use me

Use this skill when:
- A build is failing and the error message is unclear
- Tests are failing in CI but not locally (or vice versa)
- The application crashes or hangs at startup or runtime
- Environment or configuration issues
- Dependency resolution failures
- Toolchain errors (compiler, bundler, linter)

## How I work

1. **Read the error** — Capture the full error output. Parse the error type, message, stack trace, and any error codes. Look for the most specific line, not just the first or last.
2. **Reproduce** — Attempt to reproduce the issue. If the error only happens in certain environments, identify what differs.
3. **Check common causes first** — Work through likely causes in order of probability:
   - Missing or misconfigured dependencies
   - Version conflicts
   - Environment variable issues
   - File path or permission problems
   - Configuration typos
   - Port conflicts
   - Resource exhaustion (memory, disk, connections)
4. **Isolate** — Narrow down by removing variables. Comment out code, use minimal configs, test in isolation.
5. **Fix** — Apply the minimal change that resolves the issue.
6. **Verify** — Run the failing command again. Run the full test suite if applicable.

## Quick diagnostic checks

- `git status` — any uncommitted changes causing issues?
- `git diff` — what changed recently?
- Check lockfiles — are dependencies in sync?
- Check Node/Python/etc. version — does it match expected?
- Check environment variables — are required ones set?
- Check file permissions — can the process read/write needed files?
- Check running processes — port already in use?

## Guidelines

- Never skip reading the full error message
- Try the simplest fix first
- If a fix doesn't work, revert before trying the next
- Document what was tried and what failed — avoid repeating steps
- If the issue is environment-specific, note what makes the environment different