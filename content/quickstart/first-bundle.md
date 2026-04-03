---
id: first-bundle
type: quickstart
title: "Your First Bundle"
---

# Your First Bundle

Bundles are how you tell Amplifier what it can do. A bundle is a composable configuration package that defines which tools, agents, hooks, and instructions are available in a session. Think of bundles as loadouts — you pick one, and it determines what capabilities your AI assistant has.

In this guide you'll use a built-in bundle, then create your own from scratch.

## What You'll Learn

- What bundles are and why they matter
- How to use existing bundles from the command line
- How to create a minimal custom bundle
- How to compose bundles together (the "thin bundle" pattern)
- How to set a default bundle so you don't have to specify one every time

## Prerequisites

- Amplifier installed and configured (see [Installation](./installation.md))
- A working provider and API key (`amplifier run "hello"` should respond)
- A text editor for creating bundle files

## Step 1: Use an Existing Bundle

You already have a bundle active — the setup wizard picked one during `amplifier init`. Check which one:

```bash
amplifier bundle current
```

> foundation (from global)

The `foundation` bundle is Amplifier's standard loadout. It gives you filesystem tools, bash, web search, code search, and the other core capabilities you've been using.

You can switch bundles for a single command without changing your default:

```bash
amplifier run --bundle recipes "What recipe templates are available?"
```

This runs one session with the `recipes` bundle (which adds multi-step workflow support), then your next session goes back to whatever default you had.

To switch your default persistently:

```bash
amplifier bundle use recipes
```

And to see everything available:

```bash
amplifier bundle list
```

### Key Bundles to Know

| Bundle | What It Adds |
|--------|-------------|
| **foundation** | Standard development tools — filesystem, bash, web search, code intelligence |
| **recipes** | Multi-step workflow engine for repeatable, declarative task automation |
| **superpowers** | TDD methodology, systematic debugging, and disciplined development agents |
| **python-dev** | Python-specific tooling — quality checks, virtual environment management |
| **dot-graph** | Graph visualization and structural analysis with Graphviz |
| **browser-tester** | Web automation — browser control, screenshots, form interaction |

Each bundle builds on foundation, so you get the core tools plus whatever the bundle specializes in.

## Step 2: Install a Bundle from Git

Some bundles aren't built in — they live in their own repositories. Install one with `bundle add`:

```bash
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
amplifier bundle use recipes
```

The first command downloads the bundle and registers it locally. The second makes it your active bundle. Now when you start a session, you'll have recipe capabilities available alongside all the foundation tools.

## Step 3: Create Your Own Bundle

Here's where it gets interesting. Let's build a custom bundle from scratch.

Create a directory for it:

```bash
mkdir -p ~/.amplifier/bundles/my-tools
cd ~/.amplifier/bundles/my-tools
```

Now create the bundle definition. Bundles are markdown files with YAML frontmatter — the YAML declares configuration, and the markdown body becomes the system instruction. Create `bundle.md`:

```markdown
---
bundle:
  name: my-tools
  version: 1.0.0
  description: My personal development toolkit

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
---

# My Tools

You are a helpful development assistant with my preferred defaults.

When writing code, always:
- Use type hints for all function signatures
- Add docstrings to public functions
- Prefer explicit error handling over bare except clauses

When explaining code, be concise. Lead with what the code does,
then cover how it works.
```

That's a working bundle. The `includes` line pulls in foundation (so you keep all the standard tools), and the markdown body customizes the system prompt with your preferences.

Try it:

```bash
amplifier run --bundle ~/.amplifier/bundles/my-tools/bundle.md "Write a function that parses CSV files"
```

The response should follow your instructions — type hints, docstrings, proper error handling. The assistant is using foundation's tools but your custom instructions.

## Step 4: Add an Agent

Agents are specialized sub-assistants your bundle provides. Let's add one.

Create the agents directory and an agent file:

```bash
mkdir -p agents
```

Create `agents/quick-reviewer.md`:

```markdown
---
meta:
  name: quick-reviewer
  description: "Fast code review — checks style, types, and common mistakes."
---

# Quick Reviewer

You are a focused code reviewer. When given code to review:

1. Run python_check to catch formatting and type issues
2. Read the code and check for common mistakes
3. Provide feedback organized by severity (critical, warning, suggestion)

Be direct. Skip praise — focus on what needs to change and why.
```

Now update your `bundle.md` to register this agent:

```markdown
---
bundle:
  name: my-tools
  version: 1.0.0
  description: My personal development toolkit

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main

agents:
  include:
    - my-tools:quick-reviewer
---

# My Tools

You are a helpful development assistant with my preferred defaults.

When writing code, always:
- Use type hints for all function signatures
- Add docstrings to public functions
- Prefer explicit error handling over bare except clauses

When explaining code, be concise. Lead with what the code does,
then cover how it works.

## Agents

You can delegate code review tasks to **quick-reviewer**, a fast
code review specialist.
```

Notice the agent reference uses `my-tools:quick-reviewer` — the bundle name as namespace, then the agent name (which matches the file in `agents/`).

## Step 5: Understand the Thin Bundle Pattern

Look at what we built: our `bundle.md` is only about 15 lines of YAML. All the heavy lifting — tools, session management, hooks — comes from foundation via `includes`. We only declared what's unique to us: custom instructions and one agent.

This is called the **thin bundle pattern**, and it's how most bundles should work. Don't redeclare tools that foundation already provides. Don't copy session configuration. Just include foundation and add your layer on top.

Here's what a thin bundle looks like versus a bloated one:

```yaml
# GOOD: thin bundle — only declares what's new
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
agents:
  include:
    - my-tools:quick-reviewer
```

```yaml
# BAD: redeclares things foundation already has
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
tools:
  - module: tool-filesystem       # foundation has this
    source: git+https://...
  - module: tool-bash             # foundation has this too
    source: git+https://...
session:
  orchestrator:                   # foundation handles this
    module: loop-streaming
```

The thin approach means you automatically get foundation updates. When foundation adds a new tool or improves a hook, your bundle picks it up for free.

## Step 6: Compose Multiple Bundles

Bundles can include other bundles, building capability in layers. The `includes` key accepts multiple entries:

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
```

This gives you everything from foundation plus everything from recipes. Your custom instructions and agents layer on top. Later entries override earlier ones if there's a conflict — so your bundle's instructions replace foundation's, and your agents merge in alongside any agents from included bundles.

## Verify

Let's confirm your custom bundle works end to end:

```bash
# Load your bundle
amplifier run --bundle ~/.amplifier/bundles/my-tools/bundle.md "List the agents you have available"
```

> I have access to **quick-reviewer** — a fast code review specialist...

```bash
# Test the agent
amplifier run --bundle ~/.amplifier/bundles/my-tools/bundle.md \
  "Delegate to quick-reviewer: review this function:

def calc(a, b, op):
    if op == 'add': return a+b
    elif op == 'sub': return a-b"
```

The assistant should delegate to your quick-reviewer agent, which will flag the missing type hints, absent docstring, and possibly suggest using a match statement or dictionary dispatch.

## Try It Out: Set Your Default Bundle

If you want your custom bundle to be the default for all sessions, add it to your global settings:

```bash
amplifier bundle use ~/.amplifier/bundles/my-tools/bundle.md --global
```

Or edit `~/.amplifier/settings.yaml` directly:

```yaml
bundle: ~/.amplifier/bundles/my-tools/bundle.md
```

Now every time you run `amplifier`, it loads your bundle automatically. No `--bundle` flag needed.

You can always override it for a single session:

```bash
amplifier run --bundle foundation "Quick question without my customizations"
```

## Your Final Bundle Structure

Here's what you built:

```
~/.amplifier/bundles/my-tools/
├── bundle.md              # Bundle definition (YAML frontmatter + instructions)
└── agents/
    └── quick-reviewer.md  # Agent definition
```

Compact, composable, and entirely yours. As your needs grow, you can add more agents, include additional bundles, or create context files with domain knowledge.

## Next Steps

You've created a working custom bundle. Here's where to go from here:

1. **[Core Concepts](../concepts/index.md)** — Understand bundles, agents, and modules at a deeper level
2. **[Bundles Guide](../bundles/index.md)** — Advanced patterns: behaviors, context files, local tool modules
3. **[Tools Reference](../tools/index.md)** — Every tool available and how to configure it
4. **[How It's Different](./how-its-different.md)** — Why Amplifier's architecture works this way
