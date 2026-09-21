# SPEC — <project name>

<!--
This is the software-factory SPEC.md mold. The spec-init skill populates it
via iterative interview with the operator. Every section should end up filled
with something concrete; sections that would have been left blank should be
deleted from the final SPEC.md rather than kept as placeholders.

Delete these comment blocks as you fill each section.
-->

## Purpose

<!--
What problem does this project solve? For whom? Why now?

Concrete over aspirational. If you can't name a specific person or use case,
the spec isn't ready.
-->

## Users & UX

<!--
Who uses this? In what modality — CLI, web, mobile, API-only, TUI?

What are the 1–3 critical flows a user follows? Describe end-to-end from
entry point to outcome, not just "there is an auth screen."
-->

## Tech stack

<!--
Language, framework, key dependencies. For each, one sentence on WHY this
over the obvious alternative. "We use FastAPI because we want async and the
team knows Python" is fine; "We use FastAPI" without the why is not.

Include the deploy target if it constrains the stack (Cloudflare Workers,
Fly.io, self-hosted, Vercel, etc.).
-->

## Testing

<!--
How does the agent verify features are working WITHOUT the operator clicking
around by hand?

- Unit tests? Framework?
- Integration tests? What do they exercise?
- End-to-end automation? How does the agent inspect state from a shell?
- For UI apps: headless mode + screencaptures? Which library?
- Coverage floor on changed lines?
- Branch coverage or line coverage?

This section is load-bearing for the whole factory. Weak here = manual
verification later = operator attention drain.
-->

## Deploy

<!--
Which tiers does this project target? Dev / staging / prod / preview / other?

For each tier, one sentence on what "successfully deployed" means (URL up,
health check passes, background workers running, etc.).

Concrete deploy commands live in `<project_dir>/deploy.yaml`, not in this
document. This section is about intent + tier semantics only.
-->

## GitHub integration

<!--
Is there a primary GitHub repo? (`primary_repo` template variable.)

Do you want the factory to pick up labeled issues autonomously (via
/issue-scan + operator go)? What label? (Default is `factory`.)

If not GitHub-integrated: say so and explain how work items flow instead
(operator dictation, backlog.md, external tracker).
-->

## Parallelism & checkpoint cadence

<!--
How many parallel worktrees do you want the factory to run at once?
(Default is 3 via `parallel_worktree_limit`.)

When should the factory tag a checkpoint?
- After every merge to primary_branch?
- Only at milestones (operator-called)?
- Before every deploy?

Any of these is fine — but make it explicit so the herder knows.
-->

## Sub-agent delegation preferences

<!--
Is it OK for the herder to auto-delegate implementer + reviewer without
asking permission on each cycle? Or do you want a per-cycle "start?"
confirmation?

For deploys, the herder always asks on prod (see AGENTS.md). Other tiers —
auto-deploy after successful review, or always ask?
-->

## Non-goals

<!--
What this project explicitly does NOT do.

This is often the most valuable section in a spec. It's what stops scope
creep from turning "add auth" into "add auth + multi-tenancy + audit log +
role hierarchy" three cycles later.

Bullet points are fine. Aim for 3–8 items.
-->
