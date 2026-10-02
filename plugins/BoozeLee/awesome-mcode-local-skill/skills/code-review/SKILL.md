---
name: code-review
description: |
  Use when the user says "review this", "review my changes", "look at this
  PR", "is this safe to merge", or asks the agent to evaluate pending code
  before committing. Reads the diff, checks it against AGENTS.md, runs
  relevant tests, and produces a structured review with severity-tagged
  findings. Do NOT use for wholesale refactor suggestions — those are a
  separate task.
---

# code-review

Reviews uncommitted / unstaged changes or a pull request. Surfaces findings
in priority order so the user can decide what to fix before commit or merge.

## When to trigger

- "review this", "review my changes", "PR review"
- Before running `git commit` if the user asked for a sanity pass

## When NOT to trigger

- The change is one or two lines of trivial cleanup
- The user explicitly says they want a refactor proposal (different task)
- There are no changes (`git diff` is empty) — say so and stop

## Procedure

1. **Establish the diff.**
   - Default: `git diff` (unstaged) + `git diff --staged` (staged).
   - For a PR: `gh pr diff <number>` or `git diff main...HEAD`.
   - If the diff is empty, stop and report.

2. **Read the project rules.** Open `AGENTS.md` (or `CLAUDE.md` /
   `CONTRIBUTING.md` in that order of preference). Capture: style tool,
   test gate, error style, forbidden ops.

3. **Run the gate.** Execute the project's lint + format + test commands
   from AGENTS.md. Capture pass/fail and any output.

4. **Read the diff carefully.** For each file, look for:
   - **Correctness.** Off-by-one, null/undefined, race conditions, error
     swallowing, partial writes, type mismatches.
   - **Security.** Untrusted input → SQL/Command, secrets in code,
     missing auth, path traversal, SSRF, prototype pollution.
   - **Tests.** New code without a test; new branch without a case;
     flaky-looking setup.
   - **Style.** Imports, naming, formatting, dead code.
   - **API surface.** Public exports changed without a doc update.
   - **Performance.** O(n²) where O(n) is obvious, sync I/O in async
     paths, missing indexes.

5. **Tag each finding.** Use:
   - 🔴 **blocker** — must fix before merge (bug, security, data loss)
   - 🟠 **major** — should fix before merge (perf, correctness, tests)
   - 🟡 **minor** — nit, style, optional cleanup
   - 💬 **question** — needs author clarification

6. **Produce the review.** Output structure:

```markdown
# Review: <branch / commit range>

**Gate:** ✅ / ❌ / ⚠️ partial  (<commands run>)
**Files:** <count>  **+/-:** <insertions>/<deletions>

## Findings

### 🔴 blockers
- `<file>:<line>` — <issue> [suggested change]

### 🟠 majors
- ...

### 🟡 minors
- ...

### 💬 questions
- ...

## Summary
<one paragraph: ship it, fix-and-ship, or needs work>
```

7. **Hand off.** Ask: *"Want me to address the blockers, or leave them
for the author?"*

## Failure handling

- **Diff is empty.** Report and stop.
- **Gate fails.** Still produce the review but mark the gate ❌. Do not
  attempt to fix the gate output automatically — surface it.
- **AGENTS.md missing.** Fall back to defaults: lint = `npm run lint`
  or repo equivalent; test = `npm test`. Note in the review that rules
  were inferred.
- **Reviewing a PR with >50 files.** Decline to "broad review" mode:
  group by directory, only surface cross-cutting issues, and ask the
  user to point at the risky areas first.

## Acceptance bar

Every blocker has a one-line fix proposal. Every major has either a fix
or a question for the author. No "you might want to consider" mush.