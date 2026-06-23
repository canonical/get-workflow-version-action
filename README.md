# Get reusable workflow version

> [!CAUTION]
> This action is **deprecated** and is no longer maintained. Use `${{ job.workflow_sha }}` instead.
> 
> On 2026-04-23, GitHub added `job.workflow_sha`. Previously, this action provided a workaround for a limitation in GitHub. Now that GitHub supports this functionality natively, this action is no longer needed.
> 
> Announcement for `job.workflow_sha`: https://github.com/actions/runner/issues/2417#issuecomment-4307892505
> 
> Documentation for `job.workflow_sha`: https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#job-context
> 
> As of 2026-06-23, `job.workflow_sha` is not available on GitHub Enterprise Server. While we do not plan to continue maintaining this action, users of GitHub Enterprise Server are welcome to fork this action and continue maintaining it themselves. This action is available under the Apache-2.0 license.
> 
> ### Example migration
> 
> Replace
> ```yaml
> jobs:
>   foo:
>     runs-on: ubuntu-latest
>     steps:
>       - name: Get workflow version
>         id: workflow-version
>         uses: canonical/get-workflow-version-action@v1
>         with:
>           repository-name: canonical/data-platform-workflows
>           file-name: build_charm.yaml
>           github-token: ${{ secrets.GITHUB_TOKEN }}
>       - name: Install Python CLI
>         run: pipx install git+https://github.com/canonical/data-platform-workflows@"${VAR_SHA}"#subdirectory=_cli
>         env:
>           VAR_SHA: ${{ steps.workflow-version.outputs.sha }}
>     permissions:
>       actions: read  # Needed for GitHub API call to get workflow version
> ```
> with
> ```yaml
> jobs:
>   foo:
>     runs-on: ubuntu-latest
>     steps:
>       - name: Install Python CLI
>         run: pipx install git+https://github.com/canonical/data-platform-workflows@"${VAR_SHA}"#subdirectory=_cli
>         env:
>           VAR_SHA: ${{ job.workflow_sha }}
>     permissions: {}
> ```

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
        run: pipx install git+https://github.com/canonical/data-platform-workflows@"${VAR_SHA}"#subdirectory=_cli
        env:
          VAR_SHA: ${{ steps.workflow-version.outputs.sha }}
    permissions:
      # Only required for private repositories. For public repositories, use `permissions: {}`
      actions: read  # Needed for GitHub API call to get workflow version
```
