Invoke the `deploy` skill.

The command's argument (`$ARGUMENTS`) is the tier name: `dev`, `staging`, `prod`, `preview`, or any custom tier defined in `<project_dir>/deploy.yaml`.

If `$ARGUMENTS` is empty, ask via `AskUserQuestion` which tier — offer the tiers defined in `deploy.yaml` as options.

For `prod` (or any tier with `require_confirmation: true` in `deploy.yaml`), the skill always requires an in-the-moment operator confirmation, regardless of prior authorizations in the session. Prod deploys also auto-tag a `factory/checkpoint/...-before-deploy-prod-...` tag first.

If `deploy.yaml` is missing, the skill will refuse and offer to seed one from `.template/templates/deploy.yaml.example`.
