# Data Platform AI Toolkit

[![Ministry of Justice Repository Compliance Badge](https://github-community.service.justice.gov.uk/repository-standards/api/data-platform-ai-toolkit/badge)](https://github-community.service.justice.gov.uk/repository-standards/data-platform-ai-toolkit)

[![Open in Dev Container](https://raw.githubusercontent.com/ministryofjustice/data-platform-ai-toolkit/refs/heads/main/contrib/badge.svg)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/ministryofjustice/data-platform-ai-toolkit)

A central repository of GitHub Copilot instructions and prompt files for Data Platform Services. It provides reusable agent packages — such as language-specific coding instructions — that are distributed to consuming repositories via APM (Agent Package Manager), ensuring consistent AI-assisted development practices across the platform.

## Setup Instructions

To consume Agent Packages defined in this repository:

1. [Install](https://github.com/ministryofjustice/.devcontainer#adding-a-development-container-to-a-project) the Ministry of Justice [Development Container](https://github.com/ministryofjustice/.devcontainer) in your repository.
1. Add the APM feature to your `.devcontainer/devcontainer.json`:
   ```json
   {
     "features": {
       "ghcr.io/ministryofjustice/devcontainer-feature/apm:1": {}
     }
   }
   ```
1. Populate `.devcontainer/devcontainer-lock.json` with the resolved hash:
   ```json
   {
     "features": {
       "ghcr.io/ministryofjustice/devcontainer-feature/apm:1": {
         "version": "1.0.0",
         "resolved": "ghcr.io/ministryofjustice/devcontainer-feature/apm@sha256:b562e126312767cf69b772bce79ae551a82f97c2eda7cce400fe068530261f71",
         "integrity": "sha256:b562e126312767cf69b772bce79ae551a82f97c2eda7cce400fe068530261f71"
       }
     }
   }
   ```
1. Create an `apm.yml` in the root of your repository to declare this toolkit as a dependency:
   ```yaml
   name: my-project
   version: 1.0.0
   dependencies:
     - name: data-platform-ai-toolkit
       source: github:ministryofjustice/data-platform-ai-toolkit
   ```
1. Add `apm install` to your `.devcontainer/post-create.sh` script:
   ```bash
   #!/usr/bin/env bash
   apm install
   ```
