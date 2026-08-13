# MrGreenBoutiqueOffices GitHub defaults

This repository contains shared GitHub configuration for the
`MrGreenBoutiqueOffices` organization.

## Release Drafter

The default Release Drafter configuration lives in
`.github/release-drafter.yml`. Repositories without a local file at that path
automatically use this organization-wide configuration.

The Release Drafter workflow intentionally remains in each repository. This
allows repositories to choose their own triggers, permissions, and action
version:

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

A repository can override the organization default by adding its own
`.github/release-drafter.yml`.

To keep the shared defaults and only override individual scalar settings, extend
the organization configuration:

```yaml
---
_extends: MrGreenBoutiqueOffices/.github
name-template: "service-v$RESOLVED_VERSION"
tag-template: "service-v$RESOLVED_VERSION"
```

List settings such as `categories`, `autolabeler`, and `replacers` are replaced
as a whole by default. Release Drafter also supports `append` and `prepend`
merge strategies, but category order affects matching and exclusivity. Prefer a
complete local list when a repository needs substantially different categories.

A repository that intentionally needs fully independent behavior can use a
standalone local configuration without `_extends`.

### Label aliases for lean type labels

Category matching stays on the shared org labels. Where product repos use lean
type labels instead of the historical org names, the shared config also accepts:

| Changelog section | Historical org labels | Lean aliases |
| ----------------- | --------------------- | ------------ |
| Bug fixes | `bugfix` | `bug` |
| Maintenance | `maintenance`, `ci` | `tech-debt` |

SemVer resolution remains label-driven via `major` / `minor`, with patch as the
default when neither is present. Do not introduce a second release label
taxonomy in application repositories.

