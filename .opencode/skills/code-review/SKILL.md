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

## Security review checklist

```
[CRITICAL] Hardcoded secret in source code
  src/config.ts:15
  API key visible in source code.
  Fix: Move to environment variable.

[CRITICAL] SQL injection risk
  src/db/queries.ts:42
  User input concatenated directly into SQL query.
  Fix: Use parameterized queries.

[WARNING] Missing auth check
  src/api/users.ts:88
  Endpoint returns user data without verifying authentication.
  Fix: Add auth middleware.

[WARNING] No input validation
  src/api/orders.ts:23
  Request body passed directly to database without validation.
  Fix: Add Zod schema validation.
```

## Performance review patterns

### N+1 query detection
```ts
// VULNERABLE: N+1 queries
const users = await db.user.findMany();
for (const user of users) {
  const posts = await db.post.findMany({ where: { authorId: user.id } });
  // Each iteration = 1 query. 100 users = 101 queries total
}

// SECURE: Single query with include
const users = await db.user.findMany({
  include: { posts: true },
});
// 1 query total
```

### React re-render detection
```tsx
// VULNERABLE: Object created inline causes re-render
function Parent() {
  return <Child config={{ theme: 'dark' }} />; // New object every render
}

// SECURE: Memoize or define outside
const CONFIG = { theme: 'dark' };
function Parent() {
  return <Child config={CONFIG} />;
}
```

### Bundle size concerns
```ts
// VULNERABLE: Importing entire library
import _ from 'lodash'; // ~70KB

// SECURE: Import only needed functions
import debounce from 'lodash/debounce'; // ~2KB

// Even better: Use native or lighter alternatives
const debounce = (fn: Function, ms: number) => { /* ... */ };
```

## Correctness review checklist

### Null/undefined handling
```ts
// VULNERABLE: Potential null reference
const name = user.profile.name; // profile might be null

// SECURE: Optional chaining
const name = user.profile?.name ?? 'Anonymous';
```

### Race conditions
```ts
// VULNERABLE: Read-modify-write race condition
async function incrementCounter() {
  const current = await db.counter.findUnique({ where: { id: 1 } });
  await db.counter.update({
    where: { id: 1 },
    data: { value: current.value + 1 },
  });
}

// SECURE: Atomic operation
async function incrementCounter() {
  await db.counter.update({
    where: { id: 1 },
    data: { value: { increment: 1 } },
  });
}
```

### Off-by-one errors
```ts
// VULNERABLE: Off-by-one
const lastItem = array[array.length]; // undefined, should be length - 1

// SECURE: Correct indexing
const lastItem = array[array.length - 1];
```

## Style review checklist

### Naming conventions
- Functions: `camelCase`, verbs (`getUser`, `processPayment`)
- Components: `PascalCase` (`UserCard`, `PaymentForm`)
- Constants: `SCREAMING_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- Files: match default export name
- Booleans: prefix with `is`, `has`, `should` (`isLoading`, `hasPermission`)

### Code organization
- Single responsibility per function (max 20-30 lines)
- Early returns over nested conditionals
- Extract magic numbers to named constants
- Remove dead code and unused imports
- Consistent error handling patterns

## Test review checklist

```
For each changed function/method:
- [ ] Happy path tested
- [ ] Edge cases tested (empty, null, boundary values)
- [ ] Error paths tested
- [ ] Async behavior tested (loading, error states)
- [ ] Integration with dependencies mocked appropriately
- [ ] Test names describe behavior, not implementation
- [ ] No test.only or test.skip left in code
```

## Review etiquette

### Comment format
```
**Critical**: Must fix before merge. Blocks functionality or introduces security risk.
**Warning**: Should fix. Impacts maintainability, performance, or robustness.
**Suggestion**: Nice to have. Style, naming, or minor improvements.
**Question**: Seeking clarification. Not necessarily a problem.
**Praise**: Good practice worth acknowledging.
```

### Example review comments
```
[CRITICAL] src/auth.ts:45 — Passwords compared with timing-unsafe `===`
Use `crypto.timingSafeEqual()` to prevent timing attacks.

[WARNING] src/api/users.ts:23 — No rate limiting on password reset
Add rate limiting to prevent brute force attacks.

[SUGGESTION] src/utils.ts:12 — Consider using `URLSearchParams` instead
of manual string concatenation for better readability and safety.

[QUESTION] src/services/payment.ts:78 — Why is `amount` cast to `number`
instead of validated with Zod? Could this accept `NaN`?
```

## Guidelines

- Focus on substantive issues, not cosmetic preferences unless they violate project config
- Always check if existing tests cover the changed code
- Never approve code that exposes secrets or has auth bypasses without flagging as CRITICAL
- When unsure if something is a bug, flag it as WARNING with your reasoning
- Explain the "why" behind suggestions, not just the "what"
- Be respectful and constructive in feedback
- Distinguish between personal preference and project convention
- Check for consistency with existing codebase patterns

## Anti-patterns I avoid

- Approving code with security vulnerabilities (secrets, auth bypasses, injection risks)
- Only checking style without reviewing logic
- Not checking if tests exist or pass
- Being overly pedantic about minor style issues
- Not explaining why a change is requested
- Approving without actually reading the code
- Ignoring context and constraints (deadlines, prototypes)
- Not considering backward compatibility for public APIs