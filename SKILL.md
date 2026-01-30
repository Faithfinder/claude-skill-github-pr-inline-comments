---
name: github-pr-inline-comments
description: This skill should be used when posting inline comments on GitHub pull requests. It provides the correct API format and workflow for adding review comments to specific lines in PR diffs using the gh CLI.
---

# GitHub PR Inline Comments

## Overview

Post inline comments on GitHub pull requests using the `gh` CLI and GitHub API. The key challenge is that line numbers must match the actual file content in the PR branch, not the diff positions.

## Workflow

### Step 1: Get the PR Head Commit SHA

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number} --jq '.head.sha'
```

### Step 2: Find Correct Line Numbers

Line numbers must reference the actual file in the PR branch, not diff positions. Fetch file contents from the PR branch:

```bash
gh api repos/{owner}/{repo}/contents/{file_path}?ref={branch_name} -q '.content' | base64 -d | grep -n "pattern"
```

Or get the branch name first:
```bash
gh pr view {pr_number} --json headRefName -q '.headRefName'
```

### Step 3: Create Review JSON

Create a JSON file with this structure:

```json
{
  "commit_id": "<commit_sha>",
  "event": "COMMENT",
  "body": "Review summary message",
  "comments": [
    {
      "path": "path/to/file.py",
      "line": 42,
      "body": "Comment text with **markdown** support"
    }
  ]
}
```

**Fields:**
- `commit_id`: The HEAD SHA from step 1 (required)
- `event`: One of `COMMENT`, `APPROVE`, or `REQUEST_CHANGES`
- `body`: Overall review message (appears at top)
- `comments`: Array of inline comments
  - `path`: File path relative to repo root
  - `line`: Line number in the file (must exist in the diff)
  - `body`: Comment text (supports GitHub markdown)

### Step 4: Submit the Review

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews --input /tmp/review.json
```

## Common Errors

### "Line could not be resolved"

The line number doesn't exist in the diff. Causes:
- Line number from wrong file version (use PR branch, not local)
- Line not modified in the PR (can only comment on changed lines)
- Off-by-one error

**Fix:** Fetch the actual file from the PR branch and verify line numbers:
```bash
gh api repos/{owner}/{repo}/contents/{path}?ref={branch} -q '.content' | base64 -d | sed -n '{line}p'
```

### "Unprocessable Entity" with positioning errors

Using wrong API format. Do not use `position`, `side`, or `subject_type` fields - use only `line` for the reviews endpoint.

## Complete Example

```bash
# 1. Get commit SHA
COMMIT_SHA=$(gh api repos/myorg/myrepo/pulls/123 --jq '.head.sha')

# 2. Find line numbers in PR branch
gh api repos/myorg/myrepo/contents/src/main.py?ref=feature-branch \
  -q '.content' | base64 -d | grep -n "def buggy_function"

# 3. Create review JSON
cat << 'EOF' > /tmp/review.json
{
  "commit_id": "abc123...",
  "event": "COMMENT",
  "body": "Code review with inline comments",
  "comments": [
    {
      "path": "src/main.py",
      "line": 45,
      "body": "**Bug:** This will raise a KeyError when `data` is empty."
    },
    {
      "path": "src/utils.py",
      "line": 12,
      "body": "Consider using `pathlib` instead of string concatenation."
    }
  ]
}
EOF

# 4. Submit review
gh api repos/myorg/myrepo/pulls/123/reviews --input /tmp/review.json
```

## Tips

- Post multiple comments in one review to avoid notification spam
- Use markdown formatting: `**bold**`, `` `code` ``, code blocks with syntax highlighting
- For multi-line comments, only specify the ending line number
- Comments can only be placed on lines that appear in the diff (added or context lines)
