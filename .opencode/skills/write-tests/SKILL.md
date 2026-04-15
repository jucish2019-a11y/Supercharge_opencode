---
name: write-tests
description: Generate thorough tests following project conventions and best practices
---

## What I do

I write comprehensive tests for code:

- Discover the project's test framework and conventions
- Cover happy paths, edge cases, and error paths
- Follow existing test structure and naming patterns
- Ensure tests are deterministic and isolated

## When to use me

Use this skill when:
- Writing tests for new or existing code
- Increasing test coverage for a module
- Adding regression tests after fixing a bug
- Setting up test infrastructure for a new project

## How I work

1. **Discover conventions** — Find the test framework, runner config, assertion style, directory structure, and naming patterns by examining existing tests.
2. **Identify what to test** — For each function/method: happy path inputs, boundary values, invalid inputs, error conditions, side effects.
3. **Follow existing patterns** — Match import style, describe/it structure, setup/teardown, mocking approach, and assertion style of existing tests.
4. **Write isolated tests** — Each test should be independent. No shared mutable state between tests. Mock external dependencies (network, filesystem, time) when needed.
5. **Make tests deterministic** — Use fixed values, not random or time-dependent ones. Control async ordering where relevant.
6. **Run and verify** — Execute the tests and confirm they pass. Fix any failures.

## Test structure per case

- **Unit tests**: Test one function in isolation. Mock dependencies.
- **Integration tests**: Test multiple modules together. Use real dependencies when possible.
- **Regression tests**: Write a test that reproduces the bug first (should fail), then verify it passes after the fix.

## Guidelines

- Always check for existing tests before writing new ones
- Name tests descriptively: "should [expected behavior] when [condition]"
- Test error paths, not just happy paths
- Avoid testing implementation details — test behavior
- Never skip tests with TODO — write the test or don't create it
- Run lint and typecheck after writing tests