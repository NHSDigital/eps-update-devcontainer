# EPS Update devcontainer versions

This repository contains a composite GitHub Action that updates the devcontainer version in devcontainer.json to the latest published version

## What this repository contains

- A composite GitHub Action defined in `action.yml`

## Action overview

The action is intended to be called from another repository. It:

1. Checks out the calling repository at the requested base branch
2. Gets the current image name and version from .devcontainer/devcontainer.json
2. Gets the latest published image for the image name from .devcontainer/devcontainer.json
3. Updates the version in .devcontainer/devcontainer.json
4. If there was a change, it Creates a signed pull request containing the new version


## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `calling_repo_base_branch` | No | `main` | Base branch in the calling repository that the pull request should target |
| `CREATE_PULL_REQUEST_APP_ID` | Yes | None | GitHub App ID used to generate a token for pull request creation |
| `CREATE_PULL_REQUEST_PEM` | Yes | None | GitHub App private key in PEM format |

## Example usage

```yaml
name: Update devcontainer version

on:
  workflow_dispatch:
  schedule:
    - cron: '0 6 * * 1'

permissions: {}

jobs:
  update-devcontainer-version:
    runs-on: ubuntu-22.04
    environment: create_pull_request
    permissions:
      contents: read
      packages: read

    steps:
      - name: Update devcontainer version
        uses: NHSDigital/eps-update-devcontainer@95118f6746ca7081258cc7f651dca1c5bb7339f1
        with:
          calling_repo_base_branch: main
          CREATE_PULL_REQUEST_APP_ID: ${{ secrets.CREATE_PULL_REQUEST_APP_ID }}
          CREATE_PULL_REQUEST_PEM: ${{ secrets.CREATE_PULL_REQUEST_PEM }}
```

## Pull request behavior

When changes are detected, the action creates a pull request with:

- Branch prefix: `copilot-instructions-sync`
- Commit message: `Upgrade: [dependabot] - sync Copilot instructions`
- Pull request title: `Upgrade: [dependabot] - sync Copilot instructions`
- Signed commits enabled
- Automatic branch cleanup enabled

The pull request body includes the ref used for the sync.

## Prerequisites

Before using the action in a repository, ensure that https://github.com/NHSDigital/electronic-prescription-service-account-resources/blob/main/scripts/set_github_secrets.py has been run to create the environment and secrets

## Notes

- The action is implemented as a composite action in `action.yml`
- It uses pinned action SHAs for its external action dependencies
- The action opens a pull request instead of pushing directly to the base branch
