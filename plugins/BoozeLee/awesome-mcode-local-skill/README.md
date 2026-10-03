# awesome-mcode-local-skill

Four project skills from the
[awesome-mcode](https://github.com/BoozeLee/awesome-mcode) starter, packaged so any
MiniMax Code install can use them.

| Skill | Trigger | Output |
|---|---|---|
| `project-onboarding` | "set up this project", "onboard this codebase" | AGENTS.md + entry-point map |
| `code-review` | "review this", "is this safe to merge" | Severity-tagged review of the pending diff |
| `git-workflow` | "commit this", "make a PR" | Conventional Commit, push, optional `gh pr create` |
| `daily-standup` | "what did I do", "standup" | Yesterday / today / blockers from `git log` |

Each Skill is a Markdown file with frontmatter that states what it does and when it should
activate, so MiniMax Code can pick the right one from the request alone.

## Try it

```text
Onboard this repository: read AGENTS.md, then map the entry points and give me a tour.
```

Expected result: a written orientation of the project — what it is, where the code starts,
which commands build and test it, and any entry-point map worth keeping. If the repo already
has a current `AGENTS.md`, the Skill says so instead of regenerating one.

```text
Review my pending changes against AGENTS.md and tell me if this is safe to merge.
```

Expected result: a structured review of the working-tree diff, each finding tagged by
severity, with file and line references, and an explicit verdict.

## Requirements

- MiniMax Code 0.3 or newer.
- `git` on `PATH`. `git-workflow` and `daily-standup` read history and stage commits; they
  will report that the directory is not a git repository rather than failing.
- `gh` on `PATH` — **only** if you want `git-workflow` to open a pull request. Without it the
  Skill stops after the local commit and tells you the exact command to run yourself.
- No accounts, no paid services, no API keys.

Supported platforms: any platform with a POSIX shell and `git`. The Skills themselves are
platform-neutral Markdown.

## Data and network

- **Network access: none.** The Plugin ships no MCP server and makes no requests. It reads
  and writes only the files in the repository you point it at.
- **Credentials: none required.** It never reads or writes your MiniMax configuration and
  never reads tokens.
- **Data handled:** local repository contents and `git` history. `daily-standup` reads commit
  metadata for a date window; `code-review` reads the pending diff; `git-workflow` stages and
  commits the changes you asked it to land, and pushes only after you confirm.
- **Destructive actions are gated.** `git-workflow` will not push to a protected branch or
  force-push without explicit confirmation. Nothing here deletes files.

## Install

This Plugin is not published to the `official` marketplace yet, so install it from a clone
into the `local` marketplace, which is a plain directory:

```bash
git clone --branch feat/awesome-mcode-local-skill --single-branch \
  https://github.com/BoozeLee/MiniMax-Code-Plugins.git
cp -r MiniMax-Code-Plugins/plugins/BoozeLee/awesome-mcode-local-skill ~/.minimax/plugins/
mcode plugin add awesome-mcode-local-skill@local
mcode plugin list -m local
```

The `local` marketplace is `~/.minimax/plugins/`. The Plugin root must be a **physical
directory** — MiniMax Code opens plugin roots with `rejectSymlink: true` and rejects a
symlinked one with `PLUGIN_ROOT_SYMLINK`, which drops the Plugin from `mcode plugin list -m
local` without a visible error. `cp -r`, not `ln -s`.

If you already use the [awesome-mcode](https://github.com/BoozeLee/awesome-mcode) starter, it
installs this Plugin for you with `./scripts/sync-skills.sh --install`.

## Layout

```
awesome-mcode-local-skill/
├── plugin.json
├── README.md
├── LICENSE
└── skills/
    ├── project-onboarding/SKILL.md
    ├── code-review/SKILL.md
    ├── git-workflow/SKILL.md
    └── daily-standup/SKILL.md
```

`plugin.json` sits at the plugin root because that is the path the community registry
validator reads. It deliberately carries no `skills`, `mcpServers`, or `apps` array: the
validator's allowed-field list rejects those as unknown fields, and it discovers Skills from
the filesystem instead. The Skill directory names match their frontmatter `name` fields,
which the validator requires.

Source of truth for the Skills is
[github.com/BoozeLee/awesome-mcode/tree/main/skills](https://github.com/BoozeLee/awesome-mcode/tree/main/skills);
this package mirrors them.

## License

MIT — see [LICENSE](LICENSE).
