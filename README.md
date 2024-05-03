# Get reusable workflow version
GitHub [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) to get commit SHA that GitHub Actions [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) was called with

Workaround for https://github.com/actions/toolkit/issues/1264
  
When a reusable workflow is called, the `github` context is always associated with the caller workflow

https://docs.github.com/en/actions/using-workflows/reusing-workflows#overview

If a reusable workflow needs to checkout files (e.g. a Python script) from its repository—instead of the repository of the caller workflow—it needs to know what version it was called with.

## Usage
```yaml
# Reusable workflow (e.g. build_charm.yaml)
on:
  workflow_call:

jobs:
  foo:
    runs-on: ubuntu-latest
    steps:
      - name: Get workflow version
        id: workflow-version
        uses: canonical/get-workflow-version-action@v1
        with:
          # Repository where reusable workflow is located
          repository-name: canonical/data-platform-workflows
          # Name of reusable workflow
          file-name: build_charm.yaml
          # Only required for private repositories
          github-token: ${{ secrets.GITHUB_TOKEN }}
      # Use the version. For example:
      - name: Install Python CLI
        run: pipx install git+https://github.com/canonical/data-platform-workflows@'${{ steps.workflow-version.outputs.sha }}'#subdirectory=python/cli
    # Only required for private repositories
    permissions:
      actions: read  # Needed for GitHub API call to get workflow version
      
```
