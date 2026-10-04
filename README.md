# Dutch Drone Squad GitHub defaults

Shared configuration for repositories in the `dutchdronesquad` organization.

Workflows remain local to each repository. This repository supplies shared configuration that their actions and tools can load; it does not replace repository workflows with reusable callers. Central label synchronization is the agreed exception and remains here.

## Community defaults

Funding and the pull request template live in `.github/`. GitHub uses these for repositories in `dutchdronesquad` when no local override exists. Keep local files for intentional differences.

Issue templates are intentionally not shared: repositories differ too much in the details a useful report needs. Each repository can add its own `.github/ISSUE_TEMPLATE/` when needed.

## Release Drafter

The default lives in `.github/release-drafter.yml`. Release Drafter automatically uses it when a repository in this organization has no local configuration. Keep the Release Drafter workflow in each repository; no `config-name` input is needed.

A repository can override the default with its own `.github/release-drafter.yml`, or extend it to override individual settings:

```yaml
---
_extends: dutchdronesquad/.github
name-template: "service-v$RESOLVED_VERSION"
```

List settings such as `categories` are replaced as a whole when extended. Prefer a complete local list when a repository needs substantially different categories. The shared configuration uses Release Drafter v7.7.0's current syntax.

## Labels

`labels/base.yml` is synchronized by `.github/workflows/sync-labels.yaml`. Pull requests preview changes using the read-only workflow token; merges to `main` apply them. Manual runs are also supported. Extra repository labels are preserved.

The target list covers the organization repositories that use the shared label set. New repositories must be added to the list and token access before removing their local label sync. Archived repositories are excluded.

For synchronization, add the `LABEL_SYNC_TOKEN` Actions secret here: a fine-grained token owned by `dutchdronesquad` with Issues read/write access to every listed target repository. The built-in workflow token cannot update other repositories. Extend token access when adding a target. Previews can read labels from public repositories without that secret.

## Renovate

Repositories retain only a small `.github/renovate.json`, plus any repository-specific overrides. Python projects extend the Python preset and Node.js projects the Node preset:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>dutchdronesquad/.github:renovate-python"]
}
```

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>dutchdronesquad/.github:renovate-node"]
}
```

Other repositories extend `github>dutchdronesquad/.github:renovate-base`. `renovate-base.json` contains common scheduling, dashboard and GitHub Actions policy. `renovate-python.json` extends it with Python dependency rules. `renovate-node.json` extends it with vulnerability alerts and grouped, automerged npm minor and patch updates; repository-specific groups, such as related package families, stay local. `.github/renovate.json` maintains this repository's own action dependencies. CI runs the official Renovate validator.

## Maintenance issues

`scripts/maintenance.py` creates an epic with one sub-issue per affected repository for organization-wide maintenance. See [maintenance issues](docs/maintenance-issues.md).

## Migration

Repositories that still sync labels locally (from the former `.github/labels.yml` raw URL or their own label file) can remove that workflow once the central synchronization has run successfully. Workflows, licenses, tests and project metadata remain local.
