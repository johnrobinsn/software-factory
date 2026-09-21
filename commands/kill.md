Tear down a stuck or unwanted worktree cleanly.

The command's argument (`$ARGUMENTS`) is the worktree slug (matches the directory name under `{{project.dir}}/worktrees/`).

If `$ARGUMENTS` is empty, list active worktrees (as `/worktrees` does) and ask via `AskUserQuestion` which to kill.

Steps:

1. Verify the worktree exists at `{{project.dir}}/worktrees/<slug>/`. If not, list what does exist and stop.

2. Check for uncommitted work: `git -C {{project.dir}}/worktrees/<slug> status --porcelain`. If non-empty, surface — offer to stash to the worktree's branch, commit as WIP, or force-kill (destructive).

3. Check for unpushed commits on the worktree's branch: `git -C {{project.dir}}/worktrees/<slug> log <upstream>..HEAD --oneline` (or vs. `{{primary_branch}}` if no upstream). If unpushed commits exist and the operator is killing anyway, warn — the commits are recoverable by SHA but the branch reference is going away if the operator opts to delete the branch too.

4. Ask: "Kill worktree only (leave branch), or kill worktree AND delete branch?"

5. Kill:
   - `git worktree remove {{project.dir}}/worktrees/<slug>` (add `--force` if operator authorized).
   - If also deleting branch: `git -C {{project.dir}}/code/<repo> branch -D <branch>`.

6. Confirm what was done: worktree removed, branch fate, recovery hint if operator wants to un-do (`git reflog` on the deleted branch's tip SHA within reflog expiry).

This is a destructive operation. Confirmation is required for any force option; uncommitted work is a mandatory prompt.
