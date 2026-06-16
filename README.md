# Data Platform AI Toolkit

[![Ministry of Justice Repository Compliance Badge](https://github-community.service.justice.gov.uk/repository-standards/api/data-platform-ai-toolkit/badge)](https://github-community.service.justice.gov.uk/repository-standards/data-platform-ai-toolkit)

[![Open in Dev Container](https://raw.githubusercontent.com/ministryofjustice/.devcontainer/refs/heads/main/contrib/badge.svg)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/ministryofjustice/data-platform-ai-toolkit)

A central repository of GitHub Copilot instructions and prompt files for Data Platform Services. It provides reusable agent packages — such as language-specific coding instructions — that are distributed to consuming repositories via APM (Agent Package Manager), ensuring consistent AI-assisted development practices across the platform.

## Available Toolkits

| Toolkit              | Package name                   | Source path                     | Description                                                       |
| -------------------- | ------------------------------ | ------------------------------- | ----------------------------------------------------------------- |
| Universal            | `universal-toolkit`            | `toolkits/universal`            | Universal Copilot instructions for all Data Platform repositories |
| Platform Engineering | `platform-engineering-toolkit` | `toolkits/platform-engineering` | Platform Engineering Copilot instructions and prompts             |
| Software Engineering | `software-engineering-toolkit` | `toolkits/software-engineering` | Software Engineering Copilot instructions (Python, Django)        |

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
       - ministryofjustice/data-platform-ai-toolkit/toolkits/universal#0.0.1
   ```
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
apm install universal-toolkit@data-platform                      # add a toolkit by name
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

Each toolkit is versioned independently (`versioning.strategy: tag_pattern`). Pushing a tag of the form `<package-name>-<version>` (for example `universal-toolkit-1.0.0`) triggers the [Release Marketplace](.github/workflows/release-marketplace.yml) workflow, which rebuilds the artifact, attaches a checksum, and publishes a GitHub release.
