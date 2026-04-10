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

### Notify Unlabeled Issue
File: `.github/workflows/reusable-notify-unlabeled-issue.yml`

Waits briefly after an issue is opened or updated, then comments on still-unlabeled issues and can add a helper label for triage.

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
