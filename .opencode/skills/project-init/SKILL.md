---
name: project-init
description: Scaffold new projects with proper structure, tooling, and configuration from day one
---

## What I do

I scaffold new projects with production-ready foundations:

- **Project structure** — Sensible directory layout matching language/framework conventions
- **Configuration** — Linter, formatter, type checker, editor config all aligned
- **Toolchain** — Build system, package manager, scripts configured correctly
- **Git** — Initial commit, `.gitignore`, README skeleton
- **CI readiness** — Project ready for continuous integration from the start

## When to use me

Use this skill when:
- Starting a brand new project from scratch
- Re-initializing a project that lacks proper structure
- Setting up a monorepo workspace
- Creating a new service within an existing organization

## How I work

1. **Clarify the project** — Ask about language, framework, target environment, and key dependencies before generating anything.
2. **Check for org templates** — Look for existing templates, monorepo configs, or shared configs (`tsconfig/base.json`, `.eslintrc.shared.js`, etc.) that should be reused.
3. **Generate the structure** — Create directories and files following the framework's community conventions (Next.js app router, Django project layout, Go standard layout, etc.).
4. **Configure tooling** — Set up all development tools to work together without conflicts:
   - Linter + formatter (never fight each other)
   - Type checker with strict mode
   - Test runner with sensible defaults
   - Git hooks for pre-commit checks
5. **Add scripts** — `package.json` scripts, `Makefile` targets, or `justfile` recipes for common tasks (dev, build, test, lint, typecheck).
6. **Initialize git** — `.gitignore` for the stack, initial commit.
7. **Verify** — Run the project to confirm it starts. Run linter and tests to confirm they pass.

## Project structure templates

### Node.js / TypeScript
```
project/
  src/
    index.ts
  tests/
  package.json
  tsconfig.json
  .eslintrc
  .prettierrc
  .gitignore
```

### Python
```
project/
  src/package/
    __init__.py
  tests/
    conftest.py
  pyproject.toml
  .gitignore
```

### Go
```
project/
  cmd/
  internal/
  pkg/
  go.mod
  go.sum
  .gitignore
```

### Full-stack monorepo
```
monorepo/
  apps/
    web/
    api/
  packages/
    shared/
  package.json
  turbo.json (or nx.json)
```

## Key principles

- Every project should be able to build, test, and lint from minute one
- Never check in secrets — use `.env.example` with placeholder values
- Pin dependency versions in lock files, never in range in `package.json`/`pyproject.toml`
- Use the language's standard package manager (npm, poetry, go modules)
- Editor config (`.editorconfig`) ensures consistency across contributors
- Pre-commit hooks catch issues before they reach CI

## Anti-patterns I avoid

- Checking in `node_modules/`, `__pycache__/`, `.venv/`, build artifacts
- Missing `.gitignore` entries
- Unpinned or ranged dependency versions without a lock file
- Mixing concerns in root directory (src files next to config)
- Skipping the initial "hello world" verification