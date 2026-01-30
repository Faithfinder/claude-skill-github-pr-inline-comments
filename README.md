# Faithfinder's Claude Code Plugins

A marketplace of Claude Code plugins.

## Installation

Add this marketplace to Claude Code:

```shell
/plugin marketplace add Faithfinder/claude-skill-github-pr-inline-comments
```

Or for local testing:

```shell
/plugin marketplace add ./path/to/this/directory
```

## Available Plugins

### github-pr-inline-comments

Post inline review comments on GitHub PRs using the `gh` CLI. Solves the tricky line number resolution issues.

**Install:**

```shell
/plugin install github-pr-inline-comments@faithfinder-plugins
```

**What it does:**

When reviewing a PR, Claude will:
1. Get the PR's head commit SHA
2. Fetch file contents from the PR branch to find correct line numbers
3. Create properly formatted review JSON
4. Submit the review via GitHub API

See the [plugin README](./plugins/github-pr-inline-comments/README.md) for more details.
