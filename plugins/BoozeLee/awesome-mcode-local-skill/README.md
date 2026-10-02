# awesome-mcode-local-skill

A bundle of four project skills from the
[awesome-mcode](https://github.com/BoozeLee/awesome-mcode) starter.

| Skill | Trigger |
|---|---|
| `project-onboarding` | "set up this project" |
| `code-review` | "review this" |
| `git-workflow` | "commit this" |
| `daily-standup` | "what did I do" |

## Install

```bash
mcode plugin add BoozeLee/awesome-mcode-local-skill@official
```

(Once this plugin is published. Until then, install from a local clone:
`mcode plugin add /absolute/path/to/awesome-mcode-local-skill@local`.)

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

Source of truth lives at
[github.com/BoozeLee/awesome-mcode/tree/main/plugins/local-skill](https://github.com/BoozeLee/awesome-mcode/tree/main/plugins/local-skill);
this fork mirrors that layout with `plugin.json` at the root so the
registry validator accepts it.
