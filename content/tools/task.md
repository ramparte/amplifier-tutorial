---
id: tool-task
type: tool
title: "Delegate Tool"
---

# Delegate Tool

Most of the time, Amplifier handles your requests directly — reading files, running commands, editing code. But some tasks are too big or too specialized for a single pass. That's where the delegate tool comes in: it spawns an independent sub-agent, hands it a task, and gets back a distilled result.

Think of it as sending a colleague off to investigate something. You describe what you need, they go away and do the work autonomously — reading files, running commands, exploring code — and come back with a summary. You never see their intermediate steps, just the final report.

The real power isn't just delegation itself. It's *context control* — you decide exactly how much of your current conversation the sub-agent sees, and you can run multiple agents in parallel without any of them blocking each other.

## Core Capabilities

The delegate tool takes these parameters:

- **agent** (required) — which agent to spawn: `"foundation:explorer"`, `"self"`, or any bundle path
- **instruction** (required) — what you want the agent to do
- **context_depth** — how far back in history to share: `"none"`, `"recent"`, or `"all"`
- **context_scope** — what kinds of content to include: `"conversation"`, `"agents"`, or `"full"`
- **context_turns** — number of turns to include when depth is `"recent"`
- **session_id** — resume a previous agent session instead of starting fresh
- **model_role** — override which model the sub-agent uses (e.g., `"reasoning"`, `"fast"`)
- **provider_preferences** — ordered list of provider/model preferences

The first two get you started. The rest give you fine-grained control over cost, speed, and what the agent knows.

## Context Control

This is the key feature that separates naive delegation from effective delegation. Context has two independent dimensions — *depth* and *scope* — and you control them separately.

**Depth** decides how far back in time the agent can see:

| Depth | What it means | When to use |
|-------|--------------|-------------|
| `"none"` | Clean slate — agent sees only your instruction | Default exploration, fresh investigations |
| `"recent"` | Last N turns of history | Follow-up tasks that need recent context |
| `"all"` | Full conversation history | Tasks that need everything discussed so far |

**Scope** decides what *kinds* of content from those turns get included:

| Scope | What it includes | When to use |
|-------|-----------------|-------------|
| `"conversation"` | Just the text between you and the user | Lightweight context, minimal tokens |
| `"agents"` | Text + results from previous delegations | Building on prior agent findings |
| `"full"` | Text + agent results + all tool outputs | Maximum context, highest token cost |

These are independent axes. `depth: "recent"` with `scope: "full"` gives you the last few turns including all tool output. `depth: "all"` with `scope: "conversation"` gives you the entire conversation but only the human-readable parts. Pick the combination that gives the agent what it needs without drowning it in tokens.

## Examples

### Exploring a Codebase

The most common delegation — send an explorer to map out unfamiliar code:

> I need to understand the authentication system in this project

```
[Tool: task]
  agent: "foundation:explorer"
  instruction: "Explore the authentication system. Find all auth-related
    modules, map the login flow from request to session creation, identify
    the token strategy (JWT, session cookies, etc.), and list any external
    auth providers. Return a structured summary."
  context_depth: "none"

Explorer returns:
  Authentication lives in src/auth/ with 4 modules:
  - handlers.py: Login/logout endpoints using FastAPI
  - tokens.py: JWT creation and validation (RS256, 1hr expiry)
  - providers/: Google and GitHub OAuth via authlib
  - middleware.py: Request authentication middleware
  Login flow: POST /login → validate credentials → issue JWT → set cookie
  ...
```

The agent read dozens of files, traced imports, and came back with a clean summary. You got the answer without those files ever entering your context window.

### Debugging a Failure

> The payment webhook tests are failing — can you figure out why?

```
[Tool: task]
  agent: "foundation:explorer"
  instruction: "Investigate why payment webhook tests are failing.
    Run the tests first to see the actual errors引用, then trace the code
    to find the root cause. Check recent changes to webhook handling."
  context_depth: "none"

Agent returns:
  Root cause: The Stripe webhook signature verification in
  src/payments/webhooks.py was changed to use a new signing secret
  env var (STRIPE_WEBHOOK_SECRET_V2) but the test fixtures still
  reference the old var (STRIPE_WEBHOOK_SECRET). Fix: update
  tests/fixtures/payment_fixtures.py line 34.
```

### Parallel Dispatch

When you have multiple independent questions, launch them all at once. Amplifier runs them simultaneously:

> Analyze the auth module — I need architecture, test coverage, and security review

```
[Tool: task]                              [Tool: task]                          [Tool: task]
  agent: "foundation:explorer"              agent: "foundation:explorer"          agent: "foundation:explorer"
  instruction: "Map the auth               instruction: "Analyze test            instruction: "Security review
    module architecture..."                   coverage for auth..."                 of auth module..."
  context_depth: "none"                     context_depth: "none"                 context_depth: "none"

All three return simultaneously — total wall time equals the slowest agent, not the sum.
```

Three tasks that take 90 seconds each finish in 90 seconds total, not 270. This is the parallel dispatch pattern: whenever tasks don't depend on each other's results, launch them together.

### The Context Sink Pattern

Sub-agents don't just do work — they *absorb token cost*. When an explorer reads 50 files to answer your question, those files live in the agent's context, not yours. You get back a paragraph-length summary.

This is the context sink pattern: delegate expensive exploration to a sub-agent, receive a distilled result. Your main conversation stays lean.

> What ORMs and database patterns does this project use?

```
[Tool: task]
  agent: "foundation:explorer"
  instruction: "Survey all database access patterns in this project.
    Identify the ORM (if any), raw SQL usage, migration strategy,
    and connection pooling setup. List every model/table found."
  context_depth: "none"

Agent reads 80+ files internally, returns a 15-line summary.
Your context window barely notices.
```

This matters most in long conversations where you're already using significant context. Delegation keeps the main thread from bloating.

### Self-Delegation

The special agent value `"self"` spawns a copy of yourself as a sub-agent. It has the same tools and capabilities as the main session, but runs in isolation:

> Refactor the logger module — extract the formatters into their own file

```
[Tool: task]
  agent: "self"
  instruction: "Refactor src/utils/logger.py. Extract all formatter
    classes (JsonFormatter, TextFormatter, StructuredFormatter) into
    a new file src/utils/formatters.py. Update all imports across
    the codebase. Run the tests to verify nothing breaks."
  context_depth: "recent"
  context_turns: 3

Agent performs the refactor in isolation, returns a summary of changes made.
```

Self-delegation is useful for token conservation — the sub-agent handles a well-scoped task without accumulating all the context from your main conversation. It's also a natural fit for tasks you *could* do directly but want to keep out of your current context.

### Session Resumption

When an agent finishes, it returns a `session_id`. You can resume that session to continue where it left off:

> Can you dig deeper into the OAuth provider setup you found earlier?

```
[Tool: task]
  agent: "foundation:explorer"
  session_id: "session_20260402_143022_a3f2"
  instruction: "Go deeper on the OAuth providers. How is token refresh
    handled? Is there error handling for expired tokens? Check if the
    Google and GitHub providers share a common interface."

Agent resumes with full memory of its previous exploration.
```

Resumption avoids re-exploring code the agent already mapped. Use it for iterative investigation — first pass gets the lay of the land, follow-up passes drill into specifics.

## Delegate vs. Do It Yourself

| Situation | Approach | Why |
|-----------|----------|-----|
| Read 1-2 known files | Do it yourself | `read_file` is instant, no overhead |
| Grep for a specific string | Do it yourself | `grep` returns in milliseconds |
| Understand a whole module | **Delegate** | Agent reads dozens of files, returns a summary |
| Multi-step investigation | **Delegate** | Agent can reason across many files autonomously |
| Run a single test | Do it yourself | `bash pytest test_foo.py` is faster |
| Debug a complex failure | **Delegate** | Agent runs tests, reads code, traces the issue |
| Three independent analyses | **Delegate in parallel** | Wall time = max(tasks), not sum(tasks) |
| Long conversation, need exploration | **Delegate** | Context sink keeps your main thread lean |

The rule of thumb: if a task requires reading more than a handful of files or involves multi-step reasoning, delegate it. If it's a single command or a single file, just do it directly.

## Tips

- **Write specific instructions.** "Check the auth code" gives vague results. "Map the login flow from HTTP request to session creation, identify the token strategy, and list all middleware involved" gives precise results.
- **Specify the return format.** Tell the agent what you want back: "Return a bullet-point list of findings" or "Return a JSON summary with file paths and issue descriptions."
- **Default to `context_depth: "none"`.** Most delegated tasks work better with a clean slate and a clear instruction. Only share history when the agent genuinely needs it.
- **Use parallel dispatch aggressively.** If you have three independent questions, launch three agents. There's no reason to serialize them.
- **Self-delegate for isolation.** When you want to keep a task's intermediate work out of your main context, use `agent: "self"` — same capabilities, contained token cost.
- **Resume sparingly.** A fresh agent with good instructions usually outperforms a resumed session with vague follow-ups. Reserve resumption for genuinely iterative exploration.
- **Match model_role to the task.** Use `"reasoning"` for architectural analysis, `"fast"` for simple file operations, `"coding"` for implementation work.

## Next Steps

- See the [Search Tools](./search.md) for the `grep` and `glob` tools that agents use internally
- Learn about [Recipes](./recipes-tool.md) for multi-step workflows that go beyond single delegations
- Read [Agents](../concepts/agents.md) for how agent bundles and specializations work
