---
name: github-issues-workflow
description: Work with GitHub issues in environments without gh CLI (like Claude Code Web). Use when starting work on an issue, reading issue content, or adding comments to issues.
allowed-tools: Bash(python*), Bash(git*), Read, Edit, Write
---

# GitHub Issues Workflow

This skill enables working with GitHub issues in restricted environments like Claude Code Web where:
- GH CLI is not available or cannot be installed
- Need to read issue content and add comments programmatically

## Prerequisites

**Authentication**: One of the following:
- GH CLI installed with `gh auth login` completed, OR
- `GITHUB_TOKEN` environment variable set with a GitHub personal access token

**Python**: Python 3 (uses only standard library, no external packages needed)

## Core Script: gh_issue.py

Located at `github-issues/gh_issue.py` in the skills directory.

### Commands

```bash
# Get issue details
python gh_issue.py get <issue_number>

# Add comment to issue
python gh_issue.py comment <issue_number> "Your comment message"

# List all comments on an issue
python gh_issue.py comments <issue_number>
```

The script automatically:
- Detects repository owner/name from git remote
- Gets authentication token from gh CLI or GITHUB_TOKEN env var
- Handles all GitHub API interactions

## Claude Code Web Workflow

### Starting Work on an Issue

When a user says "work on issue #5" or similar:

1. **Read the issue content:**
   ```bash
   python /path/to/skills/github-issues/gh_issue.py get 5
   ```

2. **Implement the changes** described in the issue
   - Claude Code Web handles branch creation automatically
   - Focus on implementing the requirements

3. **Commit with proper message format:**
   ```bash
   git commit -m "Fix issue #5: Brief description

   Detailed explanation of changes.

   Fixes #5"
   ```

4. **After pushing, add progress comment to issue:**
   ```bash
   BRANCH=$(git branch --show-current)
   COMMIT=$(git rev-parse --short HEAD)

   python /path/to/skills/github-issues/gh_issue.py comment 5 \
     "Work completed in branch \`$BRANCH\` at commit $COMMIT. Ready for review."
   ```

## Integration with Project Workflow

### In Project CLAUDE.md

Add this section to your project's CLAUDE.md:

````markdown
## GitHub Issues Workflow

When asked to work on an issue:

1. **Read issue content:**
   ```bash
   python .claude/skills/shared/github-issues/gh_issue.py get <number>
   ```

2. **Implement changes** as described in the issue

3. **Commit with proper format:**
   ```
   Fix issue #<number>: Brief description

   Detailed explanation.

   Fixes #<number>
   ```

4. **After work is pushed, add comment to issue:**
   ```bash
   BRANCH=$(git branch --show-current)
   COMMIT=$(git rev-parse HEAD)
   python .claude/skills/shared/github-issues/gh_issue.py comment <number> \
     "Work completed in branch \`$BRANCH\` at commit $COMMIT"
   ```

**Note**: The `Fixes #<number>` keyword will automatically close the issue when merged to main.
````

## Authentication Setup

### Using GH CLI (Recommended)

```bash
# Install gh CLI (if available in environment)
# macOS: brew install gh
# Linux: see https://github.com/cli/cli#installation

# Authenticate
gh auth login

# Verify
gh auth status
```

### Using Environment Variable

```bash
# Create a Personal Access Token on GitHub:
# Settings → Developer settings → Personal access tokens → Generate new token
# Required scopes: repo (full control of private repositories)

export GITHUB_TOKEN="ghp_your_token_here"

# Add to shell profile for persistence
echo 'export GITHUB_TOKEN="ghp_your_token_here"' >> ~/.bashrc
```

## Common Patterns

### Pattern 1: Issue Discovery

```bash
# User asks: "What's issue #5 about?"

python gh_issue.py get 5
```

### Pattern 2: Starting New Work

```bash
# User asks: "Work on issue #5"

# Read issue to understand requirements
python gh_issue.py get 5

# Then implement changes
# Claude Code Web will handle branch creation
```

### Pattern 3: Progress Update

```bash
# After implementing changes

BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

python gh_issue.py comment 5 \
"Progress update:
- Implemented main feature
- Added tests
- Updated documentation

Branch: \`$BRANCH\`
Commit: \`$COMMIT\`"
```

### Pattern 4: Completion Notification

```bash
# After all work is done and pushed

BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse HEAD)

python gh_issue.py comment 5 \
"✅ Work completed and pushed.

Branch: \`$BRANCH\`
Commit: \`$COMMIT\`

Ready for review. This will close the issue when merged to main."
```

## Error Handling

### No Token Available

```
Error: No GitHub token found.
Either install gh CLI and run 'gh auth login', or set GITHUB_TOKEN env var.
```

**Solution**: Set up authentication (see Authentication Setup section)

### Invalid Issue Number

```
Error: GitHub API request failed: 404 Not Found
```

**Solution**: Verify issue number exists in the repository

### Permission Denied

```
Error: GitHub API request failed: 403 Forbidden
```

**Solution**:
- Ensure token has `repo` scope
- Check token hasn't expired
- Verify token is set correctly: `echo $GITHUB_TOKEN`

## Integration with Other Skills

### With python-dev Skill

Before committing changes for an issue:

```bash
# Run quality checks
ruff format .
ruff check . --fix
ty check
pytest

# Then commit
git commit -m "Fix issue #5: Description

Fixes #5"
```

### With google-cloud Skills

When issue involves GCP deployment:

```bash
# Read issue
python gh_issue.py get 8

# Make changes and deploy
gcloud run deploy service-name ...

# Comment with deployment info
BRANCH=$(git branch --show-current)
python gh_issue.py comment 8 \
"Deployed to Cloud Run: https://service-url.run.app

Branch: \`$BRANCH\`"
```

## Script Location in Projects

When this skill is included as a submodule, typical path:

```
.claude/skills/shared/github-issues/gh_issue.py
```

Use in commands:
```bash
python .claude/skills/shared/github-issues/gh_issue.py get 5
```

Or create a project helper script:
```bash
# In project root, create bin/gh-issue
#!/bin/bash
python "$(dirname "$0")/../.claude/skills/shared/github-issues/gh_issue.py" "$@"

# Usage:
./bin/gh-issue get 5
```

## Remember

- ✅ Read issue content before starting work
- ✅ Add comments to keep issue updated with progress
- ✅ Include branch name and commit hash in comments
- ✅ Use `Fixes #<number>` in commit message to auto-close issue
- ✅ Push regularly so work isn't lost between sessions
- ✅ Let Claude Code Web handle branch creation automatically
