---
name: git-commit
description: Create well-structured git commits with conventional messages and atomic changes
---

## What I do

I create clean, meaningful git commits:

- Stage the right files — no unrelated changes mixed in
- Write conventional commit messages that explain why, not what
- Ensure each commit is atomic and independently revertable
- Verify the commit succeeds and the repo is clean

## When to use me

Use this skill when:
- The user asks to commit changes
- After completing a logical unit of work
- Before pushing to remote
- When creating a PR requires clean history

## How I work

1. **Review what changed** — Run `git status` and `git diff` to see all staged and unstaged changes.
2. **Group logically** — Separate unrelated changes into different commits. Each commit should represent one logical change.
3. **Stage selectively** — Use `git add` for specific files or hunks. Never `git add .` blindly.
4. **Write the message** — Use conventional commit format:
   ```
   type(scope): subject

   body (optional)
   ```
   Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `style`, `perf`, `build`, `ci`
5. **Verify** — Run `git status` after committing to confirm clean state.

## Message guidelines

- Subject line: 50 chars or less, imperative mood ("add" not "added")
- Body: explain why the change was made, not what was changed
- Reference issue numbers when applicable
- No emojis unless the project convention uses them

## Safety

- Never commit files with secrets, credentials, or .env files
- Never use --no-verify
- Never amend commits unless explicitly asked
- Never force push
- If pre-commit hooks fail, fix the issue and create a new commit