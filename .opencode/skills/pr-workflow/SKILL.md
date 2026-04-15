---
name: pr-workflow
description: Manage pull request workflows — creation, review process, CI integration, merge strategies, and automation
---

## What I do

I manage the full PR lifecycle:

- **PR creation** — Title, description, linked issues, reviewers, labels
- **Review process** — Code review standards, review checklists, approval requirements
- **CI integration** — Required status checks, test results, automatic merging
- **Merge strategies** — Squash, merge commit, or rebase based on context
- **Automation** — Auto-assign reviewers, label PRs, merge queues, branch cleanup

## When to use me

Use this skill when:
- Creating a new pull request
- Setting up PR templates and review processes for a team
- Configuring branch protection and required checks
- Managing merge queues or batch merges
- Automating PR workflows with GitHub Actions or similar

## How I work

1. **Check PR readiness** — All tests passing? Lint clean? Type checks pass? No conflicts with base branch?
2. **Create the PR** — Use the `gh` CLI for PR creation with proper title, description, and metadata.
3. **Write a quality description**:
   - **What** — What does this PR do?
   - **Why** — Why is this change needed? Link the issue.
   - **How** — Key implementation decisions worth reviewing.
   - **Testing** — How was this tested? What edge cases were considered?
   - **Screenshots** — For UI changes, before/after screenshots.
4. **Request reviewers** — 1-2 required reviewers for most changes. More for critical paths.
5. **Address review feedback** — Respond to every comment. Push fixes. Mark conversations as resolved.
6. **Merge** — After approval and CI passes, merge using the project's merge strategy. Delete the branch.

## PR title format

Conventional commits style:
```
type(scope): description

feat(auth): add OAuth2 login flow
fix(api): handle timeout on slow connections
docs: update API reference for v2
refactor(db): extract connection pooling
test(auth): add edge cases for token refresh
chore(deps): upgrade React to 19.1
```

## PR description template

```markdown
## What
[1-2 sentences describing the change]

## Why
[Link to issue or explanation of motivation]

## How
[Key implementation decisions]

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing performed: [describe]
- [ ] Edge cases considered: [list]

## Screenshots (if applicable)
[Before/After]
```

## Review checklist

- [ ] Does the PR do what it says?
- [ ] Are there any obvious bugs?
- [ ] Are edge cases handled?
- [ ] Is the code readable and well-structured?
- [ ] Are there tests for the new behavior?
- [ ] Are there any security concerns?
- [ ] Does this break any existing APIs?
- [ ] Is the PR scoped to one concern?

## Merge strategy guide

| Branch type | Merge method | Reason |
|-------------|-------------|--------|
| Feature → main | Squash merge | Clean history, one commit per feature |
| Release → main | Merge commit | Preserve release metadata |
| Hotfix → main | Merge commit | Preserve hotfix record |
| Dependent PRs | Rebase and merge | Linear history for bisect |

## Key principles

- Every change to main goes through a PR — no exceptions
- PRs should be small (<400 lines changed) for faster, better reviews
- CI must pass before merge — no bypassing
- One PR = one concern — don't mix features and refactorings
- Review your own PR before requesting reviewers
- Respond to all review comments, even if just "done"

## Anti-patterns I avoid

- PRs over 400 lines without explanation — break them up
- Merging without at least one approval
- "Looks good to me" reviews without actually reading the code
- Draft PRs that sit open for weeks — use feature flags instead
- Force-pushing after review has started (invalidates the review)