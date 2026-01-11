---
id: dev-setup-index
type: section-index
title: Development Setup
---

# Development Setup

This section covers setting up a development environment for working on Amplifier itself or building applications that extend it. Follow these guides to configure a productive local setup.

Whether you're contributing to the core project or building custom modules, proper setup ensures a smooth experience.

## Section Contents

| Page | Description |
|------|-------------|
| [Environment Setup](./environment.md) | Python, dependencies, and tooling |
| [IDE Configuration](./ide-config.md) | VS Code, PyCharm, and editor setup |
| [Local Development](./local-dev.md) | Running Amplifier from source |
| [Testing](./testing.md) | Running and writing tests |
| [Debugging](./debugging.md) | Debug techniques and tools |
| [Contributing](./contributing.md) | How to contribute to Amplifier |
| [Code Style](./code-style.md) | Conventions and formatting |
| [Release Process](./release.md) | Versioning and publishing |

## Quick Tips

- **Use uv** — It's faster than pip and handles dependencies better
- **Virtual environments** — Always isolate project dependencies
- **Pre-commit hooks** — Enable them to catch issues before commits
- **Type checking** — Run pyright/mypy regularly during development
- **Test early** — Write tests as you develop, not after

## System Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| Python | 3.11+ | 3.12 recommended |
| uv or pip | Latest | uv preferred |
| Git | 2.30+ | For version control |
| Node.js | 18+ | For some tooling (optional) |

## Quick Start

```bash
# Clone the repository
git clone https://github.com/microsoft/amplifier.git
cd amplifier

# Create virtual environment
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows

# Install in development mode
uv pip install -e ".[dev]"

# Run tests to verify setup
pytest

# Start development session
amp --dev
```

## Where to Start

**Setting up for the first time?** Follow [Environment Setup](./environment.md) for complete installation instructions.

**Contributing to Amplifier?** Read [Contributing](./contributing.md) for workflow and guidelines.

**Debugging issues?** Check [Debugging](./debugging.md) for techniques and common problems.

## Development Workflow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Feature    │───►│    Tests     │───►│     PR       │
│   Branch     │    │   Passing    │    │   Review     │
└──────────────┘    └──────────────┘    └──────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
   Write Code        Run pytest          Address Feedback
   Type Check        Check Coverage      Merge to Main
```

## Recommended Tools

| Tool | Purpose | Install |
|------|---------|---------|
| uv | Package management | `pip install uv` |
| ruff | Linting and formatting | `pip install ruff` |
| pyright | Type checking | `pip install pyright` |
| pytest | Testing | Included in dev deps |

## Common Tasks

```bash
# Format code
ruff format .

# Check types
pyright

# Run specific tests
pytest tests/test_sessions.py -v

# Run with coverage
pytest --cov=amplifier
```

## Related Sections

- [Quickstart: Installation](../quickstart/installation.md)
- [Advanced: Custom Tools](../advanced/custom-tools.md)
- [Concepts: Module System](../concepts/modules.md)
