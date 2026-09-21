---
name: issue-scan
description: List open GitHub issues labeled with issue_label on primary_repo, propose a priority order via the triager sub-agent, wait for operator OK before dispatching.
user-invocable: true
allowed-tools: Read, Bash, Agent, AskUserQuestion
---

# issue-scan

Conditional on `{{primary_repo}}` being set. If empty, this skill refuses.

## Gate

If `{{primary_repo}}` is empty (the default): "GitHub integration is off for this project. Set `primary_repo` in the project's config.yaml to enable this skill."

If `gh` is not authenticated: check `gh auth status`; if not logged in, tell the operator to run `gh auth login` themselves — do not attempt on their behalf.

## Steps

1. **Fetch labeled open issues.**
   ```bash
   gh -R {{primary_repo}} issue list --label {{issue_label}} --state open --json number,title,labels,body,createdAt,assignees --limit 50
   ```
   If empty: "No open issues labeled `{{issue_label}}` on `{{primary_repo}}`. Nothing to scan."

2. **Filter out issues assigned to humans.** If an issue has a human assignee, the factory shouldn't pick it up unless the operator overrides. Note them in a separate "assigned to humans, skipped" list at the bottom of the report.

3. **Delegate to `triager` sub-agent.** Pass the filtered issue list and the current SPEC.md. Instruction: "Propose priority order + dependency graph. For each issue: estimate size (S/M/L), flag if it depends on another issue in the list, note if the spec is thin enough that spec-init should probably run before implementation, and identify any that look ambiguous or under-specified. Do NOT dispatch — propose only."

4. **Present the triager's proposal.** Format:

   ```markdown
   # Issue scan — {{DATE}}

   Repo: `{{primary_repo}}` | Label: `{{issue_label}}` | Found: N eligible

   ## Recommended order

   1. **#42** [S] Add /health endpoint — no deps, clean spec ✓
   2. **#37** [M] OAuth login — depends on #42's health check for post-login redirect
   3. **#51** [L] Bulk export API — spec is thin, recommend spec-init pass first

   ## Ambiguous / underspecified (recommend re-spec before pickup)

   - **#48** — reproduction steps unclear; needs clarifying comment from reporter

   ## Skipped

   - **#39** — assigned to <username>

   ---

   Say "dispatch <numbers>" to start feature-cycle or bug-cycle on named issues, "top" to dispatch #1, or "hold" to just review this list.
   ```

5. **Wait for operator direction.** Do NOT auto-dispatch. Even the top-priority item goes only on operator go.

6. **On dispatch, for each named issue:**
   - Bug-labeled → invoke `bug-cycle` skill with the issue as input.
   - Everything else → invoke `feature-cycle` skill.
   - Respect the `{{parallel_worktree_limit}}` cap; queue with clear reporting rather than exceeding it.

## Autonomous mode note

This skill is invoked explicitly (either by operator or by an external scheduler like Ring0-cron). It does NOT poll on its own. If the operator wants autonomous behavior — "look for work when idle" — the intended path is: Ring0 (or another scheduler) pings the session with "if idle, run the issue-scan skill and report; if a top-priority item is clean, dispatch." That policy lives outside the template; this skill just provides the mechanism.

## What to skip

- Don't dispatch without an explicit operator go, even on obviously trivial issues.
- Don't rewrite issue bodies. If a body is thin, note it and recommend a re-spec — don't try to fill it in from your own guesses.
- Don't pick up issues without the `{{issue_label}}` label. That's how the operator scopes what the factory sees.
