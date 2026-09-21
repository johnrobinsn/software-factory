Force-invoke the `log-retro` skill in explicit-invocation mode (produces operator-facing summary, not a silent append).

Reads today's retro file, the day's `{{primary_branch}}` commits, the day's decision-log entries, and any checkpoints tagged today. Produces an end-of-day summary appended to `{{factory_root}}/retrospectives/{{DATE}}.md`, then reads the summary back to the operator.

Use this at end-of-day, end-of-session, or any time the operator wants a "what happened today" recap.
