# Dutch Drone Squad GitHub defaults

This repository contains shared GitHub configuration for the `DutchDroneSquad` organization.

## Labels

The canonical organization label set lives in `.github/labels.yml`. It defines the shared label names, colors, and descriptions used by Dutch Drone Squad repositories. Labels referenced by Release Drafter, such as `breaking-change`, `new-feature`, `bugfix`, and `dependencies`, must also be defined in this manifest.

GitHub does not automatically inherit labels from an organization `.github` repository. Each repository therefore retains its own label synchronization workflow, which reads this manifest from its public raw GitHub URL. These workflows run weekly and can also be started manually. They use the calling repository's `GITHUB_TOKEN`, so no organization-wide token is required.

Use the following workflow in a repository:

```yaml
---
name: Sync labels

# yamllint disable-line rule:truthy
on:
  schedule:
    - cron: "17 4 * * 1"
  workflow_dispatch:

permissions:
  contents: read

jobs:
  labels:
    name: ♻️ Sync labels
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: 🚀 Run Label Blueprint
        uses: klaasnicolaas/action-label-blueprint@v1.0.0
        with:
          labels-file: https://raw.githubusercontent.com/dutchdronesquad/.github/main/.github/labels.yml
```

Label Blueprint is non-destructive by default, so repositories retain additional local labels that are not part of the shared manifest. Enable `prune: true` only when a repository must match the central manifest exactly. Labels can include `aliases` when an existing label should be renamed without losing its issue and pull request assignments.

## Release Drafter

The default Release Drafter configuration lives in `.github/release-drafter.yml`. Repositories without a local file at that path automatically use this organization-wide configuration.

The Release Drafter workflow intentionally remains in each repository. This allows repositories to choose their own triggers, permissions, and action version:

```yaml
---
name: Release Drafter

# yamllint disable-line rule:truthy
on:
  push:
    branches:
      - main
  workflow_dispatch:

concurrency:
  group: release-drafter-${{ github.repository }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  update_release_draft:
    name: ✏️ Draft release
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: read
    steps:
      - name: 🚀 Run Release Drafter
        uses: release-drafter/release-drafter@v7.7.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Repository-specific configuration

A repository can override the organization default by adding its own `.github/release-drafter.yml`.

To keep the shared defaults and only override individual scalar settings, extend the organization configuration:

```yaml
---
_extends: DutchDroneSquad/.github
name-template: "service-v$RESOLVED_VERSION"
tag-template: "service-v$RESOLVED_VERSION"
```

List settings such as `categories`, `autolabeler`, and `replacers` are replaced as a whole by default. Release Drafter also supports `append` and `prepend` merge strategies, but category order affects matching and exclusivity. Prefer a complete local list when a repository needs substantially different categories.

A repository that intentionally needs fully independent behavior can use a standalone local configuration without `_extends`.
