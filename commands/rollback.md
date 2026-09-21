Invoke the `rollback` skill.

The command's argument (`$ARGUMENTS`) is optional — a checkpoint tag or the slug portion.

If `$ARGUMENTS` is provided, the skill uses it directly. If empty, the skill lists recent checkpoints and asks the operator to pick via `AskUserQuestion`.

The skill always shows the diff summary (commits + files that will be removed from `{{primary_branch}}`) and waits for explicit in-the-moment operator confirmation before performing the reset. This is a destructive operation — the confirmation is not skippable.

If the branch is remote-tracked and already pushed, the operator will need to `git push --force-with-lease origin {{primary_branch}}` themselves — the skill surfaces the command but does not run it.
