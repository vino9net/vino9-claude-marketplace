# Claude Code Skills Marketplace

A collection of reusable plugins for AI coding tools.

## TL;DR

Add this to your `~/.claude/settings.json` (or project `.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "vino9": {
      "source": {
        "source": "git",
        "url": "https://github.com/vino9net/vino9-claude-marketplace.git"
      }
    }
  },
  "enabledPlugins": {
    "boot-gcp-vm@vino9": true,
    "github-issues@vino9": true,
    "python-dev@vino9": true
  }
}
```

## Available Plugins

| Plugin | Source | Description |
|--------|--------|-------------|
| [boot-gcp-vm](plugins/boot-gcp-vm/) | local | Start a GCP VM and update local SSH config with its new IP address |
| [github-issues](plugins/github-issues/) | local | Read and comment on GitHub issues in Claude Code Web (where gh CLI doesn't work) |
| [python-dev](https://github.com/vino9net/claude-python-skill) | remote | Python project scaffolding, quality standards (ruff, ty), and test runner |

Plugins are registered in [`plugins.json`](plugins.json). Local plugins live in `plugins/`, remote plugins are referenced by GitHub repo.

## Contributing

To add a plugin:
1. **Local**: create `plugins/<name>/SKILL.md` and add to `plugins.json`
2. **Remote**: just add a `plugins.json` entry pointing to the GitHub repo

## License
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)
