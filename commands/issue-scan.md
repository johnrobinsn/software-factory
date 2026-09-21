Invoke the `issue-scan` skill.

Conditional on `{{primary_repo}}` being set. If it isn't, the skill refuses and tells the operator how to opt in.

The skill fetches open GitHub issues labeled `{{issue_label}}` on `{{primary_repo}}`, delegates the priority-order proposal to the `triager` sub-agent, and presents the ranking. It does NOT dispatch — the operator picks what to work on from the proposal.
