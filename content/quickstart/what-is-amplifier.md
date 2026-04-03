---
id: what-is-amplifier
type: quickstart
title: "What is Amplifier?"
---

# What is Amplifier?

You've probably used AI coding assistants before. Maybe GitHub Copilot suggests completions as you type, or you've tried Cursor for AI-powered editing. Those tools are useful, but they're also closed boxes. You can't see how they work, you can't change how they behave, and you certainly can't swap out the AI model underneath.

Amplifier is different. It's a **modular AI agent framework** — an open-source platform where every piece is visible, replaceable, and composable. The CLI you interact with is just one interface. Underneath it is a small, stable kernel surrounded by modules you can mix, match, and extend to build exactly the AI experience you need.

## What You'll Learn

By the end of this page, you'll understand:

- What Amplifier is and why it exists
- The kernel architecture and the five module types
- How bundles compose modules into ready-to-use configurations
- How agents give the AI specialized personas for different tasks
- How Amplifier compares to other AI development tools
- What your first interaction looks like

## Prerequisites

None. This is a concepts page — no installation required. If you want to follow along with the interactive examples, you'll need [Amplifier installed](./installation.md).

---

## The Linux Kernel Philosophy

Amplifier is built on a principle borrowed from the Linux kernel: **mechanism, not policy**.

The kernel provides *capabilities* without making *decisions*. It knows how to load modules, manage sessions, dispatch hooks, and coordinate events. But it doesn't decide which AI model to use, what tools to make available, or how conversations should be orchestrated. Those are all policy decisions — and they belong in modules, not the kernel.

Here's what that looks like in practice:

| The Kernel Provides (Mechanism) | Modules Decide (Policy) |
|--------------------------------|------------------------|
| Module loading | Which modules to load |
| Event emission | What to log, and where |
| Session lifecycle | Orchestration strategy |
| Hook registration | Security and filtering rules |

The test is simple: *"Could two teams want different behavior here?"* If yes, it's policy, and it belongs in a module.

This is why `amplifier-core` is only about 2,600 lines of Rust — with Python bindings via PyO3 so the whole ecosystem speaks Python. The kernel is small enough for one person to understand completely. Everything else lives in modules.

## The Five Module Types

Every capability in Amplifier is delivered through one of five module types. Think of them as the five kinds of building blocks you snap together:

**Provider** — The AI brain. Providers connect Amplifier to language models. Anthropic Claude, OpenAI GPT, Azure OpenAI, or a local model running on Ollama. Switch providers anytime without changing anything else.

**Tool** — Hands for the AI. Tools let the agent act on the world: reading files, running shell commands, searching the web, executing code. When you ask Amplifier to "find all TODO comments in my Python files," it's a Tool that does the searching.

**Orchestrator** — The execution loop. Orchestrators control how a conversation flows — how the agent receives a prompt, calls tools, and produces a response. Basic orchestrators run a simple loop; advanced ones handle streaming, parallel execution, or multi-step reasoning.

**Hook** — An interceptor. Hooks sit in the pipeline and can observe or transform what passes through. A logging hook records events. A redaction hook strips sensitive data before it reaches the AI. A content filter validates outputs before they reach you.

**Context** — Memory management. Context modules control what the agent remembers. They manage conversation history, decide when to compact long conversations, and can inject background knowledge into every interaction.

These five types compose into everything Amplifier can do. A fresh installation, a team's customized setup, and a complex multi-agent workflow are all just different combinations of the same five building blocks.

## Bundles: Composable Configuration

You don't need to pick modules one at a time. **Bundles** are pre-composed packages that group related modules together for a specific purpose.

When you run `amplifier init` and choose the `foundation` bundle, you get:

- **Tools**: filesystem access, bash execution, web search, code search, task delegation
- **Agents**: zen-architect, bug-hunter, modular-builder, explorer, researcher, and more
- **Hooks**: logging, redaction, streaming UI, session persistence

The `dev` bundle builds on `foundation` with additional development tools. The `recipes` bundle adds multi-step workflow support. You can install community bundles for specialized domains — design systems, data science, infrastructure automation.

Bundles compose by layering. You can extend one bundle with another, override specific modules, or create your own from scratch. The key idea is that **bundles define what you can do**, while **providers define where the AI comes from**. They're independent axes — use any bundle with any provider.

## Agents: Specialized Personas

Agents are AI personas tuned for specific tasks. They have focused system instructions, curated tool access, and domain expertise. When Amplifier delegates to an agent, it spawns a sub-session where that agent works independently and returns its results.

Here's what's available in the `foundation` bundle:

| Agent | What It Does |
|-------|-------------|
| **zen-architect** | System design with ruthless simplicity |
| **bug-hunter** | Systematic debugging — finds root causes, not just symptoms |
| **modular-builder** | Builds self-contained components from specifications |
| **explorer** | Maps codebases, surfaces key files, provides cited summaries |
| **researcher** | Synthesizes information from docs, web, and code |

You don't need to invoke agents directly. Ask Amplifier to "design a caching system" and it may delegate to zen-architect. Ask it to "debug this error" and bug-hunter gets the call. Or you can be explicit:

> Use bug-hunter to investigate the KeyError in auth.py

Behind the scenes, Amplifier forks a sub-session for the agent:

```
[Tool: task] Delegating to foundation:bug-hunter
  └─ "Investigate the KeyError in auth.py"
```

The agent works in isolation — its own context, its own tool access — then returns a structured result to the main session.

## Recipes: Multi-Step Workflows

Some tasks need more than a single conversation turn. **Recipes** are declarative YAML workflows that chain multiple steps together, with built-in checkpointing, error handling, and optional human approval gates.

Imagine a code review workflow: first an explorer maps the changed files, then a reviewer checks them against standards, then a summarizer produces the final report. Each step runs as its own agent session, with results flowing from one step to the next.

Recipes support:

- **Sequential execution** with state passing between steps
- **Approval gates** where a human reviews before the next step proceeds
- **Automatic checkpointing** so you can resume if something interrupts
- **Error handling and retries** for resilience

You'll learn more about recipes in the [Recipes concept page](../concepts/recipes.md). For now, just know they exist — they're how Amplifier handles complex, multi-step work that goes beyond a single prompt.

## What a First Interaction Looks Like

Here's what happens when you start Amplifier for the first time and ask it to do something real:

```
$ amplifier
```

Amplifier drops you into an interactive session. You type naturally:

> What files are in the current directory? Give me a quick overview of this project.

Behind the scenes, Amplifier uses tools to explore your workspace:

```
[Tool: glob] **/*.py — found 23 files
[Tool: read_file] README.md
[Tool: read_file] pyproject.toml
```

Then it responds with a structured summary of what it found — file counts by type, the project's purpose from the README, key entry points, dependencies. It's not guessing. It actually read your files.

Now you ask a follow-up:

> The authentication module looks complex. Can you map it out?

```
[Tool: task] Delegating to foundation:explorer
  └─ "Map the authentication module structure"
```

The explorer agent dives deep — reading files, tracing imports, building a mental map — and returns a cited summary. Your main session gets the result and presents it to you.

Everything is saved automatically. Close your terminal, come back tomorrow, and pick up where you left off:

```
$ amplifier continue
```

## How Amplifier Compares

If you've used other AI coding tools, here's where Amplifier sits:

| | GitHub Copilot | Cursor | Amplifier |
|---|---|---|---|
| **Architecture** | Closed, hosted | Closed, IDE-embedded | Open, modular kernel |
| **AI Model** | GitHub's choice | Their selection | You choose: Anthropic, OpenAI, Azure, Ollama, or local |
| **Extensibility** | Limited | Plugin system | Full module system — providers, tools, hooks, context |
| **Transparency** | Black box | Limited visibility | Every layer inspectable, every decision traceable |
| **Multi-agent** | No | No | First-class: delegation, parallel execution, recipes |
| **Customization** | Settings only | Some configuration | Create bundles, agents, tools, entire custom experiences |
| **License** | Proprietary | Proprietary | MIT open source |

Amplifier isn't trying to replace your IDE's autocomplete. It's a platform for building AI-powered development workflows — from simple Q&A to complex multi-agent pipelines — where you control every piece.

## Verify Your Understanding

At this point, you should be able to answer:

- **What is the kernel?** A minimal Rust core (~2,600 lines) that provides mechanisms (module loading, sessions, hooks, events) without policy decisions.
- **What are the five module types?** Provider, Tool, Orchestrator, Hook, Context.
- **What's a bundle?** A pre-composed package of modules for a specific use case.
- **What's an agent?** A specialized AI persona with focused instructions and curated tool access.
- **What's the "mechanism, not policy" principle?** The kernel provides capabilities; modules make decisions about how to use them.

## Try It Out

If you have Amplifier installed, try these:

**Start a conversation and explore:**

> What tools do you have available?

**Ask it to do real work in a project directory:**

> Read the README and summarize this project in three sentences.

**See agents in action:**

> Use zen-architect to evaluate the architecture of this codebase.

**Check what's loaded:**

Type `/tools` to see available tools, `/agents` to see available agents, and `/status` to see your current session.

## Common Issues

**"I asked a question but it didn't use any tools."**
Not every question needs tools. Factual questions, explanations, and code generation from description can happen without tool use. Amplifier reaches for tools when it needs to interact with your filesystem, run commands, or search the web.

**"It picked the wrong agent for my task."**
You can always be explicit: "Use bug-hunter to..." or "Delegate to zen-architect for..." The automatic delegation improves as you give clearer context about what you need.

**"I want to use a different AI model."**
Amplifier supports multiple providers. Run `amplifier provider use openai` or `amplifier provider use ollama` to switch. Your bundles and tools stay exactly the same.

## Next Steps

Now you know what Amplifier is. Here's where to go next:

1. **[Installation](./installation.md)** — Get Amplifier running on your machine
2. **[Your First Conversation](./first-conversation.md)** — Start using it for real work
3. **[How It's Different](./how-its-different.md)** — Deeper comparison with other approaches
4. **[Key Commands](./key-commands.md)** — Essential commands and workflows
5. **[Concepts: Architecture](../concepts/architecture.md)** — The full technical picture

---

**Welcome to Amplifier.** It's modular, it's open, and every piece is yours to inspect, extend, and compose. Let's get building.
