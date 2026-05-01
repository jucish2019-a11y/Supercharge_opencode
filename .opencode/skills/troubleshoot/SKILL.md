---
name: troubleshoot
description: Systematic troubleshooting: read errors, check logs, isolate root cause, and apply fixes
---

## What I do

I troubleshoot issues systematically — from reading error messages to isolating root cause to applying minimal fixes:

- Parse error messages and logs accurately
- Isolate the failing component or configuration
- Identify root cause through elimination
- Apply targeted fixes (not shotgun debugging)
- Verify the fix resolves the issue without introducing new ones

## When to use me

Use this skill when:
- A build is failing and the error message is unclear
- Tests are failing in CI but not locally (or vice versa)
- The application crashes or hangs at startup or runtime
- Environment or configuration issues
- Dependency resolution failures
- Toolchain errors (compiler, bundler, linter)
- Performance issues or unexpected behavior

## How I work

### Checker mode (assessing an issue before diving in)

1. **Read the full error** — Capture every line. The most specific line is rarely the first or last.
2. **Classify the error** — Is this a build error, runtime error, test failure, or configuration issue?
3. **Check recency** — What changed recently? `git diff HEAD~5`, recent deployments, dependency updates.
4. **Assess scope** — Is this one feature broken, or the whole application?
5. **Estimate blast radius** — Can other people reproduce it? Is it environment-specific?

### Applier mode (fixing the issue)

1. **Read the error** — Capture the full error output. Parse the error type, message, stack trace, and any error codes. Look for the most specific line, not just the first or last.
2. **Reproduce** — Attempt to reproduce the issue. If the error only happens in certain environments, identify what differs.
3. **Check common causes first** — Work through likely causes in order of probability.
4. **Isolate** — Narrow down by removing variables. Comment out code, use minimal configs, test in isolation.
5. **Fix** — Apply the minimal change that resolves the issue.
6. **Verify** — Run the failing command again. Run the full test suite if applicable.

## Error classification and common causes

### Build/compilation errors

```
Common causes:
  - Missing or misconfigured dependencies (did you run install?)
  - Version conflicts (check lockfile vs manifest)
  - Type errors (run tsc --noEmit separately)
  - Circular imports (look for "circular dependency" warnings)
  - Missing environment variables at build time
  - Outdated build cache (try clearing .next/, dist/, node_modules/.cache/)

Quick diagnostic:
  1. Delete node_modules and reinstall: rm -rf node_modules && npm install
  2. Clear build cache: rm -rf .next dist .turbo
  3. Check Node version: node --version (match .nvmrc or engines field?)
  4. Check package manager: are you using the right one? (npm vs yarn vs pnpm)
```

### Runtime errors

```
Common causes:
  - Null/undefined access (most common runtime error)
  - Missing environment variables
  - Network errors (CORS, timeout, DNS)
  - Permission issues (file system, database)
  - State management bugs (stale closure, race condition)
  - Memory issues (heap overflow, object retained in closure)

Quick diagnostic:
  1. Read the stack trace bottom-up (your code → library → framework)
  2. Find the first frame in YOUR code — that's the investigation start
  3. Add console.log/logging BEFORE the error line, not after
  4. Check if the error is intermittent (race condition) vs consistent (logic bug)
```

### Test failures

```
Common causes:
  - Flaky tests (timing, network, random data)
  - Environment differences (CI vs local: timezone, locale, CPU cores)
  - Test order dependencies (tests that pass alone but fail in suite)
  - Stale snapshots or fixtures
  - Mock drift (mocks don't match actual API)

CI passes, local fails:
  - Check Node version matches CI
  - Check .env variables (CI may set different ones)
  - Run with same test isolation (--runInBand for Jest)

Local passes, CI fails:
  - Check for hardcoded paths (/Users/you/...)
  - Check for time-dependent tests (use Date.now mock)
  - Check test parallelism (race conditions exposed by CI)
  - Check for missing test fixtures (not committed to git)
```

### Environment/configuration errors

```
Common causes:
  - Missing .env file or wrong .env values
  - Wrong Node.js version (use nvm or volta)
  - Port already in use (lsof -i :3000 or netstat -tlnp | grep 3000)
  - File permissions (chmod, chown)
  - Disk space (df -h)
  - DNS resolution (try IP instead of hostname)

Quick diagnostic:
  1. echo $ENV_VAR — is it set?
  2. node --version && npm --version — match expected?
  3. curl http://localhost:3000 — is something already running?
  4. ls -la — are permissions correct?
```

## Isolation techniques

### Binary search debugging

When a large change introduced a bug:

```bash
# Find the exact commit that introduced the issue
git bisect start
git bisect bad          # current commit is broken
git bisect good abc123  # this commit was working
# Git will checkout a middle commit. Test it.
# Then: git bisect good (if it works) or git bisect bad (if it doesn't)
# Repeat until git identifies the culprit commit
```

### Minimal reproduction

Strip away everything that doesn't cause the error:

```
1. Comment out half the code
2. Does the error still occur?
   - Yes: the bug is in the remaining half
   - No: the bug is in the commented half
3. Repeat until you've isolated the minimal failing case
4. This gives you the root cause AND a test case
```

### Dependency isolation

```bash
# Check if a dependency causes the issue
# Remove it temporarily and see if the error changes
npm uninstall <suspect-package>
npm install

# Or check the lockfile for recent changes
git diff HEAD~10 package-lock.json
```

## Logging strategies

### Add logging BEFORE the failing line

```
BAD:  console.log("after the error")  // never runs — error thrown before
GOOD: console.log("before the error", { variable, state })  // runs, shows what happened
```

### Log the right data

```typescript
// Bad: too vague
console.log("data:", data);

// Good: specific shape and type
console.log("data type:", typeof data, "keys:", Object.keys(data ?? {}), "value:", JSON.stringify(data, null, 2));

// For errors: log the full error, not just the message
catch (error) {
  // Bad
  console.log("Error:", error);

  // Good
  console.error("Failed to fetch user:", {
    error,
    message: error instanceof Error ? error.message : String(error),
    stack: error instanceof Error ? error.stack : undefined,
    context: { userId, requestId },
  });
}
```

### Use conditional breakpoints in DevTools

For browser bugs:
1. Open DevTools → Sources
2. Find the failing line
3. Right-click the line number → "Add conditional breakpoint"
4. Set a condition like `user.id === 'abc123'` or `items.length > 1000`
5. Step through when it hits

## Platform-specific troubleshooting

### Node.js

```bash
# Check if Node is the issue
node -e "console.log(process.version, process.arch, process.platform)"

# Check memory usage
node -e "console.log(process.memoryUsage())"

# Enable verbose logging
NODE_DEBUG=http,net node server.js

# Check for native module issues
node -e "require('sharp')"  # test if native module loads
```

### Docker

```bash
# Check if the container has the right files
docker exec -it <container> ls -la /app

# Check environment inside the container
docker exec -it <container> env | sort

# Check logs
docker logs <container> --tail 100 --follow

# Shell into the container
docker exec -it <container> /bin/sh

# Check if the port is actually listening
docker exec -it <container> netstat -tlnp
```

### CI/CD

```bash
# Reproduce CI environment locally
# GitHub Actions: act (nektos/act)
act -j build

# Check what CI installed
# Add a step that dumps the environment:
- run: |
    echo "node: $(node --version)"
    echo "npm: $(npm --version)"
    echo "os: $(uname -a)"
    echo "env: $(env | sort)"
```

## Quick diagnostic checklist

Run these before deep investigation:

- [ ] `git status` — any uncommitted changes causing issues?
- [ ] `git diff HEAD~5` — what changed recently?
- [ ] `node --version` — does it match `.nvmrc` or `engines` field?
- [ ] `cat .env.example` — are all required env vars set in `.env`?
- [ ] `npm ls <package>` — is the dependency installed and at the right version?
- [ ] Check lockfile — is `package-lock.json` in sync with `package.json`?
- [ ] Check running processes — is the port already in use?
- [ ] Check disk space — `df -h` (full disk causes cryptic errors)
- [ ] Check file permissions — can the process read/write needed files?
- [ ] Clear caches — delete `node_modules/.cache`, `.next`, `dist`, `.turbo`

## Anti-patterns I avoid

- Skipping the full error message — the answer is usually in the stack trace
- Changing multiple things at once — when something "fixes" it, you won't know which change helped
- Adding logging AFTER the failing line — it never runs if an error is thrown
- Reinstalling everything as a first step — it might work but doesn't tell you why
- Assuming the error is in your code — check the stack trace for library/framework frames
- Ignoring intermittent failures — they're usually race conditions and get worse under load
- "It works on my machine" without documenting why — the difference IS the bug
- Fixing symptoms instead of root cause — if you don't understand why the fix works, it's not a fix, it's a workaround