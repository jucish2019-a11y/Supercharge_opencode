---
name: onboarding
description: Create project setup docs, AGENTS.md, and contributor guides for new developers
---

## What I do

I create onboarding documentation for projects:

- Generate or update AGENTS.md for AI agent context
- Write setup guides for new developers
- Document build, test, and lint commands
- Create contribution guides with conventions and workflows

## When to use me

Use this skill when:
- Creating or updating AGENTS.md for a project
- Writing onboarding docs for new team members
- Setting up a new project that needs contributor documentation
- When the project has undocumented setup steps or conventions

## How I work

1. **Explore the codebase** — Understand the project structure, tech stack, build tools, test runner, lint config, and key conventions.
2. **Identify setup requirements** — Find: required runtime versions, environment variables, dependency installation, database setup, seed data. Try running the project to verify steps.
3. **Document commands** — Record the exact commands for:
   - Installing dependencies
   - Running in development mode
   - Running tests (unit, integration, e2e)
   - Running lint and type checks
   - Building for production
4. **Document conventions** — Capture coding patterns, naming conventions, file structure, commit message style, PR workflow.
5. **Write the document** — Use the project's existing documentation format. For AGENTS.md, follow the standard format.

## AGENTS.md template

```markdown
# Project Name

## Overview
Brief description of what this project does and its architecture.

## Tech Stack
- Language: X
- Framework: Y
- Database: Z
- Package manager: X

## Commands
- Install: `cmd`
- Dev: `cmd`
- Test: `cmd`
- Lint: `cmd`
- Typecheck: `cmd`
- Build: `cmd`

## Project Structure
Brief explanation of key directories and their purposes.

## Conventions
- Naming: (e.g., camelCase for variables, PascalCase for components)
- Commits: (e.g., conventional commits)
- Exports: (e.g., named exports, default exports)
- Testing: (e.g., colocated tests, __tests__ directories)

## Key Patterns
Patterns that are important to follow in this codebase.
```

## Guidelines

- Verify every command actually works before documenting it
- Be specific: exact commands, exact versions, exact file paths
- Include environment variable names but never their values (secrets)
- Keep it concise — onboarding docs should be scannable
- Update when build tools, commands, or conventions change
- For AGENTS.md particularly: focus on what the AI needs to know to work effectively