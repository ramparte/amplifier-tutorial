---
id: apps-overview
type: app
title: "Apps Overview"
---

# Apps Overview

You've seen the kernel, the modules, and the bundles. Now we arrive at the top of the stack: **applications**. An application is the layer you actually interact with — it takes the entire Amplifier architecture and presents it as a usable experience. When you type `amplifier run` in your terminal or open a chat window in your browser, you're talking to an application.

Applications don't implement AI logic themselves. They compose bundles and modules into user-facing interfaces, then get out of the way. The same kernel, the same bundles, the same tools — they all work identically regardless of which application wraps them. This is by design: the application layer is thin, opinionated about UX, and completely replaceable.

## Section Contents

| Page | Description |
|------|-------------|
| [CLI App](./cli.md) | The reference command-line interface |
| [Chat App](./chat.md) | Web-based visual interface for browser workflows |
| [Building Apps](./building-apps.md) | How to build your own application on amplifier-foundation |
| [MCP Integration](./mcp.md) | Model Context Protocol server and client integration |

## What You'll Learn

By the end of this page, you'll understand:

- Where applications sit in the Amplifier architecture
- What official applications are available today
- How community applications extend the ecosystem
- How to choose the right application for your workflow
- How to build your own application using the foundation library

## Prerequisites

You should understand [the architecture](../concepts/architecture.md) and [bundles](../concepts/bundles.md). No installation is needed to read this page, but you'll want [Amplifier installed](../quickstart/installation.md) to try things.

---

## Applications in the Architecture

Recall the architecture stack from the concepts section:

```
+------------------------------------------------------------------+
|  APPLICATIONS        <-- you are here                            |
|  amplifier-app-cli, amplifier-app-chat, amplifierd               |
+----------------------------+-------------------------------------+
                             |
+----------------------------v-------------------------------------+
|  LIBRARIES (Foundation)                                          |
|  amplifier-foundation: bundles, module resolution, utilities     |
+----------------------------+-------------------------------------+
                             |
+----------------------------v-------------------------------------+
|  KERNEL (amplifier-core)                    ~2,600 lines of Rust |
+----------------------------+-------------------------------------+
                             |
              Providers, Tools, Orchestrators, Hooks, Context
```

Applications sit at the very top. They depend on `amplifier-foundation`, which depends on `amplifier-core`. Modules — the providers, tools, orchestrators, hooks, and context managers — only depend on the kernel. This means every module works identically in every application. A tool you write doesn't need to know whether it's running inside the CLI or a web browser.

## The Official Applications

Amplifier ships with two official applications, each designed for a different workflow:

### CLI (`amplifier-app-cli`)

The command-line interface is the reference implementation and the primary way most people use Amplifier. It runs in your terminal, supports both interactive conversations and one-shot commands, and manages sessions, bundles, and configuration through a clean command structure.

> Help me refactor this authentication module

```
[Tool: read_file] src/auth.py
[Tool: grep] pattern="def authenticate" path="src/"
[Tool: bash] pytest tests/test_auth.py

I've analyzed the authentication module. Here are the changes I'd recommend...
```

The CLI is where Amplifier development happens fastest. It's the most mature application, has the richest feature set, and is the first to receive new capabilities. If you're unsure where to start, start here.

**Read more:** [CLI App](./cli.md)

### Chat (`amplifier-app-chat`)

The web-based chat interface provides a visual, browser-based experience. It uses the same foundation library and the same modules as the CLI — the difference is entirely in presentation. Streaming responses appear in a conversation view, sessions persist in the browser, and you navigate with mouse and keyboard together.

The Chat app is ideal for teams that prefer a visual interface, for demonstrations, or for workflows where you want to see a full conversation history rendered cleanly.

**Read more:** [Chat App](./chat.md)

## Quick Comparison

| Feature | CLI | Chat |
|---------|-----|------|
| Interface | Terminal | Web browser |
| Installation | `pip install amplifier-app-cli` | `pip install amplifier-app-chat` |
| Interaction style | Keyboard-driven, prompt-based | Visual, mouse + keyboard |
| Session management | File-based, resumable | Browser-based, persistent |
| Streaming | Line-by-line terminal output | Real-time rendered markdown |
| One-shot mode | Yes (`amplifier run "..."`) | No — conversational only |
| Bundle management | Built-in commands | Configured externally |
| Best for | Daily development, automation | Teams, demos, visual review |

Both applications use the same kernel, the same modules, and the same bundles. A session started in the CLI can be reasoned about in the Chat app. Configuration in `settings.yaml` applies to both.

## Community Applications

The Amplifier ecosystem is open, and the application layer is designed to be extended. Community applications use the same `amplifier-foundation` library to build experiences the official apps don't cover:

- **IDE integrations** — embed Amplifier inside VS Code, JetBrains, or other editors
- **Slack and Teams bots** — bring Amplifier into team chat
- **API servers** — expose Amplifier as a service behind REST or GraphQL endpoints
- **Voice interfaces** — control Amplifier with speech
- **Custom dashboards** — build domain-specific UIs for particular workflows

If someone has published a community application, you install it and point it at your existing bundles. No changes to your modules or configuration are needed — the application layer is additive.

## Building Your Own Application

Every official and community application is built on top of `amplifier-foundation`. This library provides the machinery you need:

- **Bundle resolution** — discover and load bundles from the filesystem
- **Module discovery** — resolve modules from bundles into mount plans
- **Session management** — create, persist, and resume sessions via the kernel
- **Event streaming** — subscribe to kernel events for real-time output
- **Configuration** — load settings from `settings.yaml` and AGENTS.md

Here's the minimal shape of an application:

```python
from amplifier_foundation import Foundation

# Initialize the foundation with your configuration
foundation = Foundation.from_settings("~/.amplifier/settings.yaml")

# Load bundles and build a mount plan
plan = foundation.resolve_bundles()

# Create a session
session = foundation.create_session(plan)

# Send a prompt and stream the response
for event in session.prompt("Hello, Amplifier"):
    print(event)
```

That's the skeleton. A real application adds input handling, output formatting, error recovery, and whatever UX opinions make sense for your use case. The CLI adds command parsing and terminal rendering. The Chat app adds WebSocket streaming and a React frontend. Your application adds whatever it needs.

The key insight is that the foundation does the heavy lifting. Your application is free to focus entirely on the user experience.

## What's Next?

Dive into the specific applications:

- **[CLI App](./cli.md)** — the reference command-line interface, and the best place to start
- **[Chat App](./chat.md)** — the web-based visual interface
- **[Building Apps](./building-apps.md)** — build your own application on amplifier-foundation
- **[MCP Integration](./mcp.md)** — Model Context Protocol server and client integration

Or go back to the concepts:

- **[Architecture](../concepts/architecture.md)** — understand the full stack
- **[Bundles](../concepts/bundles.md)** — learn how modules are composed
- **[Modules](../concepts/modules.md)** — understand the five module types
