---
name: branching-strategy
description: Design and implement Git branching models for teams — trunk-based, Git Flow, GitHub Flow, and hybrids
---

## What I do

I design Git branching strategies that match the team and project:

- **Strategy selection** — Choose the right branching model based on team size, release cadence, and deployment needs
- **Branch naming** — Consistent naming conventions for feature, fix, release, and hotfix branches
- **Merge policies** — Squash merge, merge commit, or rebase — when to use each
- **Protection rules** — Branch protection, required reviews, CI checks

## When to use me

Use this skill when:
- Setting up a new repository's branching model
- Changing an existing team's branching workflow
- Resolving merge conflicts that stem from poor branching strategy
- Onboarding developers to a branching convention

## How I work

1. **Assess the context** — Team size, release frequency, deployment automation level, regulatory/audit needs.
2. **Select a model**:
   - **Trunk-based** (recommended default) — Short-lived branches (<1 day), continuous integration to main, feature flags for incomplete work. Best for teams with CI/CD.
   - **GitHub Flow** — Simple: main + feature branches. Deploy from main. Good for small teams, web apps with continuous deployment.
   - **Git Flow** — main + develop + feature + release + hotfix branches. Good for versioned software with scheduled releases, but complex for most teams.
   - **Hybrid** — Trunk-based with release branches for stabilization. Good when you need to support multiple versions.
3. **Define branch rules** — Naming, lifetime limits, required CI checks, required reviewers.
4. **Define merge strategy** — Squash merge for feature branches (clean history), merge commit for release branches (preserve metadata).
5. **Document** — Write a CONTRIBUTING.md or branch strategy doc with examples.

## Branch naming conventions

```
feature/JIRA-123-add-user-settings
fix/JIRA-456-login-timeout
hotfix/JIRA-789-production-crash
release/v2.1.0
```

## Rules of thumb

| Team size | Strategy | Branch lifetime | Merge method |
|-----------|----------|---------------|--------------|
| 1-5 | Trunk-based | <1 day | Squash |
| 6-15 | Trunk-based | <2 days | Squash |
| 15+ | Hybrid | <3 days | Squash |
| Versioned releases | Git Flow or Hybrid | Sprint-length | Merge commit for releases |

## Key principles

- Main/master must always be deployable
- Branches should be short-lived — if a branch lives >2 days, break the feature into smaller pieces
- Feature flags over long-lived branches
- CI must pass on every branch before merge
- Delete branches after merge — stale branches cause confusion
- Rebase local work onto main before opening a PR (not after review)

## Anti-patterns I avoid

- Long-lived feature branches (>3 days without merging)
- Merging without CI passing
- Direct commits to main/master without review
- Multiple merge strategies without clear rules
- Not deleting branches after merge
- Merge hell: too many long-lived branches diverging from main