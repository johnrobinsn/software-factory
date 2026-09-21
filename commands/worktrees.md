List active worktrees under `{{project.dir}}/worktrees/`.

For each:

- **Slug** (directory name).
- **Branch** (from `git -C <worktree> rev-parse --abbrev-ref HEAD`).
- **Assigned sub-agent** (from the worktree's BRIEF.md `Owner:` line, if present).
- **Current state** — one of: implementing / reviewing / awaiting-op / stalled. Derive from BRIEF.md status field, or from git activity signal (recent commits = implementing, no commits + no changes for >30 min = stalled).
- **Age** — how long since the worktree was created (`stat -c %Y <dir>` or equivalent).
- **Uncommitted files** — count from `git -C <worktree> status --porcelain | wc -l`.

Present as a compact table. If no worktrees are active, say so plainly ("No active worktrees. `/feature "..."` or `/bug "..."` to start one.").

Include the current worktree budget: `<active> / {{parallel_worktree_limit}}` — helps the operator see how close they are to the cap.
