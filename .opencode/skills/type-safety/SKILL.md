---
name: type-safety
description: Fix type errors, improve type coverage, and enforce stricter type checking
---

## What I do

I improve type safety in a codebase:

- Fix type errors and eliminate `any` usage
- Add precise types for untyped or loosely typed code
- Configure stricter type checking rules
- Design generic and utility types for reusable patterns

## When to use me

Use this skill when:
- TypeScript/Python/etc. type errors need fixing
- Reducing `any`, `unknown`, or loose type usage
- Setting up or tightening type checker configuration
- Adding types to an untyped JavaScript or Python codebase
- Making a type narrowing issue work correctly

## How I work

1. **Assess current state** — Run the type checker and count errors. Check the config for current strictness level. Identify the most common type issues (any casts, missing types, type assertions).
2. **Prioritize** — Fix errors by impact:
   - Start with `strict` mode violations that affect runtime behavior
   - Then address `any` types in shared/public APIs
   - Then internal `any` types and type assertions
   - Finally, enable stricter config rules incrementally
3. **Fix incrementally** — Enable strict checks one at a time when possible. Fix errors for each new rule before enabling the next. Use `// @ts-expect-error` temporarily to unblock, but track with TODOs.
4. **Write proper types** — Prefer:
   - Union types over `any` for known variants
   - Generic types for reusable abstractions
   - Discriminated unions for state modeling
   - Branded types for domain primitives (not plain `string`)
   - `readonly` for immutable data
5. **Verify** — Run type checker after each batch of fixes. Run tests to ensure runtime behavior is preserved.

## Common type improvements

| Instead of | Use |
|---|---|
| `any` | `unknown` + type narrowing |
| `as SomeType` | Type guard / discriminated union |
| `string` for IDs | Branded type `type UserId = string & { __brand: 'UserId' }` |
| Optional + undefined | Explicit union `string \| undefined` |
| Loose object type | Exact / required utility type |
| Type assertion | Schema validation (zod, io-ts) |

## Guidelines

- Never suppress errors with `any` or `@ts-ignore` without a TODO comment
- Prefer compile-time checks over runtime checks, but use runtime validation at boundaries
- Tighten types from the outside in: public API types first, then internals
- When enabling `strict`, do it incrementally if there are many errors
- Use the project's existing type utilities and patterns