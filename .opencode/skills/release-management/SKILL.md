---
name: release-management
description: Manage software releases with versioning, changelogs, tags, and coordinated deployment
---

## What I do

I manage the release process from code freeze to production deployment:

- **Versioning** — Semantic versioning, calendar versioning, or custom schemes
- **Changelog** — Auto-generated change logs from commit history
- **Release branches** — Stabilization, cherry-picks, and hotfixes
- **Tagging** — Git tags, GitHub releases, artifact versioning
- **Coordination** — Release checklists, communication, rollback plans

## When to use me

Use this skill when:
- Preparing a new version release
- Creating a release branch for stabilization
- Writing release notes or changelogs
- Managing hotfixes on a previous release
- Setting up a release automation pipeline

## How I work

1. **Determine the version** — Based on the changes since last release:
   - `MAJOR`: Breaking changes
   - `MINOR`: New features, backward-compatible
   - `PATCH`: Bug fixes, backward-compatible
2. **Gather changes** — Collect commits since the last tag. Classify as features, fixes, breaking changes, or chores.
3. **Create release branch** (if applicable) — `release/v{version}` from main. Only bug fixes and critical changes from this point.
4. **Generate changelog** — Organize changes by category, include migration notes for breaking changes.
5. **Freeze and stabilize** — No new features on the release branch. Only cherry-picked fixes.
6. **Tag and release** — Git tag, GitHub/GitLab release, artifact publication.
7. **Post-release** — Merge release branch back to main, update version numbers, announce.

## Version bump decision tree

```
Since last release:
  ├── Breaking API change? → MAJOR (1.0.0 → 2.0.0)
  ├── New feature? → MINOR (1.0.0 → 1.1.0)
  └── Bug fix only? → PATCH (1.0.0 → 1.0.1)
```

## Changelog format

```markdown
## v2.1.0 — 2026-04-15

### Features
- Add user settings page (#142)

### Bug Fixes
- Fix login timeout on slow connections (#139)

### Breaking Changes
- `getUser()` now returns `User | null` instead of `User | undefined` (#130)

### Internal
- Upgrade dependencies (#141)
```

## Key principles

- Semantic versioning is a contract — follow it strictly
- Every release gets a git tag and a changelog entry
- Release branches exist for stabilization, not development
- Hotfixes branch from the release tag, not from main
- Automate version bumping, changelog generation, and tagging where possible
- Never push a tag that points to a broken commit — CI must pass first

## Anti-patterns I avoid

- Skipping the changelog or writing "various fixes"
- Tagging without a release branch for critical software
- Breaking changes without bumping the major version
- Manual version bumps without updating all package files
- Deploying without a tag/tagging without deploying