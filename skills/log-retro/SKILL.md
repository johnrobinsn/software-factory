---
name: log-retro
description: Non-invocable helper. Append a retrospective line to {{factory_root}}/retrospectives/. Called at the end of cycles or explicitly via /retrospective.
user-invocable: false
allowed-tools: Read, Write, Edit, Bash
---

# log-retro

Records what happened in a cycle (or a day) in a form that feeds the next planning pass. Retros are the raw material for the operator's own reflection AND for improving the factory's defaults.

## When to use

Called by other skills at cycle end:
- `initial-build` completes.
- `feature-cycle` completes (either merge or PR-open).
- `bug-cycle` completes.
- Explicit `/retrospective` invocation.

## Steps

1. **Compose the filename.** `{{factory_root}}/retrospectives/{{DATE}}.md` — one file per day. Append to it rather than creating a new file if one exists for today.

2. **Append entry.** Format for cycle completions:
   ```markdown
   ## {{TIME}} — <feature|bug|initial-build|manual>: <one-line title>

   **What shipped:** <specific — e.g., "OAuth via GitHub, tests + coverage 91%, deployed to staging">

   **Cycle stats:**
   - Worktree: `<slug>` (torn down / kept)
   - Sub-agents: implementer + reviewer (+ deployer if applicable)
   - Reviewer rounds: <1|2|3+>
   - Ambiguities surfaced to operator: <count>
   - Duration: <wall clock from cycle start to close>

   **What was surprising:** <anything worth remembering — a bug that turned out to be adjacent, an implementer choice that surprised the operator, a dep addition, etc.>

   **What we'd do differently:** <one sentence — could be "nothing"; be honest>
   ```

3. **For explicit `/retrospective` invocations (no cycle context)**: read the day's retro file (if any), the day's commits on `{{primary_branch}}`, the day's decision log entries, and produce an end-of-day summary appended to the same file:
   ```markdown
   ## {{TIME}} — end of day

   **Cycles completed:** <count>
   **Checkpoints tagged:** <count>
   **Deploys:** <list by tier>
   **Ambiguities in the day:** <patterns worth naming for next spec pass>
   **Backlog change:** <items added, items closed, net change>
   ```

## Rules

- Silent to the operator on cycle-end auto-invokes. Other skills mention it as part of their close-out report.
- Explicit `/retrospective` invocations DO produce operator-facing output — read the file back after appending so the operator sees the summary.
- Never fabricate stats. If duration is unknown, say "unknown." If reviewer rounds are unclear, say so.
- Retro files stay short. If today's file is over ~200 lines, it's probably a symptom of something worth surfacing in the day's end-of-day entry.
