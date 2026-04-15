---
name: ci-pipeline
description: Set up and optimize CI/CD pipelines, fix build failures, and improve build performance
---

## What I do

I configure and optimize CI/CD pipelines:

- Set up new CI pipelines from scratch
- Fix failing builds and flaky tests
- Optimize build times with caching and parallelism
- Configure deployment pipelines with proper gating

## When to use me

Use this skill when:
- Setting up CI for a new project
- Debugging a failing CI build
- Reducing build time that has become too slow
- Adding deployment stages or environments
- Configuring branch protection and merge requirements

## How I work

1. **Understand the project** — Identify language, framework, package manager, test runner, lint command, build command. Check for existing CI config.
2. **Choose the right structure** — Based on project needs:
   - **Simple**: Lint → Test → Build
   - **With deploy**: Lint → Test → Build → Deploy (staging → production)
   - **Monorepo**: Affected-project detection → per-project pipeline
3. **Implement the pipeline** — Write the CI config file(s). Follow the platform's conventions (GitHub Actions, GitLab CI, CircleCI, etc.).
4. **Add caching** — Cache package manager directories, build artifacts, and test results. Use content-hash-based cache keys.
5. **Parallelize** — Run independent jobs in parallel (lint + typecheck + test). Split slow test suites across workers.
6. **Set up gating** — Required status checks for merge. Branch protection rules. Deploy approvals for production.
7. **Optimize** — Measure build time. Eliminate unnecessary steps. Use incremental builds. Skip CI for docs-only changes when appropriate.

## Pipeline stages (in order)

1. **Lint & format** — Fast, catches style issues early
2. **Type check** — Fast, catches type errors
3. **Unit tests** — Medium speed, validates logic
4. **Integration tests** — Slower, validates components together
5. **Build** — Produces deployable artifact
6. **Deploy** — Staged: preview → staging → production

## Common fixes

- **Flaky tests**: Add retry with `retry: N` for known-flaky tests, but create issues to fix them properly
- **Dependency install slow**: Cache `node_modules`/`.cache`/`venv`, use lockfile-based cache keys
- **Build too slow**: Split jobs, use matrix for parallel test runs, cache build artifacts between stages
- **Secrets in CI**: Use CI secret variables, never hardcode in config

## Guidelines

- PRs should get fast feedback — run the fast checks first
- Never skip tests to fix CI — fix the test or mark it as known-failure with a tracking issue
- Cache aggressively but use proper cache keys to avoid stale builds
- Keep secrets out of CI config — use the platform's secret management
- Document the CI setup in the project README or AGENTS.md