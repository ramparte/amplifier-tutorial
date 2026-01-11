---
id: quickstart-index
type: section-index
title: Quickstart Guide
---

# Quickstart Guide

Get up and running with Amplifier in minutes. This section walks you through installation, configuration, and building your first agent—no prior AI development experience required.

Follow these guides in order for the smoothest learning path, or jump to specific topics as needed.

## Section Contents

| Page | Description | Time |
|------|-------------|------|
| [Installation](./installation.md) | Install Amplifier and dependencies | 5 min |
| [Configuration](./configuration.md) | Set up API keys and preferences | 5 min |
| [First Agent](./first-agent.md) | Build and run your first agent | 10 min |
| [Using Tools](./using-tools.md) | Learn tool basics with hands-on examples | 15 min |
| [Adding Context](./adding-context.md) | Give agents knowledge with context files | 10 min |
| [Creating Skills](./creating-skills.md) | Package reusable domain knowledge | 15 min |
| [Next Steps](./next-steps.md) | Where to go from here | 5 min |

## Quick Tips

- **API keys required** — You'll need at least one LLM provider key (Anthropic recommended)
- **Python 3.11+** — Amplifier requires modern Python; check with `python --version`
- **Start simple** — The first-agent guide uses minimal configuration intentionally
- **Experiment freely** — Sessions are isolated; you can't break anything permanently
- **Use the help** — Run `amp --help` for CLI options at any time

## Prerequisites

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| Python | 3.11+ | `python --version` |
| pip or uv | Latest | `pip --version` or `uv --version` |
| API Key | Any LLM | Set in environment or config |

## Recommended Path

```
Installation ──► Configuration ──► First Agent
                                       │
                     ┌─────────────────┼─────────────────┐
                     ▼                 ▼                 ▼
              Using Tools      Adding Context    Creating Skills
                     │                 │                 │
                     └─────────────────┴─────────────────┘
                                       │
                                       ▼
                                  Next Steps
```

## Where to Start

**Brand new to Amplifier?** Start with [Installation](./installation.md) and follow the guides in order. The whole quickstart takes about an hour.

**Already installed?** Jump to [First Agent](./first-agent.md) to build something immediately.

**Experienced with AI tools?** Skim [Configuration](./configuration.md) for Amplifier-specific settings, then explore [Creating Skills](./creating-skills.md).

## Quick Commands

```bash
# Install Amplifier
pip install amplifier-cli

# Verify installation
amp --version

# Start interactive session
amp

# Run with specific bundle
amp --bundle foundation
```

## What You'll Build

By the end of this quickstart, you'll have:
- A working Amplifier installation
- Your first custom agent
- Understanding of tools, context, and skills
- Foundation for building complex AI applications

## Related Sections

- [Concepts: Core Architecture](../concepts/index.md)
- [Dev Setup: Development Environment](../dev-setup/index.md)
- [Bundles: Using Bundles](../bundles/index.md)
