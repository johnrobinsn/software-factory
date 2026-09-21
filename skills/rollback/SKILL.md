---
name: rollback
description: Restore the primary branch to a prior checkpoint tag. Destructive — always confirms with the operator.
user-invocable: true
allowed-tools: Read, Write, Bash, Glob, AskUserQuestion
---

# rollback

Restore `{{primary_branch}}` in `{{project.dir}}/code/<repo>/` to a prior checkpoint. This is destructive — commits after the checkpoint on the primary branch will be lost from HEAD (they still exist in git history, reachable by SHA if needed).

## When to use

- Operator invoked `/rollback [<checkpoint>]`.
- Something broke after a checkpoint and forward-fix is going to take longer than a rollback.

## Steps

1. **Identify the target checkpoint.**
   - If the operator named one on the command line, use it. Verify with `git rev-parse <tag>` that it exists.
   - If not named, list recent checkpoints. Read `{{factory_root}}/checkpoints/*.md` and show the last ~10, newest first, with reason + date. Use `AskUserQuestion` to let the operator pick.

2. **Show the diff summary before doing anything.**
   ```bash
   cd {{project.dir}}/code/<repo>/
   git log --oneline factory/checkpoint/<tag>..{{primary_branch}}
   git diff --stat factory/checkpoint/<tag>..{{primary_branch}}
   ```
   Report: "Rolling back will remove N commits from `{{primary_branch}}`: [list]. Files changed: [summary]. This is destructive. Confirm to proceed."

3. **Wait for explicit operator confirmation.** Not implicit, not "you said yes to something like this earlier" — an in-the-moment "yes, roll back." If they say no, stop.

4. **Check for uncommitted changes and active worktrees.**
   - Uncommitted on `{{primary_branch}}`: surface, offer to stash or abort.
   - Active worktrees on branches cut from the removed commits: surface. Rollback will leave them dangling. Offer: keep them (they'll rebase onto old base later), tear them down, or abort.

5. **Perform the rollback.**
   ```bash
   cd {{project.dir}}/code/<repo>/
   git checkout {{primary_branch}}
   git reset --hard factory/checkpoint/<tag>
   ```
   For a remote-tracked branch that's already pushed to origin, the operator will need to `git push --force-with-lease origin {{primary_branch}}` — do NOT do this yourself. Surface it and let the operator run it.

6. **Write a rollback note.** Create `{{factory_root}}/checkpoints/{{DATE}}-rollback-to-<slug>.md`:
   ```markdown
   # Rollback to <checkpoint reason>

   **Rolled back to:** `factory/checkpoint/<tag>`
   **From SHA:** `<pre-rollback HEAD>`
   **To SHA:** `<post-rollback HEAD>`
   **When:** {{DATE}} {{TIME}}

   ## Why

   <one paragraph: what broke that made rollback the right call over fix-forward>

   ## Removed from primary branch

   <commit list from step 2>

   ## What happened to in-flight work

   <notes about worktrees / open PRs affected>
   ```

7. **Report.** One line summary of what was rolled back + the note path + the `git push --force-with-lease` command if applicable.

## What to skip

- Don't roll back without explicit in-the-moment confirmation. This is the highest-stakes non-deploy op the factory does.
- Don't force-push to remotes yourself. That's an operator-authorized action per-turn.
- Don't automatically tear down active worktrees during rollback. Ask.
