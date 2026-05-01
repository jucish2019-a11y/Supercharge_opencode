---
name: migrate-deps
description: Safely upgrade or replace dependencies: check compatibility, update code, and verify
---

## What I do

I handle dependency migrations safely — upgrading versions, replacing libraries, and verifying nothing breaks:

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

### Checker mode (assessing migration feasibility)

1. **Inventory current usage** — Find all import sites, version pins, and peer dependency constraints.
2. **Check the changelog** — What changed between current and target version? Catalog every breaking change.
3. **Assess blast radius** — How many files import this dependency? How many tests cover its behavior?
4. **Estimate effort** — Classify as trivial (patch), moderate (minor with deprecations), or major (breaking changes).
5. **Flag risks** — Dependencies with no test coverage, runtime-only behavior, or implicit side effects.

### Applier mode (performing the migration)

1. **Identify current state** — Check package manifest and lockfile for current versions. Find all import/usage sites.
2. **Research the target** — Read the changelog, migration guide, or release notes for the new version. Identify breaking changes.
3. **Plan the migration** — List each breaking change and the files affected. Classify each change as: rename, signature change, behavior change, removed API, or new requirement.
4. **Update the dependency** — Change the version in the manifest. Run install.
5. **Fix breaking changes** — Update each usage site. Handle API changes, renamed exports, changed behaviors.
6. **Run tests** — Execute the full test suite. Fix any failures.
7. **Clean up** — Remove unused imports, dead code from the old API, and unused transitive dependencies.
8. **Verify** — Run lint, typecheck, and the application if possible.

## Common migration patterns

### Renamed exports

```
Before: import { oldName } from 'package'
After:  import { newName } from 'package'

Strategy: Find-and-replace all occurrences. If both names exist
in a transition release, alias in a barrel file:

  export { newName as oldName } from 'package'
```

### Changed function signatures

```
Before: fn(requiredArg, optionalArg)
After:  fn(requiredArg, { optionalArg })

Strategy: Update each call site. Use the options object pattern.
If there are many call sites, create a wrapper during transition:

  function fnCompat(requiredArg, optionalArg) {
    return fn(requiredArg, { optionalArg })
  }
```

### Removed APIs

```
Before: import { removedFn } from 'package'
After:  (removed — use replacementFn instead)

Strategy: Search for all usage sites. If the replacement API
exists in the current version, migrate first, then upgrade.
If not, create a polyfill until upgrade is complete.
```

### Package replacement (different library)

```
Before: import moment from 'moment'
After:  import { format, parseISO } from 'date-fns'

Strategy:
1. Create an adapter module that re-exports the new API
   with the old API's function signatures
2. Replace imports one file at a time
3. Run tests after each file migration
4. Remove the adapter once all imports are migrated
5. Remove the old dependency
```

## Migration checklist

- [ ] Read the full changelog between current and target versions
- [ ] List every breaking change and map to affected files
- [ ] Create a Git branch or commit before starting
- [ ] Migrate one dependency at a time (never batch upgrades)
- [ ] Update the version in package manifest
- [ ] Run install to update the lockfile
- [ ] Fix all import/usage sites
- [ ] Run the full test suite
- [ ] Run typecheck (if applicable)
- [ ] Run linter (if applicable)
- [ ] Remove unused imports and dead code
- [ ] Check for peer dependency warnings
- [ ] Verify the application starts and runs correctly
- [ ] Check bundle size impact (if applicable)

## Vulnerability response

When migrating due to a security advisory:

```
1. Check the advisory severity (critical, high, medium, low)
2. Check if the vulnerable code path is actually used
3. If critical or high: upgrade immediately, even if it
   requires a major version jump
4. If medium: plan the upgrade in the next sprint
5. If low: batch with other dependency updates
6. If no fix is available: add a resolution/override in
   package.json to pin a patched version
```

## Anti-patterns I avoid

- Upgrading multiple dependencies at once — when something breaks, you won't know which upgrade caused it
- Skipping the changelog — breaking changes are documented; missing one means mysterious failures
- Using `--force` or `--legacy-peer-deps` to bypass peer dependency warnings — these warnings indicate real incompatibilities
- Removing a dependency's lockfile entry without removing its imports — runtime crashes
- Upgrading without running tests — the tests are what tell you if the migration succeeded
- Ignoring deprecation warnings — they become breaking changes in the next major version
- Pinning to an old version indefinitely because "it works" — security debt accumulates silently