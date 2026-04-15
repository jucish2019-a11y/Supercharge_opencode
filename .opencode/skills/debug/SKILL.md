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

## Key principles

- Never skip reproduction — if the bug can't be reproduced, say so
- State hypotheses before making changes
- Fix one thing at a time
- Always add or update a test that catches the bug
- If the fix is unclear, add diagnostic logging first rather than guessing