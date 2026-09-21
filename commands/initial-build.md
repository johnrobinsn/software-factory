Invoke the `initial-build` skill.

The skill reads `{{factory_root}}/SPEC.md`, delegates to the `implementer` sub-agent to produce the first working artifact in `{{project.dir}}/code/`, then delegates to the `reviewer` sub-agent. Refuses if SPEC.md is missing (points at the `spec-init` skill) or if `code/` already contains a non-empty repo (unless the operator explicitly authorizes redo).

Ends with a checkpoint proposal on operator go.
