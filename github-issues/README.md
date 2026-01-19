# GitHub Issues Skill

Work with GitHub Issues in environments where GH CLI is not available, specifically designed for Claude Code Web.

## Overview

This skill provides a Python script (`gh_issue.py`) that uses only the Python standard library to interact with GitHub Issues via the REST API. It's particularly useful in Claude Code Web environments where:

- GH CLI cannot be installed
- Branch names must start with `claude/` for push access
- Need programmatic access to issue content and comments

## Quick Start

### 1. Setup Authentication

**Option A: Using GH CLI (if available)**
```bash
gh auth login
```

**Option B: Using Environment Variable**
```bash
export GITHUB_TOKEN="ghp_your_token_here"
```

### 2. Basic Commands

```bash
# Read issue content
python gh_issue.py get 5

# Add comment to issue
python gh_issue.py comment 5 "Your comment here"

# List all comments
python gh_issue.py comments 5
```

### 3. Typical Workflow in Claude Code Web

```bash
# When user says: "work on issue #5"

# 1. Read the issue
python .claude/skills/shared/github-issues/gh_issue.py get 5

# 2. Create branch with claude/ prefix (REQUIRED for Claude Code Web)
git checkout -b claude/issue-5

# 3. Do the work...

# 4. Commit with proper format
git commit -m "Fix issue #5: Brief description

Fixes #5"

# 5. Push branch
git push origin claude/issue-5

# 6. Update issue
python .claude/skills/shared/github-issues/gh_issue.py comment 5 \
  "Work completed in branch \`claude/issue-5\`"
```

## Files

- **SKILL.md** - Complete skill documentation with workflows and patterns
- **gh_issue.py** - Python script for GitHub API interactions (stdlib only)
- **PROJECT_CLAUDE_EXAMPLE.md** - Sample sections to add to project CLAUDE.md files
- **README.md** - This file

## Requirements

- Python 3.6+ (uses only standard library)
- Git repository with GitHub remote
- GitHub authentication (gh CLI or GITHUB_TOKEN env var)
- Internet connection for GitHub API

## Authentication

### Creating a GitHub Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Claude Code Web")
4. Select scopes:
   - `repo` (Full control of private repositories) - Required
5. Click "Generate token"
6. Copy the token (starts with `ghp_`)
7. Set environment variable:
   ```bash
   export GITHUB_TOKEN="ghp_your_token_here"
   ```

### Making Token Persistent

Add to your shell profile:

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_your_token_here"
```

Or use a credential manager:

```bash
# macOS Keychain
security add-generic-password -a "$USER" -s github_token -w "ghp_your_token"

# Retrieve it
export GITHUB_TOKEN=$(security find-generic-password -a "$USER" -s github_token -w)
```

## Usage in Projects

### As Submodule

When this skills repository is added as a submodule:

```bash
# Typical path
.claude/skills/shared/github-issues/gh_issue.py

# Usage
python .claude/skills/shared/github-issues/gh_issue.py get 5
```

### Create Project Helper Script

Make it easier to use in your project:

```bash
# Create bin/gh-issue
#!/bin/bash
python "$(dirname "$0")/../.claude/skills/shared/github-issues/gh_issue.py" "$@"

# Make executable
chmod +x bin/gh-issue

# Usage
./bin/gh-issue get 5
```

## Examples

### Example 1: Start Work on Issue

```bash
#!/bin/bash
ISSUE_NUM=5

# Read issue
python gh_issue.py get $ISSUE_NUM

# Create branch
git checkout -b "claude/issue-${ISSUE_NUM}"

echo "Ready to work on issue #${ISSUE_NUM}"
```

### Example 2: Complete Issue Work

```bash
#!/bin/bash
ISSUE_NUM=5
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

# Ensure branch starts with claude/
if [[ ! $BRANCH == claude/* ]]; then
  echo "Error: Branch must start with claude/"
  exit 1
fi

# Push branch
git push origin "$BRANCH"

# Update issue
python gh_issue.py comment $ISSUE_NUM \
"✅ Work completed

Branch: \`${BRANCH}\`
Commit: \`${COMMIT}\`

Changes are ready for review."

echo "Issue #${ISSUE_NUM} updated!"
```

### Example 3: Daily Standup - List Your Issues

```bash
#!/bin/bash
# List issues assigned to you

TOKEN=$(gh auth token 2>/dev/null || echo "$GITHUB_TOKEN")
REPO_INFO=$(git remote get-url origin | sed 's/.*github.com[:/]\(.*\)\.git/\1/')

curl -s -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/$REPO_INFO/issues?assignee=@me&state=open" \
  | python -m json.tool
```

## Troubleshooting

### "No GitHub token found"

**Problem:** Script cannot find authentication token

**Solution:**
```bash
# Check if gh CLI is authenticated
gh auth status

# Or set token manually
export GITHUB_TOKEN="ghp_your_token"
```

### "Could not get git remote"

**Problem:** Not in a git repository or no GitHub remote

**Solution:**
```bash
# Check git remote
git remote -v

# Should show GitHub URL like:
# origin  https://github.com/user/repo.git (fetch)
```

### "Branch push rejected (GH013)"

**Problem:** Branch name doesn't start with `claude/` in Claude Code Web

**Solution:**
```bash
# Rename branch
git branch -m old-name claude/issue-5

# Or create new branch
git checkout -b claude/issue-5
```

### "Permission denied (403)"

**Problem:** Token doesn't have required permissions

**Solution:**
- Create new token with `repo` scope
- Check token hasn't expired
- Verify token is set correctly: `echo $GITHUB_TOKEN`

## Claude Code Web Specifics

### Branch Naming Rules

Claude Code Web REQUIRES branches to start with `claude/` for push access:

✅ Allowed:
- `claude/issue-5`
- `claude/issue-12-add-feature`
- `claude/fix-bug`

❌ Not Allowed:
- `issue-5`
- `feature/add-login`
- `main`

### Workflow Adaptation

Standard GitHub flow:
```bash
git checkout -b feature/new-login  # ❌ Won't work in Claude Code Web
```

Claude Code Web flow:
```bash
git checkout -b claude/issue-5  # ✅ Works!
```

### Session Persistence

Claude Code Web sessions are temporary. To preserve work:

1. **Push frequently:**
   ```bash
   git push origin claude/issue-5
   ```

2. **Comment on issue with branch name:**
   ```bash
   python gh_issue.py comment 5 "Work in progress: \`claude/issue-5\`"
   ```

3. **Use descriptive branch names:**
   ```bash
   # Include session ID or timestamp if needed
   git checkout -b "claude/issue-5-$(date +%Y%m%d)"
   ```

## Integration with Project CLAUDE.md

See `PROJECT_CLAUDE_EXAMPLE.md` for complete examples to add to your project's CLAUDE.md file.

Quick snippet:

```markdown
## GitHub Issues

When working on issues, use branch naming: `claude/issue-<N>`

Commands:
- Read: `python .claude/skills/shared/github-issues/gh_issue.py get <N>`
- Comment: `python .claude/skills/shared/github-issues/gh_issue.py comment <N> "message"`
```

## API Reference

### gh_issue.py Commands

**Get issue details:**
```bash
python gh_issue.py get <issue_number>
```
Output: Formatted issue with title, description, labels, assignees

**Add comment:**
```bash
python gh_issue.py comment <issue_number> "Comment text"
```
Output: Comment URL

**List comments:**
```bash
python gh_issue.py comments <issue_number>
```
Output: All comments with author and timestamps

## Design Notes

### Why Python stdlib only?

- No dependency installation in restricted environments
- Faster startup (no pip install needed)
- Portable across all Python 3.6+ environments
- Minimal attack surface

### Why not use GH CLI?

- GH CLI not available in Claude Code Web
- Cannot install system packages in browser environment
- Python is already available

### Authentication Strategy

1. Try `gh auth token` first (best UX if available)
2. Fall back to `GITHUB_TOKEN` env var
3. Fail with clear error message

This provides best experience across environments.

## Contributing

When improving this skill:

1. Keep Python stdlib-only requirement
2. Test in actual Claude Code Web environment
3. Document branch naming requirements clearly
4. Provide project integration examples
5. Update PROJECT_CLAUDE_EXAMPLE.md with new patterns

## License

Apache License 2.0 (same as parent repository)
