---
name: config-setup
description: Configure development tooling — linters, formatters, type checkers, editors, and CI — to work together consistently
---

## What I do

I configure development tools so they work together without conflicts:

- **Linters** — ESLint, Ruff, Flake8, golangci-lint configured for the project's needs
- **Formatters** — Prettier, Black, gofmt, auto-formatting on save
- **Type checkers** — TypeScript strict mode, mypy strict, proper config
- **Editor config** — `.editorconfig`, VS Code settings, recommended extensions
- **Git hooks** — Pre-commit checks that run fast and catch real issues
- **CI alignment** — Same checks in CI as in local hooks, same config

## When to use me

Use this skill when:
- Setting up linting/formatting for a new or existing project
- Fixing tool conflicts (linter disagrees with formatter, etc.)
- Enabling strict type checking incrementally
- Adding pre-commit hooks to a project
- Aligning local dev tooling with CI pipelines

## How I work

1. **Discover existing config** — Find all config files: `.eslintrc*`, `.prettierrc*`, `tsconfig*.json`, `pyproject.toml`, `.flake8`, `.mypy.ini`, `.editorconfig`, `.vscode/settings.json`. Understand what's already set up.
2. **Identify gaps and conflicts**:
   - Linter and formatter must agree (e.g., ESLint + Prettier without conflicting rules)
   - Type checker level matches the project's maturity
   - CI runs the same commands as local hooks
3. **Configure holistically** — All tools should share the same understanding of the project:
   - Same file patterns to include/exclude
   - Same line length, indent style, quote style
   - Same TypeScript/Python version targets
4. **Start strict, allow opt-out** — Enable strict rules globally, use directed `eslint-disable`/`# type: ignore` only where needed with explanations.
5. **Add speed optimizations** — Caching, incremental checks, parallel execution, file filtering (only lint changed files in CI).
6. **Document** — Add scripts in `package.json`/`Makefile`/`justfile` for each check. Document how to run them.

## Configuration matrix

| Tool | Config file | Key settings |
|------|------------|--------------|
| ESLint | `.eslintrc.js` or `eslint.config.js` | `extends`, `rules`, `overrides` |
| Prettier | `.prettierrc` | `printWidth`, `semi`, `singleQuote` |
| TypeScript | `tsconfig.json` | `strict`, `noUncheckedIndexedAccess` |
| Ruff | `pyproject.toml [tool.ruff]` | `line-length`, `select`, `ignore` |
| mypy | `pyproject.toml [tool.mypy]` | `strict`, `plugins` |
| .editorconfig | `.editorconfig` | `indent_style`, `indent_size`, `end_of_line` |

## Key principles

- Formatter wins arguments — linter should not have formatting rules if a formatter exists
- All config files must agree on line length and quote style
- Pre-commit hooks must run in under 10 seconds — use caching and file filtering
- CI runs the exact same commands as pre-commit hooks
- Never use `any` / `# type: ignore` without a comment explaining why
- Use `eslint-config-prettier` (or equivalent) to disable formatter-conflicting rules

## Anti-patterns I avoid

- Linter rules that contradict the formatter
- `strict: false` in tsconfig when the project has types
- Pre-commit hooks that take >30 seconds
- Config files that duplicate settings (DRY for config too)
- Ignoring entire files/directories instead of specific rules