---
name: refactor
description: Safe refactoring: identify code smells, plan changes, preserve behavior, and verify
---

## What I do

I refactor code safely by:

- Identifying code smells and improvement opportunities
- Planning changes that preserve existing behavior
- Making incremental, verifiable transformations
- Running tests to confirm no regressions

## When to use me

Use this skill when:
- Code is hard to read or understand
- Duplication exists across modules
- A function or module has too many responsibilities
- A change would be easier after restructuring existing code
- The user asks to "clean up", "simplify", or "reorganize" code

## How I work

1. **Understand current behavior** — Read the code thoroughly. Run existing tests to confirm they pass before starting.
2. **Identify smells** — Look for: long functions, duplicated logic, deep nesting, unclear names, excessive parameters, God objects, feature envy, inappropriate intimacy.
3. **Plan the refactoring** — List each transformation step. Each step should be small enough to verify independently.
4. **Apply one transformation at a time** — Common transformations:
   - Extract function/method
   - Rename for clarity
   - Move code to its proper home
   - Replace conditional with polymorphism
   - Simplify conditional logic
   - Introduce parameter object
5. **Verify after each step** — Run tests after every transformation. If tests fail, revert and reassess.
6. **Clean up** — Remove dead code, unused imports, unreachable branches after transformations are complete.

## Key principles

- **Behavior preservation**: Refactoring must not change observable behavior
- **Small steps**: Each transformation is independently verifiable
- **Test first**: If tests don't exist, write them before refactoring
- **No mixed changes**: Don't refactor and add features in the same change
- **Revert on failure**: If any step breaks tests, revert it rather than patching