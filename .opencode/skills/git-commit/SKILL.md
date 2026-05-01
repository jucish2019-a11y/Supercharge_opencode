---
name: git-commit
description: Create well-structured git commits with conventional messages and atomic changes
---

## What I do

I create git commits that are atomic, well-structured, and tell a coherent story in the git log:

- Write conventional commit messages that explain intent, not just what changed
- Split large changes into logical atomic commits
- Stage the right files and nothing else
- Avoid common anti-patterns (mega-commits, meaningless messages, secrets)

## When to use me

Use this skill when:
- Creating a new commit (you should always follow these guidelines)
- Writing a commit message for staged changes
- Deciding whether to split a change into multiple commits
- Reviewing a commit for quality before pushing

## How I work

### Checker mode (reviewing existing commits)

1. **Read the commit message** — Does it explain *why* or just *what*?
2. **Check the diff** — Is the commit atomic (one logical change) or mixed (unrelated changes)?
3. **Check the size** — Is the diff under 400 lines? Larger changes are harder to review and revert.
4. **Check for secrets** — Any API keys, tokens, or credentials in the diff?
5. **Check for generated files** — Any dist/, build/, or lockfile changes that should be separate?

### Applier mode (creating a commit)

1. **Review what changed** — Run `git status` and `git diff`. Understand the full scope.
2. **Decide atomicity** — Should this be one commit or multiple?
3. **Stage selectively** — `git add` only the files for this logical change.
4. **Write the message** — Follow the conventional commit format.
5. **Verify** — `git show` the commit. Does it tell a coherent story?

## Conventional commit format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | When to use | Example |
|------|-------------|---------|
| `feat` | New feature | feat(auth): add SSO login support |
| `fix` | Bug fix | fix(api): handle null response from payment provider |
| `docs` | Documentation only | docs(api): update authentication error codes |
| `style` | Formatting, whitespace, semicolons | style(lint): fix prettier violations in auth module |
| `refactor` | Code restructure without behavior change | refactor(auth): extract token validation into utility |
| `perf` | Performance improvement | perf(list): virtualize 10k+ item rendering |
| `test` | Adding or fixing tests | test(auth): add SSO login integration tests |
| `chore` | Build, tooling, dependencies | chore(deps): upgrade Next.js to 14.2 |
| `ci` | CI/CD configuration | ci(pipeline): add PostgreSQL service container |

### Subject line rules

- Lowercase, no period at the end
- Imperative mood: "add feature" not "added feature" or "adds feature"
- Maximum 72 characters (50 is better)
- No trailing punctuation

### Body rules

- Blank line between subject and body
- Explain *why*, not *what* — the diff shows what changed
- Wrap at 72 characters
- Can be omitted if the subject is self-explanatory

### Footer rules

- Reference issues: `Closes #123` or `Refs #456`
- Note breaking changes: `BREAKING CHANGE: API endpoint /v1/users removed`

### Examples

```
feat(dashboard): add real-time collaboration cursor overlay

Display other users' cursor positions on the canvas with color-coded
labels. Uses WebSocket presence channel for position updates at 60fps.

Closes #234

---

fix(payments): retry failed Stripe webhook deliveries

Stripe webhooks were silently failing on 500 responses from our handler.
Now retries up to 3 times with exponential backoff (1s, 4s, 16s).

Reported-by: @sarah-chen
Refs #189

---

refactor(auth): replace session middleware with JWT validation

The session-based middleware required Redis for session storage, which
added latency to every authenticated request. JWT validation is stateless
and doesn't require Redis, reducing p99 latency by 40ms.

BREAKING CHANGE: /api/auth/session endpoint removed. Use /api/auth/token instead.
```

## Atomic commits

An atomic commit contains one logical change — everything in the diff relates to one purpose.

### How to split a large change

```
Scenario: You've added user registration, validation, and email notification.

Instead of one commit:
  feat: add user registration with validation and email

Split into three:
  1. feat(auth): add user registration endpoint and model
  2. feat(auth): add input validation for registration form
  3. feat(auth): send welcome email after registration

Each commit is independently reviewable and revertable.
If the email breaks, you can revert just that commit.
```

### Selective staging

```bash
# Stage only specific files
git add src/auth/register.ts src/auth/register.test.ts

# Stage parts of a file (interactive)
git add -p src/auth/index.ts

# Stage everything in a directory
git add src/auth/

# Check what you've staged (before committing)
git diff --cached
```

### When commits are NOT atomic (and that's OK)

- Version bumps: `chore(release): v2.3.0` — includes version bump + changelog update
- Dependency upgrades: `chore(deps): upgrade React to 18.3` — includes all files changed by the upgrade
- Large refactors with a single purpose: `refactor(api): migrate REST to tRPC` — many files changed but one intent

## What NOT to commit

```
NEVER commit:
  - .env files (use .env.example with placeholder values instead)
  - API keys, tokens, passwords, or secrets of any kind
  - Large binary files (images, videos, PDFs over 1MB)
  - Generated files that can be recreated (dist/, build/, .next/)
  - IDE configuration (.idea/, .vscode/ unless it's a team-shared config)
  - OS-specific files (.DS_Store, Thumbs.db)

IF YOU ACCIDENTALLY COMMIT A SECRET:
  1. Immediately rotate the compromised credential
  2. Remove from git history with git filter-branch or BFG Repo-Cleaner
  3. Force-push (coordinate with team first)
  4. Do NOT just delete the file in a new commit — it's still in history
```

## Commit message checklist

- [ ] Uses conventional commit format (type + scope + subject)
- [ ] Subject is under 72 characters, lowercase, imperative mood
- [ ] Body explains *why*, not *what*
- [ ] Commit is atomic (one logical change)
- [ ] No secrets, credentials, or API keys in the diff
- [ ] No unrelated changes mixed in
- [ ] References relevant issue numbers
- [ ] Breaking changes noted in footer

## Anti-patterns I avoid

- `fix: fix bug` — says nothing about which bug or what the fix was
- `wip` or `todo: clean up` — if it's not done, don't commit it (use a stash or draft PR)
- `update index.js` — which index.js? What changed?
- Mixed concerns in one commit (feature + refactor + dependency upgrade) — impossible to review or revert selectively
- Committing generated files that can be recreated from source
- Committing with `git add .` without reviewing what's staged
- Writing commit messages in past tense ("added feature") instead of imperative ("add feature")
- Omitting the body when the subject alone doesn't explain the change