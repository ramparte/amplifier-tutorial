---
id: dev-setup-index
type: section-index
title: Development Setup
---

# Development Setup

Set up an optimal development environment for working with Amplifier. This section covers IDE configuration, debugging tools, testing workflows, and developer productivity tips. Get your local environment tuned for efficient Amplifier development.

## Section Contents

| Page | Description |
|------|-------------|
| [CLI Tools](./cli-tools.md) | Essential command-line utilities |
| [Shadow Workspace](./shadow-workspace.md) | Isolated testing environments |
| [Remote Development](./remote-dev.md) | Working with remote systems |
| [Debugging](./debugging.md) | Troubleshooting and debugging workflows |

## Quick Tips

- **Use virtual environments** - Isolate Amplifier dependencies
- **Enable LSP** - Get code intelligence in your editor
- **Set up pre-commit** - Catch issues before committing
- **Use session logs** - Debug with full conversation traces
- **Test incrementally** - Run focused tests during development

## Environment Checklist

Before starting development:

- [ ] Python 3.10+ installed
- [ ] Virtual environment created
- [ ] Amplifier installed in editable mode
- [ ] Editor/IDE configured with Python support
- [ ] LSP server (Pyright) available
- [ ] Git configured with hooks
- [ ] API keys in environment variables

## Recommended Stack

| Tool | Purpose |
|------|---------|
| `uv` | Fast Python package manager |
| `pyright` | Type checking and LSP |
| `pytest` | Test framework |
| `ruff` | Linting and formatting |
| VS Code | Editor with Python extension |

## Where to Start

**Setting up fresh?** Begin with [CLI Tools](./cli-tools.md) for essential command-line utilities.

**Testing locally?** Jump to [Shadow Workspace](./shadow-workspace.md) for isolated testing environments.

**Working remotely?** See [Remote Development](./remote-dev.md) for remote system workflows.

## Development Commands

```bash
# Install in development mode
uv pip install -e ".[dev]"

# Run tests
pytest tests/

# Type check
pyright src/

# Format code
ruff format src/

# Lint
ruff check src/ --fix
```

## Debugging Workflow

1. **Check session logs** - `~/.amplifier/sessions/`
2. **Use session-analyst** - Delegate to specialist agent
3. **Enable verbose mode** - More detailed output
4. **Inspect events.jsonl** - Raw conversation data

## Next Steps

After setting up your environment, start with [Quickstart](../quickstart/index.md) to build your first application, or explore [Tools](../tools/index.md) to understand available capabilities.
