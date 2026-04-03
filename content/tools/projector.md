---
id: tool-projector
type: tool
title: "Projector Tool"
---

# Projector Tool

Most conversations with Amplifier are about immediate work — fix this bug, write this function, explore this module. But real projects span weeks and months, with goals that evolve, strategies that shift, and work that accumulates across many sessions. The Projector tool is how Amplifier tracks that longer arc: projects with strategies, tasks, and outcomes that persist across every session you have.

Think of Projector as a lightweight project management layer built into your coding assistant. It doesn't replace Jira or Linear — it gives Amplifier itself memory about what you're trying to accomplish, so it can bring relevant context to every conversation without you having to re-explain your project's goals.

## What Is Projector?

Projector manages four connected concepts:

| Concept | What It Holds | Lifespan |
|---------|--------------|----------|
| **Projects** | Named containers for related work | Permanent until deleted |
| **Strategies** | Persistent guidelines and priorities for a project | Persist across sessions, injected via hooks |
| **Tasks** | Individual work items within a project | Tracked until completed |
| **Outcomes** | Records of what happened in each session | Logged per session, accumulated over time |

The key insight is **strategies**. A strategy isn't a one-time instruction — it's a persistent directive that Amplifier sees at the start of *every session* related to that project. Set a strategy once, and it shapes every future conversation until you change it.

## Core Capabilities

Projector has thirteen operations, organized around the four concepts.

### Projects — The Container

**create_project** — start tracking a new project:

> I'm starting work on the payment system rewrite

```
[Tool: projector] create_project
  project: "payment-rewrite"
  data: {
    "description": "Rewrite the legacy payment system to use Stripe",
    "goals": ["Replace PayPal integration", "Add subscription support", "Improve error handling"]
  }

  ✓ Project "payment-rewrite" created.
```

**get_project** — see the full state of a project:

```
[Tool: projector] get_project
  project: "payment-rewrite"

  Project: payment-rewrite
  Description: Rewrite the legacy payment system to use Stripe
  Goals: Replace PayPal integration, Add subscription support, Improve error handling
  Strategies: 2 active
  Tasks: 3 open, 1 completed
  Last activity: 2 hours ago
```

**list_projects** — see all tracked projects:

```
[Tool: projector] list_projects

  Projects:
    payment-rewrite    — 3 open tasks, 2 strategies
    auth-hardening     — 1 open task, 1 strategy
    docs-migration     — 5 open tasks, 0 strategies
```

**update_project** — modify project goals, description, or other details as the project evolves.

### Strategies — Persistent Context

Strategies are the most powerful part of Projector. They persist across sessions and are injected into your conversation context via hooks — meaning Amplifier starts every session already knowing your project's priorities and constraints.

**set_strategy** — define a strategy for a project:

> For the payment rewrite, we should always prioritize error handling and never skip webhook signature verification

```
[Tool: projector] set_strategy
  project: "payment-rewrite"
  strategy: "error-first"
  data: {
    "description": "Prioritize robust error handling in all payment code",
    "guidelines": [
      "Every external API call must have explicit error handling",
      "Never skip webhook signature verification",
      "Log all payment state transitions for audit"
    ]
  }

  ✓ Strategy "error-first" set for project "payment-rewrite".
```

Next time you open a session and work on the payment rewrite, Amplifier will already know about the "error-first" strategy. You don't have to repeat these constraints — they're injected automatically.

**list_strategies** — see all strategies for a project:

```
[Tool: projector] list_strategies
  project: "payment-rewrite"

  Strategies:
    error-first     — Prioritize robust error handling (active)
    test-coverage   — Maintain >90% coverage on payment modules (active)
```

**get_strategy** — see the full details of a specific strategy, including its guidelines and status.

**toggle_strategy** — enable or disable a strategy without deleting it:

```
[Tool: projector] toggle_strategy
  project: "payment-rewrite"
  strategy: "test-coverage"

  ✓ Strategy "test-coverage" toggled to inactive.
```

Toggling is useful when a strategy is temporarily irrelevant — you're in a prototyping phase where coverage doesn't matter yet, but you want the strategy ready for later. Toggle it back on when you're ready.

### Tasks — Tracking Work

Tasks are individual work items within a project. They're simpler than strategies — no cross-session injection — but they give Amplifier (and you) a clear picture of what needs doing.

**add_task** — create a new task:

> Add a task to implement the Stripe webhook handler

```
[Tool: projector] add_task
  project: "payment-rewrite"
  data: {
    "title": "Implement Stripe webhook handler",
    "description": "Handle payment_intent.succeeded, payment_intent.failed, and charge.refunded events",
    "priority": "high"
  }

  ✓ Task added to project "payment-rewrite".
```

**list_tasks** — see all tasks in a project:

```
[Tool: projector] list_tasks
  project: "payment-rewrite"

  Tasks:
    ☐ Implement Stripe webhook handler (high)
    ☐ Migrate customer records to new schema (medium)
    ☐ Write integration tests for checkout flow (medium)
    ✓ Set up Stripe API credentials (low) — completed 3 days ago
```

**update_task** — change a task's status, priority, or details as work progresses.

### Outcomes — Session History

After a session of work, outcomes record what happened. This builds a narrative over time — not just what you planned to do, but what you actually accomplished.

**log_outcome** — record what happened in a session:

```
[Tool: projector] log_outcome
  project: "payment-rewrite"
  data: {
    "summary": "Implemented webhook handler for payment_intent events",
    "details": "Created src/payments/webhooks.py with handlers for succeeded, failed, and refunded events. Added signature verification. All tests passing.",
    "tasks_completed": ["Implement Stripe webhook handler"]
  }

  ✓ Outcome logged for project "payment-rewrite".
```

### Status — The Big Picture

**get_status** — cross-project overview of where things stand:

> What's the status of everything?

```
[Tool: projector] get_status

  payment-rewrite:  3 tasks open, 1 completed. Last active: today.
  auth-hardening:   1 task open, 0 completed. Last active: 3 days ago.
  docs-migration:   5 tasks open, 0 completed. Last active: 1 week ago.
```

You can also filter status queries to focus on a specific project or time range.

## How Strategies Flow Across Sessions

This is the feature that makes Projector more than a to-do list. Here's the lifecycle:

1. **You set a strategy** in session A: "Always validate webhook signatures."
2. **The strategy is stored** in Projector's persistent data.
3. **In session B** (hours or days later), hooks inject the strategy into the session context.
4. **Amplifier sees the strategy** before processing any request, shaping its approach.
5. **You update or toggle** the strategy as priorities evolve.

This means your second session doesn't start cold. Amplifier already knows the project's constraints. If you ask it to implement a webhook handler, it will include signature verification because the strategy says to — without you having to remind it.

## Practical Workflow

A typical project lifecycle with Projector:

**Setup** — create the project, set strategies, add initial tasks:

```
[Tool: projector] create_project  project: "api-redesign"
[Tool: projector] set_strategy    strategy: "backwards-compat"
[Tool: projector] set_strategy    strategy: "test-coverage"
[Tool: projector] add_task        "Design new endpoint schema"
[Tool: projector] add_task        "Implement v2 endpoints"
```

**Working sessions** — strategies are injected automatically. At session end, log what happened:

```
[Tool: projector] log_outcome  "Completed v2 endpoint design. Found 3 breaking changes."
[Tool: projector] update_task  "Design new endpoint schema" → completed
```

**Priorities shift** — toggle strategies, add new ones, keep moving:

```
[Tool: projector] toggle_strategy  "backwards-compat" → inactive
[Tool: projector] set_strategy     "performance-first"
```

## Tips

**Set strategies early.** The sooner you define project constraints, the sooner every session benefits from them. Don't wait until you've been burned by a missing guideline.

**Keep strategies focused.** A strategy that says "write good code" is useless. A strategy that says "every database query must use parameterized statements" is actionable. Specific guidelines produce specific results.

**Log outcomes consistently.** The value of outcomes compounds over time. After ten sessions, you have a narrative of the project's evolution. After one session, you have a note you could have written anywhere.

**Use toggle, not delete.** When a strategy is temporarily irrelevant, toggle it off. You spent thought on crafting it — don't throw it away. Toggle it back when it matters again.

**Check status at the start of sessions.** A quick `get_status` at the beginning of a work session reminds you (and Amplifier) where things stand across all projects.

**Tasks are lightweight on purpose.** If you need detailed issue tracking with labels, assignees, and milestones, use your team's issue tracker. Projector tasks are for giving Amplifier a quick sense of what's on the plate — not for replacing project management software.

## Next Steps

- Learn about [Hooks](../concepts/hooks.md) to understand how strategies get injected into your sessions
- See the [Mode Tool](./mode.md) for another way to inject context and policies into sessions
- Explore [Recipes](./recipes-tool.md) for multi-step workflows that can interact with Projector
