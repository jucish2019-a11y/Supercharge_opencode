---
name: debug
description: Structured debugging workflow: reproduce, isolate, identify root cause, fix, and verify
---

## What I do

I follow a systematic debugging methodology to find and fix bugs efficiently:

- **Reproduce** the issue reliably
- **Isolate** the failing component or code path
- **Identify** the root cause
- **Fix** with minimal, targeted changes
- **Verify** the fix and check for regressions

## When to use me

Use this skill when:
- A test is failing and the cause is unclear
- An error is thrown at runtime with an unclear stack trace
- Behavior is wrong but no error is raised
- A regression appeared after a recent change
- Performance degraded unexpectedly

## How I work

1. **Reproduce** — Confirm the exact steps, inputs, or conditions that trigger the bug. If I can't reproduce it, I say so and ask for more context.
2. **Read the error** — Parse error messages, stack traces, and logs. Identify the failing line and module.
3. **Trace the data flow** — Follow inputs through the code path to find where behavior diverges from expectations. Use `read`, `grep`, and `glob` to trace function calls and data transformations.
4. **Form a hypothesis** — State the most likely root cause before making changes. Prefer the simplest explanation.
5. **Fix minimally** — Change only what's needed. Avoid refactoring while debugging unless the refactoring IS the fix.
6. **Verify** — Run the relevant tests. If no test existed, write one that proves the bug existed and now passes.
7. **Check for regressions** — Run the full test suite if available. Look for similar patterns elsewhere in the codebase.

## Browser DevTools workflow

### Elements panel
- Inspect DOM structure and styles
- Test CSS changes live
- Check computed styles for specificity issues
- Monitor DOM mutations

### Console
```js
// Log with context
console.log('User object:', user);
console.table(arrayOfObjects);
console.group('Processing batch');
console.log('Step 1');
console.log('Step 2');
console.groupEnd();

// Conditional breakpoints
// Right-click line number > Add conditional breakpoint > `i === 5`

// Trace function calls
console.trace('Where was this called from?');

// Measure performance
console.time('operation');
// ... code ...
console.timeEnd('operation');
```

### Network panel
- Check request/response headers and bodies
- Verify status codes and response times
- Inspect WebSocket messages
- Test with throttling (slow 3G simulation)

### Sources panel
- Set breakpoints (click line number)
- Step through code (F10 step over, F11 step into, Shift+F11 step out)
- Watch expressions (add variables to watch)
- Call stack navigation
- Blackbox scripts (skip library code)

### Performance panel
- Record and analyze runtime performance
- Identify long tasks and forced reflows
- Analyze frame rates
- Find memory leaks (take heap snapshots)

### Memory panel
- Take heap snapshots
- Compare snapshots to find leaks
- Analyze detached DOM trees
- Check retained size of objects

### Application panel
- Inspect localStorage/sessionStorage
- Check cookies
- View service workers
- Inspect cache storage

## Node.js debugging

### Built-in debugger
```bash
# Start with inspector
node --inspect-brk server.js

# Or for tests
node --inspect-brk node_modules/.bin/jest --runInBand

# Then open chrome://inspect in Chrome
```

### VS Code launch configuration
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Server",
      "program": "${workspaceFolder}/server.ts",
      "runtimeArgs": ["-r", "ts-node/register"],
      "env": { "NODE_ENV": "development" }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Tests",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand", "--testPathPattern", "${file}"],
      "console": "integratedTerminal"
    }
  ]
}
```

## React DevTools

### Components tab
- Inspect component hierarchy
- View props and state
- Identify unnecessary re-renders (highlight updates)
- Profile component render times

### Profiler tab
- Record performance profile
- Identify slow components
- Check why a component rendered (props/state change)
- Analyze commit durations

## Common bug patterns

### JavaScript
```js
// Bug: Reference equality vs value equality
const a = { x: 1 };
const b = { x: 1 };
console.log(a === b); // false (different references)

// Bug: Closure in loops
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Logs 3, 3, 3
}
// Fix: Use let or IIFE
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Logs 0, 1, 2
}

// Bug: Async/await in loops
// Wrong: parallel execution, not sequential
items.forEach(async (item) => {
  await process(item);
});

// Fix: Use for...of for sequential
for (const item of items) {
  await process(item);
}

// Bug: Floating point arithmetic
0.1 + 0.2 === 0.3 // false
// Fix: Use epsilon comparison or decimal libraries
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON

// Bug: Array mutation methods
const arr = [1, 2, 3];
const reversed = arr.reverse(); // Mutates original!
// Fix: Copy first
const reversed = [...arr].reverse();
```

### React
```tsx
// Bug: Missing dependency in useEffect
useEffect(() => {
  fetchData(userId);
}, []); // Missing userId dependency

// Bug: Stale closure in useCallback
const handleClick = useCallback(() => {
  console.log(count); // Stale if count changes
}, []); // Missing count dependency

// Bug: Not cleanup on unmount
useEffect(() => {
  const subscription = subscribe();
  // Missing return cleanup
}, []);

// Bug: Setting state from props without key
function UserProfile({ user }) {
  const [name, setName] = useState(user.name); // Won't update when user changes
  // Fix: Use key prop or useEffect
}
```

## Diagnostic logging

```ts
// Structured logging for debugging
function debugLog(
  operation: string,
  data: Record<string, unknown>,
  level: 'info' | 'warn' | 'error' = 'info'
) {
  if (process.env.DEBUG !== 'true') return;

  console[level](JSON.stringify({
    timestamp: new Date().toISOString(),
    operation,
    ...data,
  }, null, 2));
}

// Usage
debugLog('processOrder', { orderId, userId, items: items.length });
```

## Binary search debugging (git bisect)

```bash
# When a bug was introduced recently
git log --oneline -20  # Find a known good commit
git bisect start
git bisect bad HEAD     # Current is bad
git bisect good abc123  # Known good commit

# Git will checkout middle commit, test, then:
git bisect good   # or bad
# Repeat until found

git bisect reset  # When done
```

## Memory leak detection

```js
// In browser console
// 1. Take heap snapshot
// 2. Perform suspicious action
// 3. Take another snapshot
// 4. Compare: look for growing object counts

// Common causes:
// - Event listeners not removed
// - setInterval not cleared
// - Closure capturing large objects
// - DOM references preventing garbage collection
// - Subscription not unsubscribed
```

## Key principles

- Never skip reproduction — if the bug can't be reproduced, say so
- State hypotheses before making changes
- Fix one thing at a time
- Always add or update a test that catches the bug
- If the fix is unclear, add diagnostic logging first rather than guessing
- Use version control to your advantage — git diff, git bisect
- Check environment differences (local vs CI, Node versions)
- Look for the simplest explanation first (Occam's razor)

## Guidelines

- Start with the error message and stack trace
- Check if the bug is reproducible in a minimal environment
- Isolate the problem by removing unrelated code
- Use logging over breakpoints for timing-related bugs
- Test fixes thoroughly before committing
- Document the root cause in commit messages
- Look for similar patterns elsewhere in the codebase

## Anti-patterns I avoid

- Changing multiple things at once without testing each
- Assuming the bug is in a specific area without evidence
- Ignoring the stack trace and guessing
- Fixing symptoms instead of root causes
- Not writing tests that reproduce the bug
- Using console.log instead of proper debugging tools for complex issues
- Not checking if the issue exists in other environments
- Making large refactoring changes while debugging
- Not documenting the fix and root cause