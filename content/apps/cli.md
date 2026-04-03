---
id: app-cli
type: app
title: "CLI App"
---

# CLI App

The CLI is where most people meet Amplifier for the first time. You open a terminal, type `amplifier`, and start a conversation with an AI agent that can read your code, run your tests, search the web, and edit your files. It feels simple — and that's the point. Underneath, the CLI is composing bundles, loading modules, managing sessions, and coordinating tool calls through the kernel. But you never have to think about any of that unless you want to.

`amplifier-app-cli` is the reference application. It's the most complete, the most tested, and the first to receive new features. If you're building on Amplifier, the CLI is both your daily driver and your blueprint for how applications work.

## What You'll Learn

By the end of this page, you'll understand:

- How to start interactive and one-shot sessions
- The core commands and what they do
- How session persistence and resumption work
- How configuration flows from settings.yaml and AGENTS.md
- The CommandProcessor architecture that powers the CLI
- A typical development workflow using the CLI

## Prerequisites

You should have [Amplifier installed](../quickstart/installation.md) and have read the [Apps Overview](./index.md). Familiarity with [bundles](../concepts/bundles.md) helps but isn't required.

---

## Getting Started

After installation, the CLI gives you two primary ways in:

**Interactive mode** — start a conversation that continues until you end it:

```
$ amplifier
Amplifier v0.9 | bundle: default | provider: anthropic

>
```

That `>` prompt is your entry point. Type anything — a question, an instruction, a task — and the agent responds, using whatever tools and modules your bundle provides.

**One-shot mode** — run a single prompt and exit:

```
$ amplifier run "How many Python files are in this project?"
```

```
[Tool: bash] find . -name "*.py" -not -path "./.venv/*" | wc -l
47

There are 47 Python files in this project, excluding the virtual environment.
```

One-shot mode is perfect for scripting, CI pipelines, or quick questions where you don't need a back-and-forth conversation.

## Core Commands

The CLI organizes its functionality into a small set of commands:

### `amplifier` (interactive)

Launches an interactive session. This is the command you'll use most often during development. The agent persists context across turns, remembers what files it has read, and builds on previous tool calls.

> Find all the TODO comments in the auth module and suggest fixes

```
[Tool: grep] pattern="TODO" path="src/auth/"
[Tool: read_file] src/auth/session.py
[Tool: read_file] src/auth/tokens.py

I found 4 TODOs across 2 files. Here's what each needs...
```

### `amplifier run "<prompt>"`

Executes a single prompt without entering interactive mode. The session is still created and persisted — you can resume it later if needed.

```
$ amplifier run "List all exported functions in src/api/"
```

### `amplifier resume`

Picks up where a previous session left off. Every session is automatically saved, so you can close your terminal, reboot your machine, and come back to exactly where you were.

```
$ amplifier resume
Resuming session abc123... (last active 2 hours ago)

> Continue refactoring the auth module
```

The agent retains full context from the previous session — files it read, tools it called, decisions it made.

### `amplifier bundle`

Manages the bundles available to the CLI:

```
$ amplifier bundle list
  default          Active bundle with standard tools
  python-dev       Python development with LSP, testing, linting
  design-intel     Architecture analysis and diagramming

$ amplifier bundle use python-dev
Switched to bundle: python-dev

$ amplifier bundle add my-custom-bundle ./path/to/bundle
Added bundle: my-custom-bundle
```

Switching bundles changes which modules are loaded — different tools, different providers, different orchestration strategies. The kernel doesn't care; it just executes whatever the active bundle specifies.

### `amplifier update`

Updates Amplifier and its bundles to the latest versions.

### `amplifier reset`

Clears the current session state, giving you a fresh start without restarting the CLI.

## Session Persistence

Every conversation in Amplifier is a **session**, and every session is automatically persisted. Work is never lost.

Sessions are stored on disk and include the full conversation history, tool call records, module state, and the active bundle configuration. When you run `amplifier resume`, the CLI rehydrates the session through the kernel. The agent picks up with full context, as if you'd never left.

> I was refactoring the database layer yesterday. Where did I leave off?

```
Looking at the session history, you were migrating the connection
pool from sync to async. You'd completed the Pool class and were
about to update the QueryExecutor.
```

Sessions also enable **branching** — resume the same session multiple times to explore different approaches, each getting its own session ID.

## Configuration

The CLI reads configuration from two sources:

### `settings.yaml`

Located at `~/.amplifier/settings.yaml`, this file controls global behavior:

```yaml
provider:
  name: anthropic
  model: claude-sonnet-4-20250514

bundles:
  active: default
  paths:
    - ~/.amplifier/bundles/
    - ./bundles/

session:
  persist: true
  directory: ~/.amplifier/sessions/
```

Settings apply to every session. Change the provider here, and every new conversation uses the new model. Change the bundle path, and the CLI discovers new bundles on next launch.

### `AGENTS.md`

The `AGENTS.md` file in your project root gives the agent a persona — domain knowledge, coding conventions, and behavioral guidelines specific to your project:

```markdown
# Project Agent

You are working on a Python web application using FastAPI.
Always use async/await patterns. Run pytest after any code change.
Follow the Google Python Style Guide.
```

When the CLI starts, it loads `AGENTS.md` from the current directory (if present) and injects it into the system prompt. This means the agent behaves differently depending on which project you're in — without changing any configuration.

## The CommandProcessor Architecture

Under the hood, the CLI is organized around a **CommandProcessor** — a pattern that separates input parsing from execution:

```
User Input → Parser → CommandProcessor → Foundation → Kernel
                                              ↓
                                        Bundle Resolution
                                        Module Loading
                                        Session Management
```

The Parser turns raw input into structured commands. The CommandProcessor dispatches those commands — some go to the foundation (prompts, session management), others are handled locally (help text, configuration changes). It also manages the interactive loop:

1. Display the `>` prompt
2. Read user input
3. Parse into command or prompt
4. If command: execute and loop back to 1
5. If prompt: send to kernel, stream response, loop back to 1

This is why the CLI feels responsive — the loop is tight, and streaming starts as soon as the first token arrives from the provider.

## A Typical Workflow

Here's what a real development session looks like:

```
$ cd ~/projects/my-api
$ amplifier
Amplifier v0.9 | bundle: python-dev | provider: anthropic
```

> I need to add rate limiting to the /users endpoint

```
[Tool: read_file] src/api/users.py
[Tool: grep] pattern="rate.?limit" path="src/"
[Tool: read_file] requirements.txt

I see the /users endpoint in src/api/users.py. There's no existing
rate limiting. I'll add it using slowapi, which integrates with
FastAPI. Let me install it and implement the limiter...

[Tool: bash] pip install slowapi
[Tool: edit_file] src/api/users.py — adding rate limiter decorator
[Tool: edit_file] src/api/app.py — registering the limiter middleware
[Tool: bash] pytest tests/test_users.py -v
===== 12 passed in 2.1s =====

Done. Rate limiting is now active on /users — 10 requests per minute
per client IP. All existing tests pass.
```

> Now add a test for the rate limit and commit everything

```
[Tool: edit_file] tests/test_users.py — adding test_rate_limit_exceeded
[Tool: bash] pytest tests/test_users.py -v
===== 13 passed in 2.3s =====

[Tool: bash] git add -A && git commit -m "feat(api): add rate limiting to /users endpoint"
[main abc1234] 3 files changed, 47 insertions(+), 2 deletions(-)
```

From exploration to implementation to testing to commit — all in one continuous session.

## What's Next?

- **[Chat App](./chat.md)** — the web-based alternative to the CLI
- **[Apps Overview](./index.md)** — compare applications and learn about building your own
- **[Key Commands](../quickstart/key-commands.md)** — a quick reference for CLI commands
