---
id: dev-memory
type: bundle
title: "Dev Memory Bundle"
---

# Dev Memory Bundle

## What is Dev Memory?

You're three days into a refactor. You've made architectural decisions, discovered quirks in the codebase, settled on naming conventions, and figured out that the CI pipeline needs a special flag for integration tests. Then you start a new session — and all of that is gone. You're back to explaining context from scratch.

The Dev Memory bundle gives Amplifier persistent memory across every session. It remembers what you've told it, what you've worked on, and the decisions you've made — not in some cloud service, but in plain files on your local machine. When you say "remember this," it's saved. When you ask "what was I working on?" next week, it knows.

This isn't a vector database or a retrieval-augmented generation pipeline. It's deliberately simple: a set of YAML and Markdown files in a known location, written with direct appends, read by a lightweight sub-agent. The design prioritizes token efficiency and transparency over sophistication. You can open the memory files in any editor and see exactly what Amplifier remembers.

## What's Included

The bundle provides three components:

| Component | Role |
|-----------|------|
| **Memory store** | `memory-store.yaml` — categorized facts, decisions, and preferences |
| **Work log** | `work-log.yaml` — timestamped record of what you've worked on |
| **Project notes** | `project-notes.md` — freeform notes and longer-form context |

These live in `~/amplifier-dev-memory/`, a single directory you can back up, version-control, or move between machines. Everything is local — no external APIs, no cloud sync, no telemetry.

The bundle also includes a **memory-retrieval** sub-agent that handles reads. When you ask "what do you remember about the auth service?", the read is delegated to this agent so your root session doesn't spend tokens loading the full memory store. Writes are handled differently — direct appends to the YAML files, fast and cheap.

## Getting Started

Memory works through natural language. There's no special syntax or commands to learn.

**Saving a memory:**

> Remember this: the payments service uses Stripe webhooks with idempotency keys, and we validate signatures in middleware before the handler runs

```
[Tool: dev-memory] Appending to memory-store.yaml
  Category: architecture
  Content: "Payments service uses Stripe webhooks with idempotency keys.
            Signature validation happens in middleware, before handler."
  Saved.
```

That's it. You stated a fact, and it's persisted. The bundle auto-categorized it as `architecture` based on the content.

**Recalling a memory:**

> What do you remember about the payments service?

```
[Tool: task] Delegating to dev-memory:memory-retrieval...
  Searching memory-store.yaml for: payments service
  Found 2 entries:
  - [architecture] Payments service uses Stripe webhooks with idempotency keys.
    Signature validation happens in middleware, before handler.
  - [environment] Payments service runs on port 4200 locally,
    requires STRIPE_SECRET_KEY in .env
```

The read went to the memory-retrieval sub-agent. Your root session received a concise summary without loading the entire memory file.

**Checking your work log:**

> What was I working on last week?

```
[Tool: task] Delegating to dev-memory:memory-retrieval...
  Scanning work-log.yaml for recent entries...
  Last week:
  - Mon: Refactored auth middleware to support JWT rotation
  - Tue-Wed: Built rate limiting for the public API (sliding window)
  - Thu: Fixed flaky test in CI — race condition in websocket teardown
  - Fri: Started GraphQL schema design for the reporting endpoint
```

## Key Features

### Auto-Categorization

When you save a memory, the bundle classifies it into one of eight categories:

| Category | What Goes Here |
|----------|---------------|
| **architecture** | System design, service boundaries, data flow decisions |
| **workflow** | How you work — branching strategy, review process, deploy steps |
| **environment** | Local setup, ports, env vars, tool versions |
| **git** | Repository conventions, branch naming, commit message formats |
| **research** | Findings from investigation — library comparisons, API quirks |
| **pattern** | Code patterns you prefer — error handling, naming, file structure |
| **preference** | Personal preferences — editor settings, formatting rules, style choices |
| **tools** | Tool configurations, CLI shortcuts, build system quirks |

You don't need to specify the category. The bundle infers it from context. But you can override it if you want:

> Remember this as a workflow note: always run the linter before pushing to the feature branch

```
[Tool: dev-memory] Appending to memory-store.yaml
  Category: workflow (explicit)
  Content: "Always run linter before pushing to feature branch."
  Saved.
```

### Token-Efficient Architecture

Memory reads and writes are intentionally asymmetric:

**Writes** happen via direct append — no sub-agent, no round-trip. When you say "remember X," the bundle appends a structured entry to the YAML file. This costs almost nothing in tokens.

**Reads** are delegated to the memory-retrieval sub-agent. This agent loads the memory files in its own context, searches for relevant entries, and returns a concise summary. Your root session never sees the full memory store, which could be thousands of entries after months of use.

This split means that accumulating memories is cheap, and recalling them scales gracefully even as your memory grows.

### Natural Conversation Flow

Memory integrates naturally into your workflow. You don't need to stop and explicitly manage it — just talk:

> I just figured out that the Docker build fails if you don't set BUILDKIT=1. Remember that.

```
[Tool: dev-memory] Appending to memory-store.yaml
  Category: environment
  Content: "Docker build requires BUILDKIT=1 or it fails."
  Saved.
```

> We decided to use Zod for runtime validation instead of io-ts. The team prefers its API.

```
[Tool: dev-memory] Appending to memory-store.yaml
  Category: pattern
  Content: "Using Zod for runtime validation (preferred over io-ts by team)."
  Saved.
```

Later, in a completely different session:

> I need to add validation to the new endpoint. What validation library do we use?

```
[Tool: task] Delegating to dev-memory:memory-retrieval...
  Found: Using Zod for runtime validation (preferred over io-ts by team).
```

Amplifier answers from memory, not from guessing.

### Transparent Storage

Everything is stored in plain text at `~/amplifier-dev-memory/`:

```
~/amplifier-dev-memory/
├── memory-store.yaml    # Categorized facts and decisions
├── work-log.yaml        # Timestamped work entries
└── project-notes.md     # Freeform longer-form notes
```

You can read these files directly, edit them in your editor, grep through them, or commit them to a private repo. There's no opaque database — what you see is what Amplifier sees.

### Project Notes

For longer-form context that doesn't fit neatly into a YAML entry, the bundle supports freeform project notes:

> Add a project note: The reporting service architecture follows CQRS. Commands go through a message bus to write models, queries read from denormalized views in Postgres. We chose this because the read/write ratio is approximately 100:1.

```
[Tool: dev-memory] Appending to project-notes.md
  Added note under "Reporting Service Architecture"
  Saved.
```

Project notes are ideal for design rationale, meeting summaries, or any context that benefits from narrative form rather than structured entries.

## Tips

**Be specific when saving.** "Remember that the API is weird" is less useful than "Remember that the billing API returns 200 with an error body instead of using HTTP status codes." Specific memories are findable memories.

**Front-load decisions.** At the start of a project, tell Amplifier your key decisions: tech stack, conventions, architecture patterns. This pays off in every subsequent session where it already knows your context.

**Review your memory periodically.** Open `~/amplifier-dev-memory/memory-store.yaml` occasionally and prune entries that are no longer relevant. Outdated memories can mislead future sessions.

**Use the work log for continuity.** At the end of a session, say "log that I was working on X and got to Y." The next session can pick up exactly where you left off.

**Don't over-remember.** Not everything needs to be persisted. Save decisions, discoveries, and context that would be painful to re-derive. Skip transient details that won't matter tomorrow.

**Back up your memory directory.** Consider adding `~/amplifier-dev-memory/` to a private git repo or your backup system. Months of accumulated project knowledge is worth protecting.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core agents and tools
- Explore the [Projector Bundle](./projector.md) for project management and task tracking
- Read about [Strategies](../concepts/hooks.md) and how persistent instructions work across sessions
- See [Your First Bundle](../quickstart/first-bundle.md) for building custom bundles
