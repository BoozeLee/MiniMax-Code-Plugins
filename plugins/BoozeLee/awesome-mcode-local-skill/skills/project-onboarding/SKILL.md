---
name: project-onboarding
description: |
  Use when the user says "set up this project", "onboard this codebase",
  "I just cloned this", "what is this repo", "give me the tour", or asks the
  agent to understand and document an unfamiliar project. Triggers on first
  contact with a directory that has no AGENTS.md or where AGENTS.md is stale.
  Skip if AGENTS.md already exists and is recent.
---

# project-onboarding

One-shot project intake. Reads the repo, identifies build/test/lint commands,
documents entry points, and produces or refreshes `AGENTS.md` so future sessions
have full context from turn one.

## When to trigger

- "set up this project", "onboard this repo", "what is this"
- First session in a directory with no `AGENTS.md`
- A workspace where the user explicitly asks for a refresh

## When NOT to trigger

- The repo already has a recent `AGENTS.md` — point to it instead
- Trivial docs, not actual code (e.g. `index.html` showcase, single-file demo)

## Procedure

1. **Inventory the workspace.** Look for:
   - `README.md`, `LICENSE`, `CONTRIBUTING.md`
   - `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `pom.xml`, `build.gradle*`
   - `Makefile`, `justfile`, `Taskfile.yml`, `scripts/`
   - `Dockerfile`, `docker-compose*`, `*.yaml` manifests
   - CI: `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`
   - `.env.example`, `.env.sample`
   - existing `AGENTS.md`, `CLAUDE.md`, `CODEX.md`

2. **Identify the canonical commands.** For each manifest, record install,
   test, lint, build, deploy. Prefer the simplest form (`make test` /
   `npm test` / `cargo test`) and note one escape hatch for the full
   pipeline (`make ci`).

3. **Map entry points.** Where does the program start? For services, list
   the main process / handler / route registration. For libraries, list the
   public API surface (`src/lib.rs`, `src/index.ts`, etc.).

4. **Read the test setup.** Note the test runner, fixtures, coverage
   tooling. Find one command that runs a single test.

6. **Note the conventions.** Look at `.editorconfig`, `rustfmt.toml`,
   `eslint.config.*`, `biome.json`, `prettier.config.*`. Capture style in
   one line.

7. **Compose `AGENTS.md`.** Use the structure below. If one already exists,
   refresh only the changed sections and surface a diff.

## Output contract

`AGENTS.md` with this shape:

```markdown
# AGENTS.md — <project-name>

## What this is
One paragraph: what the project does, who it's for, what stage it's at.

## Build / test / lint
| Task | Command |
|---|---|
| Install | ... |
| Run tests | ... |
| Run one test | ... |
| Lint | ... |
| Format | ... |
| Dev server | ... |

## Entry points
- `<path>` — <what it is>

## Conventions
- Style: <tool + config>
- Tests: <runner, fixture convention>
- Logging: <format, where>
- Errors: <typed? where?>

## What NOT to do
- <real forbidden ops found in the repo>

## Open questions
- <things only the owner can answer>
```

Print the result and ask: *"I drafted an AGENTS.md. Want me to commit it,
or keep iterating?"*

## Failure handling

- **No manifest files found.** Ask the user what the project is before
  guessing. Do not invent commands.
- **Multiple package managers** (e.g. both `package-lock.json` and
  `pnpm-lock.yaml`). Note the conflict in Open questions; do not pick one.
- **Tests missing.** Note it explicitly — *"Test is empty. I'll treat lint
  + manual smoke as the gate."*
- **Repo is huge** (>10k files). Cap the inventory to top-level + one
  level deep; skip node_modules / target / .git in listings.

## Acceptance bar

A second agent reading only `AGENTS.md` should be able to:
- Run the test suite on a clean checkout.
- Find the main entry point.
- Match the project's style without re-deriving it.

If any of those fails, the AGENTS.md is incomplete.