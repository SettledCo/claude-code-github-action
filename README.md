# Claude Code GitHub Action

This GitHub Action integrates the [Claude Code CLI](https://github.com/anthropics/claude-code-cli) with GitHub Actions to automatically generate code changes based on GitHub Issues.

## Features

- Automatically process issues with a specific label (e.g., `claude-code`) using Claude Code.
- Generates code changes based on the issue description.
- Creates or updates Pull Requests with the changes and a summary provided by Claude.
- Uses the Anthropic API for Claude models.

## Prerequisites

- An [Anthropic API key](https://console.anthropic.com/settings/keys).
- The GitHub repository secrets must be configured with `ANTHROPIC_API_KEY` containing your API key.

## Usage

Here's an example workflow (`.github/workflows/claude_handler.yml`) that triggers the action when an issue with the label `claude-code` is opened, edited, or labeled:

```yaml
name: Claude Code Issue Handler

on:
  issues:
    types: [opened, edited, labeled] # Triggers on these issue events

permissions:
  issues: write # To comment on issues
  contents: write # To checkout code, commit, push
  pull-requests: write # To create PRs

jobs:
  process-issue:
    runs-on: ubuntu-latest
    # Only run if the issue has the 'claude-code' label
    if: contains(github.event.issue.labels.*.name, 'claude-code')

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Necessary for creating branches

      - name: Process issue with Claude Code
        # Replace with nicholaslee119/claude-code-github-action@v1 (or latest tag) if using the published action
        uses: ./ # Uses the action in the current repository
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        with:
          issue-number: ${{ github.event.issue.number }}
          issue-body: ${{ github.event.issue.body }}
          issue-title: ${{ github.event.issue.title }}
          # Optional: Specify base branch if not main
          # base-branch: 'develop'
          # Optional: Customize branch name prefix
          # branch-name: 'feature/claude'
          # Optional: Customize allowed tools for Claude Code
          # allowed-tools: 'Edit'
          # Optional: Specify a different Anthropic model
          # anthropic-model-id: 'anthropic.claude-3-haiku-20240307-v1:0'
```

## Input Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `anthropic-model-id` | Claude model ID to use (e.g., `anthropic.claude-3-sonnet-20240229-v1:0`) | No | `anthropic.claude-3-sonnet-20240229-v1:0` |
| `allowed-tools` | Tools to allow Claude Code to use (e.g., `Bash(git diff:*) Edit`) | No | `Bash(git diff:*) Bash(git log:*) Edit` |
| `issue-number` | The issue number to process | Yes | - |
| `issue-body` | The body content of the issue | Yes | - |
| `issue-title` | The title of the issue | Yes | - |
| `branch-name` | The base name for the branch to create (will be appended with `-<issue-number>`) | No | `claude-code/issue` |
| `base-branch` | The base branch to create the PR against | No | `main` |

## Outputs

| Output | Description |
|--------|-------------|
| `pr-number` | The number of the created or updated PR |
| `summary` | Summary of changes made by Claude Code (extracted from Claude's output) |

## How It Works

1.  The workflow triggers on specified issue events (e.g., labeling an issue with `claude-code`).
2.  The action checks out the code.
3.  A new branch is created (or an existing one is checked out) based on the `branch-name` input and issue number.
4.  The `claude-code` CLI is invoked with the issue body as the prompt.
5.  If Claude makes changes, they are committed and pushed to the branch.
6.  A Pull Request is created (or updated if one already exists for the branch) targeting the `base-branch`.
7.  Comments are added to the original issue and/or the PR to link them and provide status updates.

## Environment Variables

-   `GITHUB_TOKEN`: Automatically provided by GitHub Actions. Used for checking out code, committing, pushing, and creating PRs.
-   `ANTHROPIC_API_KEY`: **Required**. Your Anthropic API key. Must be set in the workflow environment using secrets.

## License

MIT
