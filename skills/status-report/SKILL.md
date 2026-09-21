---
name: status-report
description: Produce a single choreographed status — what's in flight, what shipped since last report, what needs judgment, what's blocked.
user-invocable: true
allowed-tools: Read, Bash, Glob, Grep
---

# status-report

The status summary is the primary vehicle for the "batch, don't stream" principle. One message the operator can read in under a minute that answers "what's going on with this project?"

## When to use

- Operator invoked `/status`.
- End of a multi-cycle session, before disengaging.
- After a period of autonomous work (e.g., overnight batch, if using Ring0-cron autonomy).

## Steps

1. **Gather the state.** In parallel where possible:
   - **Active worktrees:** `ls {{project.dir}}/worktrees/` — for each, read its `BRIEF.md` (if present) and check `git -C <worktree> log --oneline -5` and `git -C <worktree> status --porcelain` for progress signal.
   - **Recent primary-branch activity:** `git -C {{project.dir}}/code/<repo> log --oneline --since="<last status or 24h>" {{primary_branch}}` — commits since last report.
   - **Recent checkpoints:** `ls -t {{factory_root}}/checkpoints/*.md | head -5`.
   - **Open PRs (if `{{primary_repo}}` set):** `gh -R {{primary_repo}} pr list --state open --json number,title,statusCheckRollup,reviewDecision`.
   - **Recent retrospectives:** `ls -t {{factory_root}}/retrospectives/*.md | head -3`.
   - **Backlog top:** `head -30 {{factory_root}}/backlog.md` (if present).

2. **Categorize into four buckets:**
   - **In flight** — worktrees with an active sub-agent or awaiting review.
   - **Shipped since last status** — merges to `{{primary_branch}}`, deploys, closed issues.
   - **Needs judgment** — PRs awaiting operator review, reviewer CHANGES-REQUESTED-twice loops, ambiguities the implementer flagged, escalation triggers from any cycle.
   - **Blocked** — waiting on external service, waiting on operator answer that hasn't come, worktrees stuck without progress.

3. **Compose the status.** Format:

   ```markdown
   # Status — {{DATE}} {{TIME}}

   ## In flight ({{count}})
   - **worktrees/<slug>** — <one-line status>. Age: <since>. Owner: <sub-agent>.
   - ...

   ## Shipped since last status ({{count}})
   - <SHA> — <commit message> (<worktree slug if applicable>)
   - ...

   ## Needs your judgment ({{count}})
   - **<what>** — <one-line context> — <one-line ask>
   - ...

   ## Blocked ({{count}})
   - **<what>** — <what it's waiting on>
   - ...

   ## Recent checkpoints
   - <tag> — <reason>
   - ...

   ## Backlog top ({{n}} items)
   - <first three backlog items>

   ---

   Raw logs: `{{project.dir}}/worktrees/`. Full backlog: `{{factory_root}}/backlog.md`. Full retros: `{{factory_root}}/retrospectives/`.
   ```

4. **Present.** Don't wrap it in a preamble ("Here's the status you asked for") — the operator asked for a status; deliver it.

## Choreography rules

- **Ordering.** Needs-judgment first if non-empty; the operator's time is most valuable on that bucket. Otherwise in-flight → shipped → blocked → recent checkpoints → backlog.
- **Compression.** Each item is one line. If a worktree has fifteen commits of interesting activity, note "15 commits since last status; recent: [top 3]"; the operator drills in if they care.
- **Evidence, not assertions.** For "shipped": include the test-pass signal or PR check status. For "in flight": include how long the sub-agent has been running.
- **Uncertainty is a signal.** If a worktree hasn't moved in >30 minutes and the sub-agent hasn't reported, mark it "stalled" in the in-flight list — that's a needs-judgment prompt.

## What to skip

- Don't include worktrees that are cleaned up. Only active ones.
- Don't list every commit in "shipped" if there are dozens — collapse ("12 commits, mostly test fixes; feature commits: [top 3]").
- Don't fabricate a "needs judgment" bucket if there's nothing there. An empty bucket is a valid answer — the operator gets that back in one line ("nothing awaiting you").
