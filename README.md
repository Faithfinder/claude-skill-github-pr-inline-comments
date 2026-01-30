# GitHub PR Inline Comments Skill

A [Claude Code](https://claude.ai/code) skill for posting inline review comments on GitHub pull requests.

## Installation

Add to your Claude Code skills:

```bash
claude mcp add-skill https://github.com/Faithfinder/claude-skill-github-pr-inline-comments
```

Or manually clone to your skills directory:

```bash
git clone https://github.com/Faithfinder/claude-skill-github-pr-inline-comments.git ~/.claude/skills/github-pr-inline-comments
```

## What it does

This skill teaches Claude how to post inline comments on GitHub PRs using the `gh` CLI. The main challenge it solves is getting the correct line numbers - they must reference the actual file in the PR branch, not local files or diff positions.

## Usage

When reviewing a PR, Claude will automatically use this skill to:

1. Get the PR's head commit SHA
2. Fetch file contents from the PR branch to find correct line numbers
3. Create properly formatted review JSON
4. Submit the review via GitHub API

## Key insight

The "Line could not be resolved" error happens when line numbers don't match the PR branch. Always fetch file contents from the PR branch:

```bash
gh api repos/{owner}/{repo}/contents/{path}?ref={branch} -q '.content' | base64 -d | grep -n "pattern"
```
