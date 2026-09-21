---
name: checkpoint
description: Tag the current state of the primary branch as a known-good rollback point; write a note explaining what and why.
user-invocable: true
allowed-tools: Read, Write, Bash, AskUserQuestion
---

# checkpoint

Mark a state that's worth being able to return to. Runs against `{{primary_branch}}` in `{{project.dir}}/code/<repo>/`.

## When to use

- After a successful `/initial-build`.
- Before any hard-to-reverse operation: major refactor, dependency bump, framework migration, schema change, deploy to prod.
- After a milestone feature or bug batch lands and the project is in a known-good state.
- Anytime the operator says "checkpoint this" — trust the instinct.

## Steps

1. **Confirm reason.** Whether from `/checkpoint "<reason>"` or an auto-invoke, make sure you have a one-line reason. If not, ask via `AskUserQuestion`. The reason is what makes the checkpoint list useful later.

2. **Verify clean state.** In `{{project.dir}}/code/<repo>/`:
   - `git status --porcelain` — if not empty, don't checkpoint. Surface: "Uncommitted changes in the working tree. Commit or stash first, or say 'checkpoint anyway' if you know what you're doing."
   - `git rev-parse --abbrev-ref HEAD` — must equal `{{primary_branch}}`. If not: "Currently on branch X, not `{{primary_branch}}`. Checkpoints tag the primary branch. Switch, then re-invoke."

3. **Generate the tag.** Format: `factory/checkpoint/{{DATE}}-{{TIME}}-<slug>` where slug is a slugified version of the reason (lowercase, dashes, max 30 chars). Example: `factory/checkpoint/2026-09-21-1430-before-postgres-migration`.

4. **Tag and push (if remote exists).**
   ```bash
   cd {{project.dir}}/code/<repo>/
   git tag -a factory/checkpoint/<full-tag> -m "<reason>"
   git remote get-url origin >/dev/null 2>&1 && git push origin factory/checkpoint/<full-tag>
   ```

5. **Write the note.** Create `{{factory_root}}/checkpoints/{{DATE}}-<slug>.md`:
   ```markdown
   # Checkpoint: <reason>

   **Tag:** `factory/checkpoint/<full-tag>`
   **SHA:** `<git rev-parse HEAD>`
   **Branch:** `{{primary_branch}}`
   **Created:** {{DATE}} {{TIME}}

   ## Why

   <one paragraph on what state this captures and why it's worth being able to return to>

   ## How to restore

   ```bash
   /rollback <full-tag>
   ```
   or manually:
   ```bash
   cd {{project.dir}}/code/<repo>/
   git checkout {{primary_branch}}
   git reset --hard factory/checkpoint/<full-tag>
   # (destructive; will lose commits after the checkpoint)
   ```

   ## What was in flight

   <list active worktrees at time of checkpoint, if any — they won't be restored>
   ```

6. **Report.** One line: "Checkpoint tagged: `<full-tag>`. Note at `{{factory_root}}/checkpoints/{{DATE}}-<slug>.md`."

## What to skip

- Don't tag if the working tree is dirty. Silent tagging of half-done work is worse than no checkpoint.
- Don't skip the note. The tag alone is opaque; the note is what makes the checkpoint list navigable a week later.
- Don't push the tag to a remote that doesn't exist — check first.
