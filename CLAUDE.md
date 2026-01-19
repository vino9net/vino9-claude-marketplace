# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains reusable Claude Code skills following the **SKILL.md open standard** (https://agentskills.io/). These skills are designed to be shared across projects as Git submodules and work with Claude Code, Cursor, GitHub Copilot, Gemini CLI, and other AI coding tools.

## Repository Architecture

### Structure

```
.
├── python-dev/         # Python development standards (ruff, ty, uv)
│   └── SKILL.md
├── google-cloud/       # GCP workflows and best practices
│   ├── SKILL.md        # Main GCP skill
│   ├── cloud_run/      # Cloud Run specific operations
│   │   └── SKILL.md
│   └── compute_engine/ # Compute Engine VM management
│       └── SKILL.md
├── github-issues/      # GitHub Issues workflow for restricted environments
│   ├── SKILL.md
│   └── gh_issue.py     # Python script for GitHub API interaction
└── settings.json       # Claude Code permissions configuration
```

### Skills Organization

**python-dev**: Language-specific development standards
- Modern Python tooling (ruff for linting/formatting, ty for type checking, uv for package management)
- Python 3.10+ type annotations
- Pre-commit quality gates workflow

**google-cloud**: Cloud platform operations
- Main skill provides general GCP guidance
- Sub-skills in subdirectories provide service-specific operations
- Includes MCP server setup instructions for programmatic GCP access

**github-issues**: GitHub Issues workflow for restricted environments
- Designed for Claude Code Web where GH CLI is unavailable
- Python script (stdlib only) for reading issues and adding comments
- Works with Claude Code Web's automatic branch creation
- Authentication via `gh auth token` or `GITHUB_TOKEN` env var

### SKILL.md Format

Each skill follows the SKILL.md standard with:
- **Frontmatter**: `name`, `description`, `allowed-tools` in YAML format
- **Content**: Detailed instructions, code examples, common patterns
- **Workflows**: Step-by-step procedures for common tasks

## Working with This Repository

### Adding New Skills

When creating a new skill:

1. Choose appropriate directory structure:
   - Language/framework skills: `<language>-<framework>/SKILL.md`
   - Cloud platform skills: `<platform>/<service>/SKILL.md`

2. Follow SKILL.md standard format:
   ```markdown
   ---
   name: skill-name
   description: Brief description for when this skill should be used
   allowed-tools: Tool1, Tool2
   ---

   # Skill Title
   [Content following standard structure]
   ```

3. Include in skill documentation:
   - Clear "When to Use" guidance
   - Command examples with all required parameters
   - Common workflows and patterns
   - Integration with project CLAUDE.md files
   - Troubleshooting section

4. Test skill works standalone and integrated into projects

### Version Control

Skills use Git tags for versioning:
```bash
# Check current version
git describe --tags

# Tag new version after changes
git tag -a v1.x.x -m "Description of changes"
git push && git push --tags
```

### Usage as Submodule

Projects consume these skills as Git submodules:

```bash
# Add to project
git submodule add https://github.com/vino9net/claude-shared-skills.git .claude/skills/shared

# Update to latest
git submodule update --remote .claude/skills/shared
```

### Permissions Configuration

The `settings.json` at repository root configures Claude Code permissions:
- Allows common commands (git, uv, ls, cp, mv, grep, etc.)
- Requires confirmation for git commits/pushes
- Protects .env files with ask permission
- Sets `defaultMode: "acceptEdits"` for smoother workflow

## Key Design Principles

1. **Self-Contained**: Each skill should work standalone without requiring other skills
2. **Integration Points**: Skills should reference project CLAUDE.md for project-specific configuration
3. **Clear Triggers**: Description field clearly states when to use the skill
4. **Tool Restrictions**: `allowed-tools` field limits what the skill can execute
5. **Two-Tier Guidance**: SKILL.md for general standards, project CLAUDE.md for specifics

## Skill Integration Pattern

Skills follow this integration pattern with projects:

1. **Skill provides standards** - General best practices and tooling
2. **Project CLAUDE.md provides context** - Specific Python version, GCP project ID, domain conventions
3. **Skill reads project config** - Checks .python-version, pyproject.toml, CLAUDE.md for project-specific settings
4. **Skill adapts behavior** - Uses project configuration to apply standards appropriately

Example from python-dev skill:
- Skill defines when to run ruff/ty checks (before commits)
- Skill reads .python-version for Python version
- Skill checks pyproject.toml for line length config
- Skill defers to project CLAUDE.md for domain-specific conventions

## Working with Skills in Projects

When using these skills in a project (as a submodule):

1. The skill files are read-only reference material
2. Projects should create their own CLAUDE.md with project-specific info:
   - Python version (or reference to .python-version)
   - GCP project ID and region
   - Service names and endpoints
   - Project-specific conventions
   - Architecture decisions

3. Skills automatically check for project CLAUDE.md and integrate with it

## License

Apache License 2.0
