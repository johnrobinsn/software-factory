---
name: feature-cycle
description: One iteration of the feature loop — pick work → cut worktree → delegate implement → delegate review → checkpoint → merge → optionally deploy.
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
---

# feature-cycle

The main loop of the software factory. One feature end-to-end, one worktree.

## When to use

- Operator invoked `/feature "..."` and asked to start immediately.
- Operator picked a backlog item and said "do it."
- `/issue-scan` proposed a GitHub issue and operator said "go."

## Steps

1. **Identify the feature.** Either given by the operator or read from `{{factory_root}}/backlog.md` / a GitHub issue. Confirm the scope in one sentence back to the operator before cutting a worktree.

2. **Check the worktree budget.** If `{{project.dir}}/worktrees/` already has `{{parallel_worktree_limit}}` active worktrees, don't silently queue. Say what's in flight and ask: wait, expand the cap for this session, or reprioritize.

3. **Choose a slug.** For GitHub-issue work: `<issue-number>-<short-title-slug>` (e.g., `42-add-oauth`). For operator-dictated work: ask via `AskUserQuestion` for a short name, or propose one.

4. **Cut the worktree.**
   ```bash
   cd {{project.dir}}/code/<repo>/
   git worktree add ../../worktrees/<slug> -b feature/<slug> {{primary_branch}}
   ```
   If the branch already exists (re-doing a feature), use `git worktree add ../../worktrees/<slug> feature/<slug>` (no `-b`).

5. **Write a brief.** Create `{{project.dir}}/worktrees/<slug>/BRIEF.md` with:
   - Feature title + one-paragraph scope.
   - Link to SPEC.md section(s) this touches, if any.
   - Link to GitHub issue if applicable.
   - Non-goals (what this cycle is NOT doing).
   - Acceptance criteria (how "done" will be judged).

6. **Delegate to `implementer` sub-agent.** Pass the worktree path and BRIEF.md. Instruction: "Implement per BRIEF.md. Follow the project's testing convention from SPEC.md. Commit in logical chunks. Report with commit list, test output, coverage delta, and any spec ambiguities."

7. **Stay available to the operator while implementer runs.** Don't sit and watch the sub-agent's output stream. When it reports done, proceed.

8. **Delegate to `reviewer` sub-agent.** Pass BRIEF.md, worktree path, implementer's report. Instruction: "Second-pair-of-eyes review. Does the implementation match BRIEF.md's acceptance criteria? Are tests real (not tautological)? Coverage on changed lines? Report PASS / CHANGES-REQUESTED / RECOMMENDS-ESCALATE."

9. **Handle review outcome** (same shape as `initial-build`):
   - **PASS** → proceed to step 10.
   - **CHANGES-REQUESTED** → back to implementer with reviewer notes. Loop once. If second review is also CHANGES-REQUESTED, surface to operator.
   - **RECOMMENDS-ESCALATE** → operator call.

10. **Batched status to operator.** One choreographed message: what shipped in the worktree, evidence (test count, coverage), any judgment calls, path to raw logs (`{{project.dir}}/worktrees/<slug>/`), next step options: merge / open PR / checkpoint / deploy-preview / hold.

11. **On operator go, merge.**
    - Local project (no `{{primary_repo}}`): merge feature branch into `{{primary_branch}}` per project git convention, tear down worktree with `git worktree remove ../../worktrees/<slug>`.
    - With `{{primary_repo}}`: open PR via `gh pr create --title "..." --body "$(cat BRIEF.md; echo; echo "## Evidence"; ...)"`. PR body must include evidence, not just assertions. Wait for operator merge (unless they've pre-authorized auto-merge on PASS review).

12. **Optionally deploy.** If BRIEF.md said the feature needs a preview deploy, or operator asks, invoke the `deploy` skill with the appropriate tier.

13. **Log.** Call `log-retro` to append a summary line to `{{factory_root}}/retrospectives/{{DATE}}.md`.

## Escalation triggers within the cycle

Interrupt the operator (don't wait for the batched status) if:
- BRIEF.md and SPEC.md disagree on scope.
- Implementer flagged that the feature needs a schema change, external service, or new dependency.
- Reviewer returned CHANGES-REQUESTED twice — the implementer is stuck.
- The worktree state is dirty in a way that suggests the sub-agent lost context.

## What to skip

- Don't sit inside the worktree editing files yourself. Delegate.
- Don't merge without an operator OK unless they've pre-authorized in this session.
- Don't skip the BRIEF.md — it's what the reviewer checks acceptance against.
