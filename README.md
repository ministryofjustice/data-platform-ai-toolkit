# Data Platform AI Toolkit

[![Ministry of Justice Repository Compliance Badge](https://github-community.service.justice.gov.uk/repository-standards/api/data-platform-ai-toolkit/badge)](https://github-community.service.justice.gov.uk/repository-standards/data-platform-ai-toolkit)

[![Open in Dev Container](https://raw.githubusercontent.com/ministryofjustice/.devcontainer/refs/heads/main/contrib/badge.svg)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/ministryofjustice/data-platform-ai-toolkit)

A central repository of GitHub Copilot instructions and prompt files for Data Platform Services. It provides reusable agent packages — such as language-specific coding instructions — that are distributed to consuming repositories via APM (Agent Package Manager), ensuring consistent AI-assisted development practices across the platform.

## Available Toolkits

| Toolkit              | Package name           | Source path                     | Description                                                       |
| -------------------- | ---------------------- | ------------------------------- | ----------------------------------------------------------------- |
| Universal            | `universal`            | `toolkits/universal`            | Universal Copilot instructions for all Data Platform repositories |
| Platform Engineering | `platform-engineering` | `toolkits/platform-engineering` | Platform Engineering Copilot instructions and prompts             |
| Software Engineering | `software-engineering` | `toolkits/software-engineering` | Software Engineering Copilot instructions (Python, Django)        |

## Setup Instructions

Consuming repositories declare the toolkits they need in their own `apm.yml` and install them with `apm install`. This is the primary, fully automated flow and runs unattended inside a development container.

1. [Install](https://github.com/ministryofjustice/.devcontainer#adding-a-development-container-to-a-project) the Ministry of Justice [Development Container](https://github.com/ministryofjustice/.devcontainer) in your repository.
1. Add the APM feature to your `.devcontainer/devcontainer.json`:
   ```json
   {
     "features": {
       "ghcr.io/ministryofjustice/devcontainer-feature/apm:": {}
     }
   }
   ```
1. Populate `.devcontainer/devcontainer-lock.json` with the resolved hash:
   ```json
   {
     "features": {
       "ghcr.io/ministryofjustice/devcontainer-feature/apm:": {
         "version": "1.0.0",
         "resolved": "ghcr.io/ministryofjustice/devcontainer-feature/apm@sha256:b562e126312767cf69b772bce79ae551a82f97c2eda7cce400fe068530261f71",
         "integrity": "sha256:b562e126312767cf69b772bce79ae551a82f97c2eda7cce400fe068530261f71"
       }
     }
   }
   ```
1. Create an `apm.yml` in the root of your repository to declare this toolkit as a dependency:

   ```yaml
   name: <name>
   version: 1.0.0
   description: APM project for <name>
   author: Data Platform Engineering
   targets:
     - copilot
   dependencies:
     apm:
       - ministryofjustice/data-platform-ai-toolkit/toolkits/universal#^1.0.0
   ```

   The `#^1.0.0` is a semver range: APM resolves it to the highest matching
   `universal-v<version>` release tag and pins the exact commit in
   `apm.lock.yaml`. Use a caret range to accept compatible updates, or pin an
   exact tag (`#universal-v1.0.0`) or branch (`#main`) instead.

1. Add `apm install` to your `.devcontainer/post-create.sh` script:
   ```bash
   #!/usr/bin/env bash
   apm install
   ```

## Optional: discovering toolkits via the marketplace

This repository is also published as an **APM marketplace** — a curated index of the toolkits above. The marketplace is a discovery and naming layer; it is **not** required for the automated `apm install` flow described above, which resolves dependencies directly from `apm.yml`.

It is useful when a developer wants to browse what is available and add a toolkit by friendly name rather than by repository path:

```bash
apm marketplace add ministryofjustice/data-platform-ai-toolkit   # register the catalogue (one-time)
apm marketplace browse data-platform                             # list available toolkits
apm install universal@data-platform                              # add a toolkit by name
```

`apm install <toolkit>@data-platform` resolves the toolkit through the marketplace and writes an ordinary entry into your `apm.yml` `dependencies`, after which the standard `apm install` flow takes over. The registration step (`apm marketplace add`) is interactive, so it does not replace the unattended `apm install` in your `post-create.sh`.

## Maintaining the marketplace

The marketplace is defined by the `marketplace:` block in the root [`apm.yml`](apm.yml) and compiled to [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json). To add or update a toolkit:

1. Edit the `packages:` list in `apm.yml` (or use `apm marketplace package add ./toolkits/<name>`).
1. Regenerate and validate the artifact:
   ```bash
   apm pack --check-versions --check-clean
   ```
1. Commit both `apm.yml` and the generated `.claude-plugin/marketplace.json`.

### Creating a new release

To release a new version of a toolkit (for example bumping universal to
`1.1.0`), prepare `main` before drafting the release:

1. Bump `version:` in **both** `toolkits/<name>/apm.yml` and that package's
   entry in the root [`apm.yml`](apm.yml).
1. Regenerate and validate the artifact:
   ```bash
   apm pack --check-versions --check-clean
   ```
1. Commit `apm.yml` and the updated `.claude-plugin/marketplace.json` to `main`.
1. Then draft the release with the new tag (e.g. `universal-v1.1.0`).

Skipping the bump fails the release gates: `--check-clean` rejects a committed
`marketplace.json` that does not match a fresh `apm pack`.

To draft the new release:

1. Go to **Releases → Draft a new release**.
1. **Choose a tag → Create new tag**, of the form `<package-name>-v<version>`
   (for example `universal-v1.0.0`). The `<package-name>` matches the toolkit's
   source directory (for example `universal`) and `<version>` must match that
   package's `version:` in the root `apm.yml`. The `v` prefix and the matching
   name are what let consumers pin a semver range such as
   `toolkits/universal#^1.0.0`.
1. Set **Target** to `main`.
1. Expand **Set as the latest release** and choose **Keep existing** so the
   release is **not** marked as latest.
1. Add a title which matches the tag, and generate release notes, then click **Publish release**.
