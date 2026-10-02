---
name: daily-standup
description: |
  Use when the user says "what did I do", "standup", "yesterday's work",
  "give me the standup", "what did I ship today", or asks the agent to
  produce a status update from git history. Reads `git log` for a date
  window, groups by area (path / topic), and outputs a standup-shaped
  summary. Skip if there is no git repo or no commits in the window.
---

# daily-standup

Turns yesterday's (or N-day) commit history into a standup-shaped summary:
**yesterday** (what shipped), **today** (what's open), **blockers** (what
needs attention).

## When to trigger

- "what did I do", "standup", "yesterday", "give me my update"
- Morning routine; end-of-day wrap-up

## When NOT to trigger

- No git repo (or no `.git`) — report and stop
- No commits in the requested window — say so honestly
- The user wants a project retrospective — that's a different task

## Procedure

1. **Find the window.** Default is the last 24 hours. If the user said
   "this week", use 7 days. Parse `--since=<duration>` if the user gave a
   spec (`2d`, `1w`, `2026-09-30`).

2. **Collect commits.**
   ```bash
   git log --since="<window>" \
           --pretty=format:"%h|%ad|%s|%an" \
           --date=short \
           --no-merges
   ```
   One commit per line: hash, date, subject, author.

3. **Group by area.**
   - First by **scope** inferred from the conventional-commit prefix
     (`feat(scope):`, `fix(scope):`, etc.) if present.
   - Otherwise by top-level dir of the changed files
     (`git log --stat` + path prefix).
   - Aim for 3–6 groups, no more. Combine "fix typo in x" and "fix typo in y"
     into a single "fix: typos" group.

4. **Pick today's open work.** Look at:
   - `git status --porcelain` (uncommitted changes)
   - `git log --oneline --since=<window> --grep='WIP\|WIP-stuff\|DOING'` (or any in-progress tag the team uses)
   - The user can correct — surface them as drafts, not promises.

6. **Compose the standalone writeup.** Output structure:

```markdown
# Standup — <human date>

## Yesterday (<N> commits)

### <Area 1>
- <hash> <subject>
- <hash> <subject>

### <Area 2>
- ...

## Today
- <open item 1>
- <open item 2>

## Blockers / Needs input
- <if any; else open>
```

7. **Print the URL/commit list at the end** so the user can click through.

## Failure handling

- **No commits in window.** Be brief — *"0 commits in the last 24h."*
  Don't invent.
- **Single commit.** Don't group; just list it.
- **Window crosses branches.** Default to `HEAD` only. If they want
  multiple branches, accept `--branch <a>,<b>`.
- **Dirty working tree.** Include a note in **Today** with `git status
  --short` output.
- **Not a git repo.** Stop. *"No git working tree here. Is there a
  different repo you want me to look at?"*

## Acceptance bar

The user copies the **Yesterday** block into Slack/Discord/email without
editing more than two words per line. **Today** lists real open work.
**Blockers** are honest, not invented.