---
name: initial-build
description: Produce the first working artifact from SPEC.md. Delegates to the implementer sub-agent; lands on the primary branch; tags a checkpoint at the end.
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
---

# initial-build

Take the project from spec to first-working-artifact. Runs once per project (though it can be re-run to redo an early bad start).

## Gate

**Refuse to run if `{{factory_root}}/SPEC.md` does not exist.** Say: "No SPEC.md at `{{factory_root}}/SPEC.md`. Run the spec-init skill first (or ask me to), then come back."

**Refuse if `{{project.dir}}/code/` already contains a non-empty repo** (i.e., initial build already happened). If the operator wants to re-do it, they should either `/rollback` to a checkpoint before the initial build or explicitly say "yes, I know, redo it" — treat that as a destructive-op escalation per AGENTS.md.

## Steps

1. **Read SPEC.md fully.** Note the tech stack, testing convention, and the first user story.

2. **Decide the shape of `code/`.** Single-repo v0.1 assumption — one directory under `code/`. Ask the operator: "What should the repo directory be called? (default: same as project name)" via `AskUserQuestion`.

3. **Delegate to `implementer` sub-agent.** Pass:
   - The full SPEC.md.
   - Working directory: `{{project.dir}}/code/<name>/`.
   - Instruction: "Produce the minimum working artifact that satisfies SPEC.md's first user story end-to-end. Initialize a git repo. First commit is the scaffolding; subsequent commits are the feature. Set up the testing convention SPEC.md specified — including the automated end-to-end path, not just unit tests. Report back with (a) the commit list, (b) how to run the tests, (c) how to run the artifact, (d) any spec ambiguities you had to make judgment calls on."

4. **Wait for the implementer.** While it runs, remain available to the operator. If they interrupt with a question, answer immediately — the sub-agent runs in the background.

5. **Delegate to `reviewer` sub-agent** when the implementer reports done. Pass:
   - The SPEC.md.
   - The `code/<name>/` path.
   - The implementer's report.
   - Instruction: "Second-pair-of-eyes review. Does the artifact actually satisfy SPEC.md's first user story? Do the tests exercise the real path, or just the happy path? Coverage on the changed lines? Report PASS / CHANGES-REQUESTED with specific file:line refs / RECOMMENDS-ESCALATE if there's a spec ambiguity worth surfacing to the operator."

6. **Handle the review outcome:**
   - **PASS** → proceed to step 7.
   - **CHANGES-REQUESTED** → delegate the fix-up back to the implementer with the reviewer's notes. Loop reviewer once. If it goes CHANGES-REQUESTED again, stop looping and surface to operator.
   - **RECOMMENDS-ESCALATE** → surface the ambiguity to operator, wait for a call, then loop.

7. **Report to operator with evidence, not assertions.**
   - Commit list on `{{primary_branch}}`.
   - Test suite output (pass count, coverage on changed lines).
   - How to run the artifact.
   - Any judgment calls the implementer made and what they chose.
   - "Ready to `/checkpoint \"initial-build\"`? Or does something need to change first?"

8. **On operator go, invoke `checkpoint` skill** with reason "initial-build". This tags `factory/checkpoint/{{DATE}}-HHMM-initial-build` and writes a note at `{{factory_root}}/checkpoints/{{DATE}}-initial-build.md`.

## What to log

- `log-decision` for any spec-ambiguity judgment call the implementer made.
- `log-retro` at the end, summarizing: what was in the initial build, what was left for follow-up, any friction points worth remembering next time.

## What to skip

- Don't write the initial-build code yourself. Delegate.
- Don't skip the reviewer pass to save time. Second-pair-of-eyes is the whole point of the review-before-checkpoint pattern.
- Don't tag the checkpoint without operator go. Checkpoints are meaningful precisely because they mark operator-blessed states.
