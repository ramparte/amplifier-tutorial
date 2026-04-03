---
id: app-chat
type: app
title: "Chat App"
---

# Chat App

Not everyone lives in the terminal. Some developers prefer a visual interface — one where conversation history is rendered cleanly, responses stream in real time, and you can scroll back through a session without piping output through `less`. That's what `amplifier-app-chat` provides: a web-based chat interface that runs the full Amplifier stack in your browser.

The important thing to understand is what the Chat app *is not*. It's not a different Amplifier. It's the same kernel, the same foundation library, the same modules, the same bundles — wrapped in a web UI instead of a terminal. A tool that works in the CLI works identically in Chat. A bundle you configure for terminal use works in the browser without changes. The application layer is thin, and that's what makes this possible.

## What You'll Learn

By the end of this page, you'll understand:

- What the Chat app provides and how it relates to the CLI
- How to install and run it
- How real-time streaming works in the browser
- How session management works
- Where the Chat app shines compared to the CLI
- How to configure it for your workflow

## Prerequisites

You should have read the [Apps Overview](./index.md) and be familiar with [the architecture](../concepts/architecture.md). Installation requires Python 3.10+ and a working Amplifier setup.

---

## What Is the Chat App?

The Chat app is a web application that serves an Amplifier session through your browser. When you start it, a local server launches, opens a WebSocket connection to the kernel, and renders a chat interface where you can type prompts and see streamed responses.

Think of it as a visual skin over the same engine:

```
+-----------------------------+
|  Browser (React frontend)   |
|  - Chat input               |
|  - Conversation view        |
|  - Session sidebar          |
+-------------+---------------+
              | WebSocket
+-------------v---------------+
|  Chat Server (Python)       |
|  - WebSocket handler        |
|  - Event stream relay       |
+-------------+---------------+
              |
+-------------v---------------+
|  amplifier-foundation       |
+-------------+---------------+
              |
+-------------v---------------+
|  amplifier-core (kernel)    |
+-----------------------------+
```

The browser never talks to the kernel directly. The Chat server acts as a bridge — receiving prompts over WebSocket, forwarding them to the foundation, and relaying kernel events back to the frontend as they arrive.

## Getting Started

### Installation

```
$ pip install amplifier-app-chat
```

This installs the Chat server and its frontend assets. It expects `amplifier-foundation` and `amplifier-core` to already be installed — if you have the CLI working, you have everything you need.

### Running

```
$ amplifier-chat
Starting Amplifier Chat on http://localhost:8420
```

Open `http://localhost:8420` in your browser. You'll see a clean chat interface with a text input at the bottom and a conversation area above it. Start typing — it works just like the CLI, but visual.

You can also specify a port and host:

```
$ amplifier-chat --port 9000 --host 0.0.0.0
```

Binding to `0.0.0.0` makes the Chat app accessible from other machines on your network — useful for team demos or pair programming.

## Real-Time Streaming

One of the Chat app's strongest features is how it handles streaming. When you send a prompt, the response doesn't arrive all at once — it streams token by token, just like the CLI. But in the browser, this streaming is rendered as live-updating markdown.

> Explain the authentication flow in this codebase

The response begins appearing immediately. Code blocks syntax-highlight as they render. Tool calls show up inline with their results:

```
[Tool: grep] pattern="def authenticate" path="src/"
Found 3 matches...

[Tool: read_file] src/auth/flow.py
Reading 84 lines...
```

Each tool invocation is visually distinct from the narrative response, making it easy to follow what the agent is doing and why. When the agent reads a file, you see it. When it runs a command, you see the output. The whole process is transparent.

Behind the scenes, the Chat server subscribes to kernel events — the same events the CLI consumes — and serializes them over the WebSocket. The frontend interprets each event type (text, tool_call, tool_result, error) and renders it appropriately. This is the same event stream, just displayed differently.

## Session Management

Sessions in the Chat app work like sessions in the CLI — they're created by the kernel, persisted automatically, and resumable. The difference is in how you navigate them.

The Chat interface includes a **session sidebar** that lists your recent sessions:

```
Sessions
─────────────────────
● Current session
  "Rate limiting implementation"
  Active — 12 turns

  "Database migration"
  2 hours ago — 8 turns

  "API documentation"
  Yesterday — 23 turns
```

Click any previous session to load it into the conversation view. The full history appears — every prompt, every response, every tool call — rendered as a scrollable conversation. You can read back through it, then continue the conversation by typing a new prompt.

Sessions created in the Chat app use the same persistence format as the CLI. This means you can start a session in the CLI, then view and continue it in Chat, or vice versa. The session belongs to the kernel, not to the application.

## How It Differs from the CLI

The Chat app and the CLI do the same thing differently. Here's where each shines:

| Dimension | CLI | Chat |
|-----------|-----|------|
| **Navigation** | Scrollback in terminal | Full scroll, sidebar, clickable history |
| **Output rendering** | Plain text, ANSI colors | Rendered markdown, syntax highlighting |
| **Tool visibility** | Inline with text stream | Visually separated, collapsible |
| **Multi-session** | One session per terminal | Session sidebar, easy switching |
| **Input** | Single-line prompt (multi-line with `\`) | Full text area, multi-line by default |
| **Sharing** | Copy-paste terminal output | Share URL, screenshot-ready |
| **Automation** | Scriptable, pipe-friendly | Not scriptable — interactive only |
| **Speed** | Fastest startup, lowest overhead | Slightly more overhead (server + browser) |

The Chat app excels at **review and collaboration**. When you want to show someone what an agent did — the full reasoning chain, the tool calls, the code changes — a rendered conversation is clearer than terminal scrollback. For demos, pair programming, and team reviews, Chat is the better choice.

The CLI excels at **speed and integration**. It starts instantly, works over SSH, chains with other shell tools, and supports one-shot scripting. For daily development, automation, and CI/CD, the CLI is the better choice.

Most developers use both — the CLI for working, Chat for reviewing and sharing.

## Configuration

The Chat app reads the same `settings.yaml` and `AGENTS.md` files as the CLI. Any configuration you've set up — provider, model, bundle paths, session directory — applies automatically.

Chat-specific settings can be added under a `chat` key in settings.yaml:

```yaml
provider:
  name: anthropic
  model: claude-sonnet-4-20250514

chat:
  port: 8420
  host: localhost
  theme: dark
```

Bundle selection happens at startup. The Chat app loads the active bundle from your settings and uses it for all sessions. To switch bundles, update `settings.yaml` and restart the server.

## A Typical Chat Workflow

Here's what working in the Chat app looks like in practice:

**1. Start the server:**

```
$ cd ~/projects/my-api
$ amplifier-chat
Starting Amplifier Chat on http://localhost:8420
```

**2. Open the browser and type a prompt:**

> I need to understand the payment processing flow before I make changes

**3. The agent responds with streaming output:**

```
[Tool: grep] pattern="payment|charge|invoice" path="src/"
[Tool: read_file] src/payments/processor.py
[Tool: read_file] src/payments/models.py
[Tool: read_file] src/payments/webhooks.py

Here's how payment processing works in this codebase:

1. **Initiation** — `PaymentProcessor.charge()` in processor.py
   creates a pending Payment record and sends it to Stripe...

2. **Webhook handling** — webhooks.py listens for Stripe events...

3. **Settlement** — on successful charge, the Payment status
   updates to "completed" and an invoice is generated...
```

**4. Continue the conversation:**

> Now add support for refunds following the same pattern

The agent has full context from the exploration phase. It knows the code structure, the patterns, the naming conventions — and it builds on that knowledge to implement the feature.

**5. Review the session later:**

Click back to this session from the sidebar at any time. The full conversation is there — the exploration, the implementation, the reasoning. Share the URL with a teammate for review.

## What's Next?

- **[CLI App](./cli.md)** — the terminal-based alternative
- **[Apps Overview](./index.md)** — compare applications and learn about building your own
- **[Architecture](../concepts/architecture.md)** — understand the full stack that powers both apps
