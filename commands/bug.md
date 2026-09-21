Log a new bug to `{{factory_root}}/backlog.md` and optionally start a bug-cycle on it.

The command's argument (`$ARGUMENTS`) is the bug title / short symptom.

Steps:

1. Use `AskUserQuestion` to capture reproduction context before logging (per bug-cycle skill's step 1). At minimum:
   - What did you expect vs. what happened?
   - Reproduction steps?
   - How often does it happen?

2. Read `{{factory_root}}/backlog.md` (seed from `.template/templates/backlog.md` if absent).

3. Append under the `## Bugs` section:
   ```markdown
   - [ ] <title> — added {{DATE}}
     - Expected: <one line>
     - Actual: <one line>
     - Repro: <one line>
     - Frequency: <every time / intermittent / once>
   ```

4. Ask: "Start a bug-cycle now, or hold for later?"

5. If start-now, invoke the `bug-cycle` skill.

Capture reproduction context BEFORE asking about start-now — the reproduction details are valuable even if the cycle doesn't start immediately.
