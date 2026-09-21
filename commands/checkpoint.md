Invoke the `checkpoint` skill.

The command's argument (`$ARGUMENTS`) is the reason for the checkpoint — one short phrase (e.g., "before postgres migration", "initial build", "auth feature complete").

If `$ARGUMENTS` is empty, use `AskUserQuestion` to elicit the reason. Don't proceed without one — reasonless checkpoints are hard to navigate later.

The skill verifies the working tree is clean on `{{primary_branch}}`, generates a tag `factory/checkpoint/{{DATE}}-{{TIME}}-<slug>`, pushes to origin if a remote exists, and writes a note at `{{factory_root}}/checkpoints/{{DATE}}-<slug>.md`.
