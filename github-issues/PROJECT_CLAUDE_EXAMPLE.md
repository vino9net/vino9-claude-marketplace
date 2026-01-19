# Sample Project CLAUDE.md Sections

This file contains example sections to add to your project's CLAUDE.md file when using the github-issues skill.

---

## Example 1: Basic GitHub Issues Integration

Add this to your project's CLAUDE.md:

```markdown
## GitHub Issues Workflow

This project uses GitHub Issues for task tracking and feature development.

### Starting Work on an Issue

When asked to "work on issue #N" or similar:

1. **Read the issue:**
   ```bash
   python .claude/skills/shared/github-issues/gh_issue.py get N
   ```

2. **Implement the changes** described in the issue

3. **Commit with proper message format:**
   ```bash
   git commit -m "Fix issue #N: Brief description

   Detailed explanation of changes.

   Fixes #N"
   ```

4. **After pushing, add progress comment:**
   ```bash
   BRANCH=$(git branch --show-current)
   COMMIT=$(git rev-parse HEAD)
   python .claude/skills/shared/github-issues/gh_issue.py comment N \
     "Work completed in branch \`$BRANCH\` at commit $COMMIT. Ready for review."
   ```

### Commit Message Format

Use this format for issue-related commits:

```
Fix issue #<number>: <brief one-line description>

<Detailed explanation of what was changed and why>

Fixes #<number>
```

The `Fixes #<number>` keyword will automatically close the issue when merged to main.
```

---

## Example 2: Comprehensive Integration with Testing

Add this to your project's CLAUDE.md:

```markdown
## GitHub Issues and Development Workflow

### Issue-Driven Development Process

1. **Issue Discovery**
   ```bash
   # Read issue details
   python .claude/skills/shared/github-issues/gh_issue.py get N

   # List existing comments to see discussion
   python .claude/skills/shared/github-issues/gh_issue.py comments N
   ```

2. **Implementation**
   - Follow the requirements in the issue
   - Write tests for new functionality
   - Update documentation as needed

3. **Pre-Commit Quality Checks**
   ```bash
   # Format and lint (if Python project)
   ruff format .
   ruff check . --fix

   # Type checking
   ty check

   # Run tests
   pytest

   # Only proceed if all checks pass
   ```

4. **Commit**
   ```bash
   git commit -m "Fix issue #N: <description>

   <Detailed changes>

   Fixes #N"
   ```

5. **Update Issue with Progress**
   ```bash
   # After pushing, get current commit
   COMMIT=$(git rev-parse --short HEAD)
   BRANCH=$(git branch --show-current)

   # Add comment
   python .claude/skills/shared/github-issues/gh_issue.py comment N \
   "🤖 Changes completed and pushed.

   **Branch:** \`${BRANCH}\`
   **Commit:** \`${COMMIT}\`

   **Changes:**
   - Implemented feature/fix as described
   - Added tests with coverage
   - Updated documentation

   Ready for review. This will close issue #N when merged."
   ```

### Helper Scripts

Create `bin/start-issue` in your project:

```bash
#!/bin/bash
set -e

if [ -z "$1" ]; then
  echo "Usage: bin/start-issue <issue_number>"
  exit 1
fi

ISSUE_NUM="$1"
SKILLS_DIR=".claude/skills/shared/github-issues"

echo "=== Starting work on issue #${ISSUE_NUM} ==="
echo ""

# Read issue
python "${SKILLS_DIR}/gh_issue.py" get "${ISSUE_NUM}"

echo ""
echo "Ready to start work!"
```

Create `bin/finish-issue` in your project:

```bash
#!/bin/bash
set -e

if [ -z "$1" ]; then
  echo "Usage: bin/finish-issue <issue_number> [comment]"
  exit 1
fi

ISSUE_NUM="$1"
COMMENT="${2:-Work completed and pushed}"
SKILLS_DIR=".claude/skills/shared/github-issues"

BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

# Add comment to issue
echo "Adding comment to issue #${ISSUE_NUM}"
python "${SKILLS_DIR}/gh_issue.py" comment "${ISSUE_NUM}" \
"${COMMENT}

Branch: \`${BRANCH}\`
Commit: \`${COMMIT}\`

Ready for review."

echo "Done! Issue #${ISSUE_NUM} updated."
```

Make scripts executable:
```bash
chmod +x bin/start-issue bin/finish-issue
```

### Authentication Setup

Ensure GitHub authentication is configured:

```bash
# Option 1: Using gh CLI (if available)
gh auth login
gh auth status

# Option 2: Using environment variable
export GITHUB_TOKEN="ghp_your_token_here"
# Add to ~/.bashrc or ~/.zshrc for persistence
```
```

---

## Example 3: Quick Reference Card

Add this compact reference to your project's CLAUDE.md:

```markdown
## GitHub Issues Quick Reference

**Read Issue:**
```bash
python .claude/skills/shared/github-issues/gh_issue.py get <N>
```

**Commit Format:**
```
Fix issue #N: Description

Fixes #N
```

**Add Comment (after pushing):**
```bash
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse HEAD)
python .claude/skills/shared/github-issues/gh_issue.py comment <N> \
  "Work completed in branch \`$BRANCH\` at commit $COMMIT"
```
```

---

## Example 4: Integration with Existing Workflow

Add this to complement existing development guidelines:

```markdown
## Issue Workflow

### Overview

Our project uses GitHub Issues for tracking work.

### Quick Start

```bash
# When starting: "work on issue #N"
python .claude/skills/shared/github-issues/gh_issue.py get N

# After completing work and pushing
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse HEAD)
python .claude/skills/shared/github-issues/gh_issue.py comment N \
  "Completed in branch \`$BRANCH\` at commit $COMMIT"
```

### Integration with Project Standards

1. **Python projects:** Run `ruff format .` and `pytest` before committing
2. **GCP projects:** Test deployment to dev environment before pushing
3. **All projects:** Include `Fixes #N` in commit message

### Environment Setup

Set `GITHUB_TOKEN` in your environment:
```bash
export GITHUB_TOKEN="$(gh auth token)"  # If gh CLI available
# OR
export GITHUB_TOKEN="ghp_your_token"    # Manual token
```

Add to your shell profile for persistence.
```

---

## Notes for Skill Users

When integrating this skill into your project:

1. **Choose the example** that best matches your workflow complexity
2. **Adjust paths** if your submodule location differs from `.claude/skills/shared`
3. **Add authentication instructions** if your team uses specific token management
4. **Include helper scripts** if you want to streamline the workflow
5. **Keep it simple** - focus on reading issues and commenting, let Claude Code Web handle branches

The skill is designed to be flexible - use as much or as little as you need!
