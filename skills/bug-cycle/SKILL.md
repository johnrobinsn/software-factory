---
name: bug-cycle
description: Bug-scoped variant of feature-cycle. Reproduce → cut worktree → delegate fix → regression test → merge.
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
---

# bug-cycle

Same shape as `feature-cycle`, but scoped to a bug and with a mandatory reproduction + regression-test step.

## When to use

- Operator invoked `/bug "..."` and asked to start.
- A backlog item tagged as a bug and operator selected it.
- `/issue-scan` proposed a GitHub bug issue and operator said "go."

## Steps

1. **Capture the report.** If not already in the operator's message, use `AskUserQuestion` to elicit:
   - What did you expect to happen?
   - What actually happened?
   - Reproduction steps (exact commands, inputs, environment).
   - How often does it happen? Every time? Intermittent?
   - When did you first notice? Is there a suspected recent change?

2. **Attempt to reproduce.** Delegate a first pass to the `implementer` sub-agent in a fresh worktree:
   - Cut worktree: `bugfix/<slug>` where slug is `<issue-number>-<short-title>` or hand-picked.
   - Instruction: "Reproduce the bug in this worktree per the reproduction steps in BRIEF.md. If you can reproduce, add a failing test that captures it. If you cannot reproduce, report what you tried and what you observed."

3. **If reproduction fails**, surface to operator with what was tried. Options: try again with different environment, close as "cannot reproduce," escalate. Do not silently claim victory.

4. **Once reproduced with a failing test**, delegate the fix to the same implementer sub-agent. Instruction: "Fix the bug so the failing test passes. Don't remove or weaken the test — it becomes the regression guard. Note any adjacent code that shares the same shape and might have the same bug (report; don't necessarily fix)."

5. **Reviewer pass**, same shape as feature-cycle. Reviewer checks: is the fix at the right level (root cause, not just symptom suppression)? Does the regression test cover the actual bug, not just the surface?

6. **Report to operator with evidence:**
   - Root cause in one sentence.
   - Fix summary + link to the diff.
   - Regression test path.
   - Any adjacent code that shares the shape (reviewer's observation).
   - Test suite output including the new regression test.

7. **Merge and log**, same shape as feature-cycle.

## Escalation triggers

Interrupt the operator immediately if:
- The bug requires a schema migration or data backfill.
- The fix touches security-sensitive code (auth, permissions, secret handling).
- The root cause turns out to be an intentional design decision the operator made — fixing it is a design change, not a bug.
- The bug is a data loss or corruption path.

## What to skip

- Don't skip the reproduction step. "I think I know what it is" without a failing test first is a common way to ship a fix that doesn't fix.
- Don't remove the failing test as part of the fix. It becomes the regression guard.
- Don't fix adjacent bugs opportunistically without operator OK — one worktree per bug keeps rollback clean.
