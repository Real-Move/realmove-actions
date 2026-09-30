# realmove-actions

Reusable GitHub Actions workflows and composite actions for the Real-Move organization.

This repository includes:
- reusable workflows exposed through `workflow_call`
- example event-driven wrapper workflows
- a small sample composite action

## Index

- [Repository Layout](#repository-layout)
- [Reusable Workflows](#reusable-workflows)
  - [Hello](#hello)
  - [Build and Release Debian](#build-and-release-debian)
  - [ROS CI](#ros-ci)
  - [Notify Unlabeled Issue](#notify-unlabeled-issue)
  - [Notify Unlabeled PR](#notify-unlabeled-pr)
  - [Add Opened Issue to Project](#add-opened-issue-to-project)
  - [Trigger Target Workflow](#trigger-target-workflow)
- [Wrapper Workflows](#wrapper-workflows)
- [Composite Action](#composite-action)
- [Example Usage](#example-usage)
- [Notes](#notes)

## Repository Layout

- `.github/workflows/reusable-*.yml`: reusable workflows for other repositories
- `.github/workflows/*.yml`: example wrappers triggered by GitHub events
- `hello/action.yml`: sample composite action
- `.github/workflows/core.repos`: VCS import file used by the ROS CI workflow in `core` mode

## Reusable Workflows

### Hello
File: `.github/workflows/reusable-hello.yml`

Minimal reusable workflow example that prints a hello message and runs the sample `hello` action.

### Build and Release Debian
File: `.github/workflows/reusable-release-deb.yml`

Builds ROS Debian packages in a container, uploads them as artifacts, and can attach them to a GitHub release for tagged versions.

### ROS CI
File: `.github/workflows/reusable-ros-ci.yml`

Runs ROS CI builds with `ros-tooling/action-ros-ci`, either for repository dependencies or for a shared core workspace defined in `core.repos`.

Uses `ghcr.io/real-move/ros-ci:<ros_distro>` on the self-hosted AMD64 runner. Currently
only `humble` is built by `realmove-containers`; publish that image before enabling
this workflow. The image includes ROS build/test tools, Pinocchio, and ccache, so
core builds no longer install Pinocchio separately. Package-specific dependencies
are still installed by rosdep unless the caller explicitly skips installation.

For private GHCR images, the caller can pass `GCR_PAT` with package read access.
Otherwise `REPO_ORG_PAT` is used and must also be able to read the image. Credentials
are applied at container startup, before any workflow steps run.

C/C++ compiler results persist in the runner-local Docker volume
`realmove-ros-ci-ccache`, under a separate repository/distro/architecture directory
with a 5 GiB limit per directory. Different runner machines have independent
caches. Job summaries include per-run cache statistics when compilation occurred
and cumulative cache totals. Tests still run normally; build/install directories
are not restored from cache. Removing the Docker volume clears the compiler cache.

In `deps` mode, `deps_repos_file` is optional. If a configured local file is missing, CI emits a warning and continues without extra repository imports. Existing files and HTTP(S) URLs are passed to the ROS action; invalid contents or URL import failures still fail the build. Core mode continues to use `core.repos`.

### Notify Unlabeled Issue
File: `.github/workflows/reusable-notify-unlabeled-issue.yml`

Waits briefly after an issue is opened or updated, then comments on still-unlabeled issues and can add a helper label for triage.

### Notify Unlabeled PR
File: `.github/workflows/reusable-notify-unlabeled-pr.yml`

Waits briefly after a pull request is opened or updated, then comments on still-unlabeled pull requests and can add a helper label for triage.

### Add Opened Issue to Project
File: `.github/workflows/reusable-add-opened-issue-to-project.yml`

Adds issues or pull requests to a GitHub Projects board, with optional label-based filtering.

### Trigger Target Workflow
File: `.github/workflows/reusable-trigger-target-workflow.yml`

Sends a `repository_dispatch` event to another repository using a PAT, forwarding the source branch in `client_payload.branch_name`.

## Wrapper Workflows

These workflows are ready-to-use examples that call the reusable workflows above.

### Notify Issue Author if No Labels
File: `.github/workflows/notify-unlabeled-issue.yml`

Example wrapper that runs the unlabeled-issue reminder workflow on issue activity.

### Notify PR Author if No Labels
File: `.github/workflows/notify-unlabeled-pr.yml`

Example wrapper that runs the unlabeled-PR reminder workflow on pull request activity.

### Add Opened Issue to Main Project
File: `.github/workflows/add-opened-issue-to-project.yml`

Example wrapper that adds newly opened issues to the main Real-Move GitHub project.

## Composite Action

### Hello
File: `hello/action.yml`

Simple composite action that prints `Hello from action`, mainly as a reference and test action.

## Example Usage

Call a reusable workflow from another repository:

```yaml
name: CI

on:
  pull_request:

jobs:
  ros-ci:
    uses: Real-Move/realmove-actions/.github/workflows/reusable-ros-ci.yml@main
    with:
      ros_distro: humble
      workspace_mode: deps
    secrets:
      REPO_ORG_PAT: ${{ secrets.REPO_ORG_PAT }}
```

## Notes

- Examples in this README reference workflows at `@main`
- For better stability, pin consumers to a tag or commit SHA
