---
name: git-workflow
description: |
  Use when the user says "commit this", "make a PR", "push this", "branch
  and PR", "ship it", or asks the agent to land the pending work on
  remotely. Stages, writes a Conventional Commit, pushes, and (optionally)
  opens a PR via `gh`. Never pushes to protected branches without explicit
  confirmation.
---

# git-workflow

Lands pending changes safely: stages, writes a Conventional Commit, opens a
PR, and reports the URLs. Avoids accidental force-pushes, accidental pushes
to `main`/`release/*`, and accidental secret commits.

## When to trigger

- "commit this", "commit and push", "make a PR", "ship it", "land this
- After a successful test run if the user said "commit when green

## When NOT to trigger

- The user wants to amend, rebase, or rewrite history → ask first
- The user wants to push to a specific protected branch → confirm first
- There is nothing to commit → report and stop

## Procedure

1. **Check the state.**
   ```bash
   git status --porcelain
   git log --oneline -1
   git branch --show-current
   git remote -v
   ```
   If `status` is empty, report and stop.

2. **Detect secrets.** Run a lightweight scan on the **staged** set:
   ```bash
   git diff --staged | grep -E \
     '(BEGIN (RSA|OPENSSH|EC) PRIVATE KEY|api[_-]?key|secret|password|token)' \
     || true
   ```
   If anything matches, **stop** and surface it. Do not commit secrets.

3. **Stage the right files.**
   - If the user said "commit this" and there are both staged and unstaged
     changes, ask: *"You have unstaged files. Include in this commit?"*
   - Otherwise stage everything (`git add -A`) unless the user listed
     specific paths.

4. **Compose the commit message.** Conventional Commits, ≤72 char subject,
   imperative mood, blank line, body wrapped at 72.
   - Infer the type from the diff (`feat:`, `fix:`, `chore:`, `docs:`,
     `refactor:`, `test:`, `perf:`, `ci:`, `build:`).
   - Infer the scope from the top-level dir or `AGENTS.md`.
   - Subject must stand alone (no period at the end).
   - Body explains **why**, not **what**.
   - Footer for `BREAKING CHANGE:` and issue refs when applicable.

5. **Commit and verify.**
   ```bash
   git commit -m "<subject>" -m "<body>"
   git log --oneline -1
   ```

6. **Push and (optionally) open a PR.**
   ```bash
   # Set upstream if missing
   git push -u origin <branch>
   # Open a PR if gh is available and user asked
   gh pr create --title "<subject>" --body "<pr-body>"
   ```
   Default PR body: summary + test plan + checklist.

7. **Report.**
   ```
   ✓ committed: <sha> <subject>
   ✓ pushed to: origin/<branch>
   ✓ PR opened: <url>
   ```

## Safety rails

- **Refuse to push** to `main`, `master`, `release/*`, `production`, or
  any branch matching `*-prod` unless **ask**. Never force-push to these.
- **Refuse to use `--force`** unless they say `force-with-lease` or
  `force push`.
- **Refuse to amend a pushed commit** unless they say `amend the pushed
  commit` explicitly.
- **Stop on secret detection.** Do not commit; ask the user how to
  proceed (rotate + scrub history, exclude file, etc.).

## Failure handling

- **No remote.** Push step fails. Report the commit hash and ask whether
  to add a remote.
- **`gh` not authenticated.** `gh auth status` — report and offer to
  authenticate (`gh auth login`).
- **Pre-commit hook fails.** Surface the output verbatim. Do not bypass
  with `--no-verify` unless the user says so.
- **Detached HEAD.** Stop and ask the user where to land this.

## Acceptance bar

One command from the user lands the work as a Conventional Commit with
a real PR. Zero accidental force-pushes, zero secret leaks, zero pushes
to protected branches without confirmation.