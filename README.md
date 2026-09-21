# software-factory

A [vibr8](https://storminthecastle.com/posts/anatomy_of_a_ring_zero_session/) project template for the software development lifecycle. Spec-driven, agent-orchestrated, worktree-parallel, checkpoint-safe.

The template ships instructions — skills, commands, sub-agents, and a philosophy document — that turn a fresh vibr8 project session into a *herder* that coordinates real coding work across sub-agents rather than doing it itself. The premise, borrowed from the [Software Factories](https://storminthecastle.com/posts/software_factories/) framing: the operator's attention is the scarcest input in software work; the orchestrator's job is to protect it.

## What's in the template

```
software-factory/
├── template.yaml            # variables, layout, defaults
├── AGENTS.md                # herder philosophy + operational rules
├── README.md                # this file
├── skills/
│   ├── spec-init/           # iterative interview → SPEC.md
│   ├── initial-build/       # first working artifact from spec
│   ├── feature-cycle/       # one iteration of the feature loop
│   ├── bug-cycle/           # bug-scoped variant of the cycle
│   ├── checkpoint/          # tag a known-good state
│   ├── rollback/            # restore to a checkpoint
│   ├── status-report/       # single choreographed status
│   ├── issue-scan/          # GitHub issues → priority proposal (conditional)
│   ├── deploy/              # tier deploy via project's deploy.yaml
│   ├── log-decision/        # (non-invocable) append decision note
│   └── log-retro/           # (non-invocable) append retrospective
├── commands/
│   ├── initial-build.md     # /initial-build
│   ├── feature.md           # /feature "<title>"
│   ├── bug.md               # /bug "<title>"
│   ├── checkpoint.md        # /checkpoint "<reason>"
│   ├── rollback.md          # /rollback [<checkpoint>]
│   ├── status.md            # /status
│   ├── issue-scan.md        # /issue-scan
│   ├── deploy.md            # /deploy <tier>
│   ├── retrospective.md     # /retrospective
│   ├── worktrees.md         # /worktrees
│   └── kill.md              # /kill <slug>
├── agents/                  # Claude Code sub-agent definitions
│   ├── spec-writer.md
│   ├── implementer.md
│   ├── reviewer.md
│   ├── deployer.md
│   └── triager.md
├── .opencode/               # OpenCode-specific view
│   ├── agents/              # OpenCode sub-agent definitions (mode: subagent)
│   │   ├── spec-writer.md
│   │   ├── implementer.md
│   │   ├── reviewer.md
│   │   ├── deployer.md
│   │   └── triager.md
│   ├── skills → ../skills   # symlink; portable skills work for both
│   ├── commands → ../commands
│   └── opencode.jsonc       # model pin
└── templates/
    ├── SPEC.md              # bare section-header spec mold
    ├── backlog.md           # empty backlog seed
    └── deploy.yaml.example  # dev/staging/prod/preview deploy config shape
```

Note: no `spec-init` slash command — `spec-init` is a **skill** so it can auto-fire when the model recognizes fresh-project intent, in addition to explicit invocation.

## Registering with vibr8

Once vibr8's template registry knows about this repo:

```
template_register software-factory johnrobinsn/software-factory main
project_create my-thing --template software-factory
```

At `project_create` time, vibr8 scaffolds:

```
<project_dir>/
├── AGENTS.md               # empty project overlay (operator customizes here)
├── skills/ commands/ agents/   # empty project overlays
├── .template/              # rendered template (chmod -R a-w)
├── .claude/ .opencode/     # synthesized backend view symlinks
├── kb/                     # session cwd; briefs, decisions, retrospectives
├── code/                   # your repo(s) live here
├── worktrees/              # parallel work
└── checkpoints/            # rollback notes
```

Open a project session (`create_session`, project=my-thing) and the herder philosophy from `AGENTS.md`, all skills, all commands, and all sub-agents are loaded. Cascade rule: anything you drop into `<project_dir>/skills/`, `commands/`, or `agents/` with the same name silently overrides the template's version. Delete your override → the template's version reappears next session. See the [templates blog post](https://storminthecastle.com/posts/bot_templates/) for the design.

## Template variables

Declared in `template.yaml`. All are optional with sensible defaults; you can override per-project via `project_create --var name=value`.

| Variable | Default | Purpose |
|----------|---------|---------|
| `factory_root` | `{{project.dir}}/kb` | Where the session keeps briefs, retrospectives, checkpoint notes, `SPEC.md`. |
| `primary_repo` | `""` | Optional GitHub `owner/repo`. Empty = no GitHub features. |
| `issue_label` | `factory` | Label the factory picks up. Issues without this label are ignored. |
| `primary_branch` | `main` | Long-lived integration branch. |
| `parallel_worktree_limit` | `3` | Soft cap on concurrent worktrees for parallel work. |

Deploy configuration is intentionally NOT a template variable — it lives in a project-level `<project_dir>/deploy.yaml` (see `templates/deploy.yaml.example`).

## Lifecycle

Fresh project:

1. `project_create my-thing --template software-factory`
2. Open a session in it. `spec-init` skill fires: iterative `AskUserQuestion` interview producing `<project_dir>/kb/SPEC.md`.
3. `/initial-build` — the `implementer` sub-agent lays down the first working artifact in `<project_dir>/code/`.
4. `/checkpoint "initial-build"` — tag the state.

Steady state:

- `/feature "add auth"` or `/bug "login 500 on empty password"` — logs to `backlog.md`, optionally starts a cycle immediately in a worktree.
- Feature/bug cycles delegate to `implementer` → `reviewer` sub-agents in worktrees. The session stays available for operator interruption.
- `/status` — one choreographed status: what's in flight, what's shipped since last report, what needs judgment, what's blocked.
- `/deploy dev` (or `staging`, `prod`) — dispatches per `deploy.yaml`.
- `/retrospective` — end-of-day or end-of-cycle summary.

Optional GitHub-issue-driven mode:

Set `primary_repo` at `project_create` time (or later, by editing the project's `config.yaml`). Label issues `factory` (or your `issue_label` value). Use `/issue-scan` to see eligible work and dispatch. **Autonomous idle-pickup is not shipped in v0.1** — if you want the session to look for work when idle, wire Ring0 (or another scheduler) to prompt the session on a cron with "if idle, run the issue-scan skill and dispatch."

## Backend support

**Both Claude Code and OpenCode are supported.** The template ships parallel agent definitions:

- `agents/*.md` — Claude Code sub-agents (invoked via the `Agent` tool with `subagent_type`).
- `.opencode/agents/*.md` — OpenCode sub-agents (invoked via the `task` tool with `subagent_type`).

Same 5 roles (spec-writer, implementer, reviewer, deployer, triager), same tool restrictions per role. The delegation shape is identical between backends — both tools take `{description, prompt, subagent_type}`. Only the frontmatter differs (Claude uses `tools:` as a comma-string; OpenCode uses `mode: subagent` + `tools:` as a boolean map).

Skills (`skills/`) and commands (`commands/`) work as-is under both backends — no per-backend parallels needed. The `.opencode/skills/` and `.opencode/commands/` entries are relative symlinks pointing at the portable roots, so a single edit updates both views.

vibr8's OpenCode adapter sets `OPENCODE_CONFIG_DIR` to the template's `.opencode/` at session spawn. The template ships `.opencode/opencode.jsonc` with an explicit `model: openai/gpt-5.2-codex` pin (OpenCode's default model resolution can pick a tool-incompatible model otherwise).

## Design references

- [Anatomy of a Ring Zero Session](https://storminthecastle.com/posts/anatomy_of_a_ring_zero_session/) — the framework the template plugs into.
- [Software Factories](https://storminthecastle.com/posts/software_factories/) — the three-axis (bandwidth, attention, choreography) framing that shapes the herder philosophy.
- [Toward installable agent facilities](https://storminthecastle.com/posts/bot_templates/) — how templates + overlays let you customize this while keeping the upgrade path.
- [johnrobinsn/life-system](https://github.com/johnrobinsn/life-system) — sibling template (operator-facing lifelogging), same shape.
