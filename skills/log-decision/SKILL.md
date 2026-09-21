---
name: log-decision
description: Non-invocable helper. Append a decision note to {{factory_root}}/decisions/. Called by other skills mid-run when a judgment call is made.
user-invocable: false
allowed-tools: Read, Write, Edit, Bash
---

# log-decision

Silent, called from other skills. Records a decision + rationale so a future retrospective or spec pass can see WHY, not just WHAT.

## When to use

Called by other skills when:
- The operator makes a trade-off during `spec-init` (chose stack X over Y; ruled out feature Z).
- The implementer had to make a judgment call on a spec ambiguity mid-cycle.
- A deploy escalation happened (who confirmed, when, for what SHA).
- A rollback happened (from what to what, and why).
- A checkpoint captured a milestone worth naming.

Do NOT use for routine "the agent did the thing" logging — that's noise, not decisions. Decisions have alternatives.

## Steps

1. **Compose the filename.** Format: `{{factory_root}}/decisions/{{DATE}}-<category>-<slug>.md`. Category is one of: `spec`, `impl`, `deploy`, `rollback`, `checkpoint`, `arch`. Slug is a short kebab-case tag.

   If a file already exists for the same day+category+slug, append to it rather than overwriting.

2. **Write the entry.** Format:
   ```markdown
   ## {{TIME}} — <one-line decision>

   **Context:** <one sentence — what were we doing when the decision came up>

   **Chose:** <what was chosen>

   **Alternatives considered:** <what else was on the table>

   **Rationale:** <why this over the alternatives — the durable part>

   **Follow-up:** <anything that needs revisiting later, or "none">
   ```

3. **Report back to caller.** Silent (no operator-facing output) — return just the file path to the invoking skill so it can be surfaced if useful.

## Rules

- Silent to the operator. This runs in the background; other skills mention it as part of their own reports.
- No permission prompt for writes. Decisions dir is operator-editable but write-safe.
- Never write empty rationale. "TBD" is better than nothing but a real one-sentence why is what makes decisions logs valuable a month later.
