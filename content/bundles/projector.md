---
id: projector
type: bundle
title: "Projector Bundle"
---

# Projector Bundle

## What is Projector?

You're juggling three projects. One needs a database migration, another has a backlog of bugs, and the third is blocked waiting for an API design decision. You know all of this — but Amplifier doesn't. Every session starts fresh, with no awareness of what projects exist, what state they're in, or what you decided last time.

The Projector bundle turns Amplifier into a project-aware assistant. It tracks your projects, maintains working strategies that persist across sessions, logs task progress, and records outcomes so you always have cross-project visibility. When you start a new session and ask "what's the status of everything?", Amplifier actually knows.

This isn't a replacement for Jira or Linear. It's lighter and closer to how you think — natural language in, structured tracking out. You say "create a project for the API redesign," and Projector handles the rest. Strategies you set ("always run tests before committing in this project") are injected into every session automatically via hooks. The result is an assistant that remembers not just facts, but *how you want to work*.

## What's Included

The bundle provides one tool and two specialist agents:

| Component | Role |
|-----------|------|
| **`projector` tool** | Core operations — create projects, manage strategies, track tasks, log outcomes |
| **`project-analyst` agent** | Cross-project status analysis and reporting |
| **`strategy-advisor` agent** | Strategy refinement, conflict detection, and optimization |

The `projector` tool exposes twelve operations covering the full lifecycle: `create_project`, `get_project`, `update_project`, `list_projects` for project management; `set_strategy`, `get_strategy`, `list_strategies`, `toggle_strategy` for persistent instructions; `add_task`, `update_task`, `list_tasks` for work tracking; and `log_outcome` plus `get_status` for cross-project visibility.

## Getting Started

Start by creating a project and telling Amplifier how you want to work on it.

> Create a project for the API redesign. We're migrating from REST to GraphQL, targeting Q3 completion.

```
[Tool: projector]
  operation: create_project
  project: "api-redesign"
  data: {
    description: "Migrate public API from REST to GraphQL",
    target: "Q3 2026"
  }

Project "api-redesign" created.
```

Now add a strategy — a persistent instruction for this project:

> For the API redesign, always check backward compatibility before suggesting schema changes

```
[Tool: projector]
  operation: set_strategy
  project: "api-redesign"
  strategy: "backward-compatibility"
  data: { description: "Check backward compatibility before schema changes" }

Strategy "backward-compatibility" set for project "api-redesign".
```

That strategy is now active. Every future session on the API redesign will include it automatically — you never need to say it again. Now add tasks and check the big picture:

> Add tasks for the API redesign: design the schema, build resolvers, set up subscriptions, write migration tests

```
[Tool: projector]
  operation: add_task  (×4)
  project: "api-redesign"

  Added: "Design GraphQL schema" — pending
  Added: "Build query resolvers" — pending
  Added: "Set up subscription layer" — pending
  Added: "Write migration tests" — pending
```

> What's the status of all my projects?

```
[Tool: projector]
  operation: get_status

  api-redesign: 4 tasks (0 done, 4 pending), 1 strategy active
  billing-service: 2 tasks (1 done, 1 in-progress), 3 strategies active
  mobile-app: 6 tasks (4 done, 2 pending), 2 strategies active
```

## Key Features

### Strategies: Persistent Working Instructions

Strategies are Projector's most powerful feature. They're instructions that persist across sessions and are automatically injected into your context via hooks. Think of them as "standing orders" — things you always want Amplifier to keep in mind.

Strategies come in two scopes:

**Project-scoped strategies** apply only when working on a specific project:

> For the billing service, always validate currency codes against ISO 4217

```
[Tool: projector]
  operation: set_strategy
  project: "billing-service"
  strategy: "currency-validation"
  data: { description: "Validate currency codes against ISO 4217 before processing" }

Strategy "currency-validation" set for project "billing-service".
```

**Global strategies** apply across all projects and sessions:

> Set a global strategy: always include error handling in code examples

```
[Tool: projector]
  operation: set_strategy
  strategy: "error-handling-examples"
  data: { description: "Always include error handling in code examples." }

Global strategy "error-handling-examples" set.
```

You can temporarily disable a strategy without losing it:

> Disable the backward-compatibility strategy for now — we're doing a breaking change intentionally

```
[Tool: projector]
  operation: toggle_strategy
  project: "api-redesign"
  strategy: "backward-compatibility"

Strategy "backward-compatibility" disabled for project "api-redesign".
(Re-enable anytime with toggle_strategy)
```

### Tasks: Tracking Work Items

Tasks live inside projects and track individual work items:

> Mark the GraphQL schema design task as complete

```
[Tool: projector]
  operation: update_task
  project: "api-redesign"
  data: { task: "Design GraphQL schema", status: "completed" }

Task updated: "Design GraphQL schema" → completed
```

> What tasks are left on the API redesign?

```
[Tool: projector]
  operation: list_tasks
  project: "api-redesign"

  ☐ Build query resolvers — pending
  ☐ Set up subscription layer — pending
  ☐ Write migration tests — pending
```

Tasks are intentionally lightweight — what needs doing and what's done. No sprint points, priorities, or assignees. For that level of detail, use a dedicated tool and let Projector handle AI-side awareness.

### Outcomes: Session Results

At the end of a work session, log what you accomplished:

> Log outcome: completed the GraphQL schema design for api-redesign. Defined 12 types, 8 queries, 3 mutations.

```
[Tool: projector]
  operation: log_outcome
  project: "api-redesign"
  data: {
    description: "Completed GraphQL schema design. 12 types, 8 queries,
                  3 mutations. Relay-style pagination for list endpoints."
  }

Outcome logged for "api-redesign".
```

When you return after a break, outcomes tell you exactly where you left off:

> What outcomes have been logged for the API redesign?

```
[Tool: projector]
  operation: get_project
  project: "api-redesign"

  Recent outcomes:
  - [Apr 1] Completed GraphQL schema design. 12 types, 8 queries,
    3 mutations. Relay-style pagination for list endpoints.
  - [Mar 28] Finalized type naming conventions — PascalCase types,
    camelCase fields, SCREAMING_SNAKE enums.
```

### Specialist Agents

For deeper analysis, Projector includes two agents that go beyond simple CRUD:

**project-analyst** — Cross-project intelligence. It identifies patterns, bottlenecks, and risks across your portfolio:

> Give me a cross-project analysis — what's at risk?

```
[Tool: task] Delegating to projector:project-analyst...
  Risks:
  - api-redesign: No progress on "Build query resolvers" in 5 days.
    This blocks "Set up subscription layer."
  - billing-service: "Currency validation" in-progress for 8 days.
  Recommendations:
  - Prioritize resolver implementation to unblock the API pipeline.
  - Check if currency validation is blocked on an external dependency.
```

**strategy-advisor** — Refines and optimizes your strategies. Detects conflicts, suggests improvements, identifies gaps:

> Review my strategies for the API redesign project

```
[Tool: task] Delegating to projector:strategy-advisor...
  Findings:
  - "backward-compatibility" is disabled. Consider re-enabling now
    that the breaking change is complete.
  - No testing strategy defined. Consider: "Require integration
    tests for every new resolver."
  - Global "error-handling-examples" applies here — consider a
    project-specific override for GraphQL error patterns.
```

## Tips

**Start with strategies, not tasks.** Strategies shape how Amplifier works with you. Setting them early means every interaction in that project benefits from your conventions and preferences.

**Use global strategies sparingly.** Global strategies apply to every session. Keep them for truly universal preferences — code style, communication tone, error handling philosophy. Project-specific details belong in project-scoped strategies.

**Log outcomes consistently.** Even a one-line outcome at the end of a session builds valuable history. "Fixed the flaky test in auth middleware" is enough.

**Let project-analyst find the patterns.** You know what you worked on today. The analyst sees across all projects and across time. Use it for the perspective you can't easily get yourself.

**Toggle, don't delete.** When a strategy is temporarily irrelevant, disable it with `toggle_strategy` rather than removing it. Strategies represent decisions — preserving them (even disabled) maintains your decision history.

**Keep task granularity consistent.** If one project has 3 tasks and another has 30, the cross-project status view becomes hard to interpret. Aim for similar levels of detail across projects.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core agents and tools
- Explore the [Dev Memory Bundle](./dev-memory.md) for persistent memory across sessions
- Read about [Hooks](../concepts/hooks.md) to understand how strategies are injected
- See [Agents](../concepts/agents.md) for how project-analyst and strategy-advisor work under the hood
