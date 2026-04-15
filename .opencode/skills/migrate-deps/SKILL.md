---
name: migrate-deps
description: Safely upgrade or replace dependencies: check compatibility, update code, and verify
---

## What I do

I handle dependency migrations safely:

- Check compatibility between current and target versions
- Identify breaking changes from changelogs and release notes
- Update all usage sites in the codebase
- Run tests to verify nothing broke
- Clean up unused dependencies

## When to use me

Use this skill when:
- Upgrading a dependency to a new major/minor version
- Replacing one dependency with another (e.g., moment → date-fns)
- Updating lockfiles after version conflicts
- Removing deprecated dependencies
- Responding to vulnerability advisories in dependencies

## How I work

1. **Identify current state** — Check package manifest and lockfile for current versions. Find all import/usage sites.
2. **Research the target** — Read the changelog, migration guide, or release notes for the new version. Identify breaking changes.
3. **Plan the migration** — List each breaking change and the files affected.
4. **Update the dependency** — Change the version in the manifest. Run install.
5. **Fix breaking changes** — Update each usage site. Handle API changes, renamed exports, changed behaviors.
6. **Run tests** — Execute the full test suite. Fix any failures.
7. **Clean up** — Remove unused imports, dead code from the old API, and unused transitive dependencies.
8. **Verify** — Run lint, typecheck, and the application if possible.

## Safety guidelines

- Always check if the new version is stable (not alpha/beta/rc) unless explicitly requested
- Create a backup or commit current state before starting
- Migrate one dependency at a time
- If tests can't verify a change, flag it for manual testing
- Never downgrade other dependencies to resolve conflicts without asking