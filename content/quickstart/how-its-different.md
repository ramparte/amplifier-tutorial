---
id: how-its-different
type: quickstart
title: "How It's Different"
---

# How Amplifier Is Different

If you've used AI coding tools before — Copilot, ChatGPT, Cursor, or others — you already know what a single AI assistant feels like. Amplifier isn't trying to be a better version of that. It's a different thing entirely. This page walks you through the ideas that make it distinct, with examples you can try yourself.

## What You'll Learn

- Why Amplifier's modular design matters in practice
- How bundles let you build your own AI assistant from parts
- What agent delegation, recipes, modes, and skills look like in a real session
- Concrete things you can do in Amplifier that you can't do elsewhere

---

## The Big Picture

Most AI tools are monolithic. You get one product, made of one codebase, controlled by one vendor. You use what they ship. If it doesn't fit your workflow, you work around it.

Amplifier is the opposite. It's a platform built from **138+ independent components**, supporting **11 AI providers**, with **30+ community bundles** — and every piece is swappable.

Here's the core difference at a glance:

| | Monolithic AI Tools | Amplifier |
|---|---|---|
| **Architecture** | One product, take it or leave it | Modular kernel — swap any piece |
| **Capabilities** | Fixed feature set per release | Compose exactly what you need from bundles |
| **AI Provider** | Locked to one vendor | Route to Anthropic, OpenAI, Azure, Ollama, or others |
| **Workflows** | Manual, ad-hoc | Repeatable recipes with approval gates |
| **Specialization** | One generalist model does everything | Delegate to focused sub-agents |
| **Behavior** | Same mode all the time | Switch modes on the fly — brainstorm, debug, verify |
| **Domain knowledge** | Starts from zero every session | Load skills — packaged expertise on demand |
| **Extensibility** | Plugins, maybe, if they offer an API | MIT licensed, extend anything, fork nothing |

Let's see what each of these means in practice.

---

## 1. Modular Architecture

Amplifier's kernel is intentionally tiny — around 2,600 lines. It handles session lifecycle, module loading, and event routing. Everything else is a module.

That means you can swap your AI provider without touching your tools. You can replace a tool without affecting your agents. You can add a hook that transforms every request — and remove it tomorrow.

Here's what that looks like. Say you're using Anthropic and want to try OpenAI for a task:

> Switch to OpenAI for this project.

```
✓ Provider set to OpenAI (gpt-5.4)
  Your tools, agents, and bundles are unchanged.
```

Nothing else breaks. Your entire environment stays the same — only the model underneath changes. Try doing that with a tool that has the model baked into every layer.

## 2. Bundle Composition

Bundles are how you build your own AI assistant. Instead of getting a fixed set of features, you pick the capability packages you want and compose them.

> What bundles am I using?

```
Active bundle: foundation
  Tools: filesystem, bash, web search, grep, glob
  Agents: zen-architect, bug-hunter, explorer, researcher, modular-builder
  Behaviors: logging, streaming, redaction
```

Want to add recipe-based workflows? Add a bundle:

> Add the recipes bundle.

```
✓ Bundle installed: recipes
  New agents available: recipe-author, recipe-runner
  New tools available: recipe execution engine
```

Want something domain-specific for your team's design system? There's a bundle for that too:

> Add the design-intelligence bundle.

```
✓ Bundle installed: design-intelligence
  New agents available: component-designer, design-auditor
```

You're not configuring a product. You're assembling your own.

## 3. Agent Delegation

In most AI tools, one model does everything. It writes code, reviews code, debugs code, researches solutions — all with the same context, the same persona, the same approach.

Amplifier can delegate to specialized sub-agents. Each one has its own instructions, its own tool access, and its own isolated context. The main agent acts like a tech lead, routing work to the right specialist.

Watch what happens when you ask for a complex task:

> I'm getting a KeyError in auth.py on line 42. Figure out what's wrong and fix it.

Behind the scenes, Amplifier delegates:

```
[Tool: task → bug-hunter agent]
  "Investigate the KeyError in auth.py line 42. Trace the data flow,
   identify the root cause, and propose a fix."
```

The bug-hunter agent works in its own session — reading files, running tests, tracing call chains — then reports back with a focused diagnosis. It can't see your chat history or other unrelated files. That isolation is the point: it stays focused.

You can also ask for delegation explicitly:

> Delegate to zen-architect: Should we use a cache here or a database?

The zen-architect agent thinks about your question through the lens of system design and simplicity, not just code generation. Different agent, different expertise, different result.

## 4. The Recipe System

Some workflows need more than a single prompt. Code reviews, release processes, onboarding checklists — these are multi-step sequences where you want consistency and human checkpoints.

Recipes are declarative YAML workflows that run step by step, with built-in approval gates where you decide whether to continue.

> Run the code-review recipe on my latest changes.

```
Recipe: code-review-workflow
  Step 1/3: Exploring codebase structure...
  [Tool: task → explorer agent]
  ✓ Complete — mapped 12 changed files across 3 modules.

  Step 2/3: Reviewing changes against standards...
  [Tool: task → reviewer agent]
  ✓ Complete — found 2 issues, 4 suggestions.

  ⏸ Approval gate: Review findings before generating summary.
  Continue? [y/n]
```

> y

```
  Step 3/3: Creating review summary...
  ✓ Complete — summary written to review-notes.md.
```

That approval gate is the key. The recipe paused and waited for your judgment before proceeding. You stay in control of multi-step processes without having to re-explain context at each step.

Recipes are resumable, too. If your laptop dies mid-workflow, pick up exactly where you left off.

## 5. The Modes System

Modes change how Amplifier behaves at runtime — without changing what tools or bundles are loaded. Think of them as behavioral overlays.

> /brainstorm

```
Mode: brainstorm — activated
  I'll explore ideas freely, suggest alternatives,
  and avoid committing to any single approach.
```

Now every response leans toward divergent thinking. Ask a question, and instead of one answer, you'll get three options with tradeoffs. When you're ready to commit:

> /debug

```
Mode: debug — activated
  I'll be methodical and evidence-driven.
  Every claim will reference specific code or output.
```

Same tools. Same model. Completely different behavior. Other modes include `/verify` for careful validation passes. You shift gears without restarting your session or changing your setup.

## 6. The Skills System

Skills are loadable packages of domain knowledge. Instead of hoping the model remembers best practices for your stack, you load exactly the expertise you need.

> Load the Python standards skill before we refactor this module.

```
[Tool: load_skill → "python-standards"]
  Loaded: naming conventions, type annotation patterns,
  project structure guidelines, testing expectations.
```

Now every suggestion Amplifier makes is grounded in those standards. Skills exist for security practices, API design patterns, specific frameworks — and you can write your own.

The difference from a system prompt is precision. A skill is a structured, versioned, shareable knowledge package. Your team can maintain a `company-standards` skill and everyone loads the same expertise.

## 7. Model Routing

Not every task needs the same model. A quick file rename doesn't need the same horsepower as a complex architectural decision. Amplifier routes automatically.

When you ask a simple question, a fast model handles it. When you ask for deep analysis, a reasoning model steps in. When a sub-agent needs to do bulk file operations, it gets a model suited for that.

You don't configure this per-request. The routing matrix handles it:

| Task Type | Routed To |
|---|---|
| Quick utility work (parsing, file ops) | Fast, lightweight model |
| Code generation and debugging | Coding-optimized model |
| Architecture and system design | Deep reasoning model |
| Research and synthesis | Research-tuned model |

This happens transparently. You just work. The right model shows up for the job.

## 8. Open and Extensible

Amplifier is MIT licensed. Not "open core with a paid tier." Not "source available but don't compete with us." MIT. The whole thing.

That means you can read every line of the kernel, modify any tool or hook, contribute focused PRs without understanding the whole codebase, and build on top — creating bundles, skills, and recipes for your team or the community.

The CLI you're using right now is just one interface. The modular platform underneath can power web apps, IDE plugins, or collaborative multi-agent systems. The community decides what gets built.

---

## Try It Out

The best way to feel the difference is to try something that would be awkward in another tool. Start an Amplifier session and try these:

> /brainstorm

> I need to add caching to our API. What are my options?

Watch how the brainstorm mode shapes the response — you'll get multiple approaches with tradeoffs instead of a single recommendation. Then:

> /debug

> Actually, let me investigate the current performance first. Profile the API response times.

Notice how the tone and approach shift immediately. Same session, same context, different behavior.

Then try delegation:

> Delegate to zen-architect: Given what we found, which caching strategy fits best?

You'll see the architect agent work through the decision with a design-focused lens, referencing the performance data from your earlier investigation.

That flow — brainstorm, investigate, decide with a specialist — is natural in Amplifier. In a monolithic tool, you'd be fighting the interface the whole way.

---

## Next Steps

Now that you understand how Amplifier's approach is different, put it into practice:

- **[Installation](./installation.md)** — Get Amplifier set up on your machine
- **[First Conversation](./first-conversation.md)** — Start your first interactive session
- **[Key Commands](./key-commands.md)** — Learn the commands that make you productive
- **[Your First Bundle](./first-bundle.md)** — Compose your own capability stack
