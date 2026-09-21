---
name: spec-init
description: Iterative interview via AskUserQuestion that builds or refines the project's SPEC.md. Auto-fires when a fresh project is detected without a spec; can also be invoked explicitly.
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# spec-init

Your task is to help the operator build a specification for this project at `{{factory_root}}/SPEC.md`.

Adapted from the operator's global `~/.claude/commands/spec-init.md`, tuned for the software-factory lifecycle.

## When to use

Fire this skill (either explicitly on operator request, or automatically on session start) when:

- The project is fresh: `{{factory_root}}/SPEC.md` does not exist yet.
- The operator explicitly asks to spec, re-spec, or refine the spec.
- A downstream skill (`/initial-build`, `feature-cycle`) discovers that SPEC.md is missing or thin enough that it can't produce a good implementation.

Do NOT fire this skill mid-implementation-cycle unless the operator asks. Interrupting a running cycle to re-spec is a choreography failure.

## Steps

1. **Read what's there.** If `{{factory_root}}/SPEC.md` exists, read it first. Preserve existing sections and refine rather than overwrite. If a starter scaffold exists at `.template/templates/SPEC.md`, use its section headers as the target shape.

2. **Interview iteratively via `AskUserQuestion`.** Don't ask everything in a single wall of text — one probe at a time, driven by what the operator already told you. Push deeper than the surface answer. If a response is vague, redundant, over-engineered, or missing the point, name it.

3. **Cover the software-factory-specific sections.** The starter mold at `templates/SPEC.md` has these headers; make sure each is filled with something concrete, not aspirational:
   - **Purpose** — what problem this solves, for whom, why now.
   - **Users & UX** — who uses it, in what modality (CLI, web, API, TUI), critical flows.
   - **Tech stack** — language, framework, key dependencies, why those specifically over the obvious alternatives.
   - **Testing** — how features get verified without manual clicking. Automated end-to-end path, coverage floor, how the agent inspects state from a shell. For UI apps: headless mode + screencaptures, not "manually check it." (Carry forward the coverage/branch-coverage emphasis from the operator's global spec-init.)
   - **Deploy** — which tiers this project targets (dev, staging, prod, preview), what the deploy command shape looks like, what the health check is. Note that concrete deploy commands live in `<project_dir>/deploy.yaml`, not in SPEC.md.
   - **GitHub integration** — is `{{primary_repo}}` set? Should it be? What label goes on issues that this factory should pick up?
   - **Parallelism & checkpoint cadence** — how much parallel work does the operator want (default is `{{parallel_worktree_limit}}` worktrees)? When should checkpoints fire — every merge, every deploy, milestone-only?
   - **Sub-agent delegation preferences** — is the operator OK with the herder auto-delegating implementer + reviewer without asking each time, or do they want a per-cycle confirmation?
   - **Non-goals** — what this project explicitly does NOT do. This is the section that saves the most time downstream.

4. **Probe deeper than surface answers.** Some question archetypes that tend to surface real constraints:
   - "What's the concrete first user story, end to end?" — forces UX + tech stack + deploy alignment.
   - "How will you know it's working without opening a browser / running it by hand?" — forces testability.
   - "What breaks if this ships wrong?" — forces surfacing risk tolerance, rollback expectations.
   - "What have you built like this before, and what did you regret?" — surfaces preferences that never make it into requirements docs.
   - "If you had to cut 30% of this to ship next week, what goes?" — surfaces the real MVP.

5. **Write incrementally.** After each cluster of answers, update `{{factory_root}}/SPEC.md` and show the delta. Don't wait until the end to persist. If the operator interrupts or context is lost, the partial spec should still be usable.

6. **Confirm before ending.** When you think the spec is complete, read it back at a high level (section titles + one-liner each), ask "anything missing or misframed?" and only end the skill when the operator confirms.

7. **Point at next steps.** End with: "SPEC.md is at `{{factory_root}}/SPEC.md`. Ready for `/initial-build` when you are — or ask me to have the `spec-writer` sub-agent do a critical review pass first."

## What to log

Use `log-decision` (non-invocable) to append a line to `{{factory_root}}/decisions/{{DATE}}-spec-init.md` when the operator makes a notable trade-off during the interview (chose one stack over another, ruled out a feature, set an explicit non-goal). The rationale is usually more valuable than the choice itself.

## What to skip

- Don't invent answers when the operator's response is thin. Ask again.
- Don't ship a fully-templated SPEC.md with placeholder sections. Only sections the operator has actually filled belong in the final document.
- Don't ask permission for every write to SPEC.md — just say "updated" and keep moving. This is a co-thinking flow, not an approval flow.
