---
name: deploy
description: Dispatch a tier deploy per <project_dir>/deploy.yaml. Reads config, delegates to deployer sub-agent, verifies health, reports URL + rollback command.
user-invocable: true
allowed-tools: Read, Write, Bash, Agent, AskUserQuestion
---

# deploy

Deploys are the highest-stakes non-rollback operation the factory runs. This skill enforces the escalation rules from AGENTS.md.

## When to use

- Operator invoked `/deploy <tier>`.
- Feature-cycle asked to deploy (usually a `preview` on a per-worktree URL).
- Operator asked "deploy this."

## Gate: read deploy.yaml

1. Look for `{{project.dir}}/deploy.yaml`. If missing:
   - Show the operator `.template/templates/deploy.yaml.example` and offer to seed a starter `deploy.yaml` from it via `AskUserQuestion`. Don't just create the file silently — the operator needs to fill in real URLs and commands.
   - Refuse to deploy until `deploy.yaml` exists.

2. Parse `deploy.yaml`. Validate the requested tier is defined. If not, list the available tiers and refuse.

3. Validate the tier's config has at minimum: `deploy_cmd` and `health_check`. If not, refuse with a specific error.

## Prod-deploy escalation

If the tier is `prod` (or any tier where `deploy.yaml` sets `require_confirmation: true`):

**Always require in-the-moment operator confirmation, even if the operator pre-authorized deploys earlier in the session.** Not a stored token, not a "you said yes to prod once" — an explicit "yes deploy to prod" in the same turn as the deploy.

Show a pre-deploy summary before asking for the go:
- Current prod HEAD SHA and how far ahead `{{primary_branch}}` is.
- Recent checkpoints (in case the operator wants a checkpoint before deploy).
- Whether all in-flight worktrees are merged.
- The rollback command per `deploy.yaml`.

## Steps

1. **Pre-flight.** For non-prod tiers, still check:
   - Working tree on the branch being deployed is clean.
   - Tests have passed on this SHA (check CI status via `gh` if `{{primary_repo}}` is set; otherwise run the project's test suite).
   - If either check fails, surface — deploy anyway is an operator-authorized override.

2. **Auto-checkpoint before prod.** For prod deploys, invoke the `checkpoint` skill first with reason "before deploy prod {{DATE}} {{TIME}}". Non-prod tiers skip this by default.

3. **Delegate to `deployer` sub-agent.** Pass:
   - The tier config from `deploy.yaml`.
   - The SHA being deployed.
   - The prod-confirmation token if prod (a per-turn token, not a stored one).
   - Instruction: "Run the deploy_cmd from the config. Capture stdout+stderr. When it exits, run the health_check. Report: deploy exit code, deploy log tail (last ~50 lines), health check result, deployed URL. Refuse if the prod-confirmation token is missing when tier is prod."

4. **Report to operator with evidence:**
   - Tier deployed to.
   - SHA deployed.
   - URL (from `deploy.yaml.<tier>.url` or from the deployer's output).
   - Health check: PASS/FAIL + latency/response summary.
   - Rollback command per `deploy.yaml.<tier>.rollback_cmd` (or a generic one if not specified).
   - Deploy log path.

5. **Failed deploy handling.**
   - Deploy exit non-zero or health check FAIL: do NOT retry silently. Surface immediately. Operator decides: retry, roll back the deploy, roll back the code, debug.
   - Never leave the operator unsure whether prod is up. State it explicitly.

6. **Log.** `log-decision` for prod deploys: append to `{{factory_root}}/decisions/{{DATE}}-deploy-prod.md` with SHA, time, who confirmed, health check.

## Preview deploys

Preview tiers (per-worktree, side-by-side with stable environments) work only if `deploy.yaml` has a `preview:` section with a template URL / deploy_cmd that takes `{{BRANCH}}` or `{{ISSUE_NUMBER}}` as a runtime placeholder.

If asked to preview-deploy from a worktree that has no such section: degrade to dev-tier only and note the gap in the response. Don't invent a preview mechanism.

## What to skip

- Don't deploy without `deploy.yaml`. That's the project owner's contract; the template doesn't guess deploy commands.
- Don't skip the in-the-moment prod confirmation, ever. Even if the operator ran the exact same deploy an hour ago.
- Don't auto-retry a failed deploy. Retrying without diagnosing the failure is how partial-deploy states get worse.
- Don't run destructive rollbacks or force-pushes as part of "deploy" — that's the rollback skill's job, with its own escalation.
