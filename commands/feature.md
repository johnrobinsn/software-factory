Log a new feature to `{{factory_root}}/backlog.md` and optionally start a feature-cycle on it.

The command's argument (`$ARGUMENTS`) is the feature title / short description.

Steps:

1. Read `{{factory_root}}/backlog.md` if it exists. If not, seed it from `.template/templates/backlog.md`.

2. Append a new entry under the `## Features` section:
   ```markdown
   - [ ] <title> — added {{DATE}}
   ```

3. Ask the operator via `AskUserQuestion`: "Logged to backlog. Start a feature-cycle now, or hold for later?"

4. If start-now, invoke the `feature-cycle` skill with this feature as input. If hold, confirm the backlog was updated and end.

Do not start the cycle silently — always confirm, because starting a cycle spins up a worktree and a sub-agent.
