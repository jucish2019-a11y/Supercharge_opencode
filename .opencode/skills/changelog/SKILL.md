---
name: changelog
description: Generate structured changelogs from git history, commits, and pull requests
---

## What I do

I generate structured changelogs:

- Parse git history, commits, and PRs to extract meaningful changes
- Categorize changes by type (features, fixes, breaking changes, etc.)
- Write clear, user-facing change descriptions
- Follow existing changelog format and conventions

## When to use me

Use this skill when:
- Preparing a release and needing a changelog
- Summarizing changes since the last release
- Creating a CHANGELOG.md or release notes
- The user asks what changed since a version or commit

## How I work

1. **Determine the range** — Find the last release tag or commit to compare against. Use `git log <last-tag>..HEAD` to get all commits since then.
2. **Categorize changes** — Group commits by type:
   - **Features**: New capabilities (`feat:`, `add:`)
   - **Fixes**: Bug fixes (`fix:`, `patch:`)
   - **Breaking changes**: Incompatible API changes (`BREAKING CHANGE`, `!:`)
   - **Performance**: Performance improvements (`perf:`)
   - **Refactor**: Code restructuring with no behavior change (`refactor:`)
   - **Docs**: Documentation changes (`docs:`)
   - **Chores**: Build, CI, dependency updates (`chore:`, `build:`, `ci:`)
3. **Write user-facing descriptions** — Rewrite commit messages into descriptions that users care about. Focus on what changed and why, not how.
4. **Follow project format** — Match existing changelog style: Keep a Changelog, GitHub Releases, conventional changelog, etc.
5. **Include upgrade notes** — For breaking changes, include migration instructions.

## Output format (Keep a Changelog style)

```markdown
## [version] - YYYY-MM-DD

### Breaking Changes
- Changed `oldApi` to `newApi` — update callers to use `newApi`

### Features
- Added support for `feature`
- Now handles `scenario` correctly

### Fixes
- Fixed crash when `condition` occurred
- Fixed incorrect calculation in `function`

### Performance
- Reduced memory usage by X% in `module`

### Dependencies
- Upgraded `package` from X to Y
```

## Guidelines

- Write for users, not developers — describe impact, not implementation
- Group by significance: breaking changes first, then features, then fixes
- Include upgrade notes for any breaking change
- Link to relevant PRs or issues when available
- Don't include internal refactors, test additions, or chore changes unless they affect users
- Use consistent verb tense (past tense or imperative — pick one and stick with it)