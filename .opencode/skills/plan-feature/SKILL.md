---
name: plan-feature
description: Break down a feature into clear implementation steps before writing any code
---

## What I do

I create structured implementation plans for new features:

- Clarify requirements and edge cases
- Identify affected files and modules
- Break work into ordered, atomic steps
- Flag risks and dependencies early
- Propose test strategies upfront

## When to use me

Use this skill when:
- Starting a new feature that touches multiple files or modules
- The user asks "how should we implement..." or "what would it take to..."
- Before jumping into code on a non-trivial change
- When the scope of a change is unclear

## How I work

1. **Clarify requirements** — Restate what the feature should do. Ask about ambiguous requirements. List edge cases and error scenarios.
2. **Explore the codebase** — Find related code, existing patterns, and integration points. Understand the current architecture before proposing changes.
3. **Identify the scope** — List every file that will be created or modified. Group by concern.
4. **Break into steps** — Order the work so each step is independently testable. Each step should produce a working intermediate state.
5. **Define the test strategy** — For each step, state what to test and how (unit, integration, manual).
6. **Flag risks** — Call out breaking changes, migration needs, performance implications, or security concerns.

## Output format

```markdown
## Feature: [name]

### Requirements
- [requirement 1]
- [requirement 2]

### Scope
- New: [files to create]
- Modified: [files to change]

### Steps
1. **[step name]** — [what to do]
   - Files: [list]
   - Test: [what to verify]

2. **[step name]** — [what to do]
   ...

### Risks
- [risk and mitigation]
```

## Guidelines

- Never start coding until the plan is confirmed
- Prefer small, reversible steps over large atomic changes
- Reuse existing patterns and utilities — don't reinvent
- If a step is risky, suggest how to reduce the risk (feature flag, parallel implementation, etc.)