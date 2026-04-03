---
id: team-knowledge
type: bundle
title: "Team Knowledge Bundle"
---

# Team Knowledge Bundle

## What is the Team Knowledge Bundle?

You're working alone and Amplifier knows your codebase well. But teams don't work alone. Somebody built the authentication service six months ago. Somebody else owns the deployment pipeline. There's a convention about how error codes are structured that nobody wrote down. When a new engineer joins, they spend weeks piecing this together from Slack threads and stale wikis.

The Team Knowledge Bundle gives Amplifier shared team memory. It captures what your team builds, who works on what, and the standards you follow -- then makes all of it searchable from any session. When you ask "who knows about the payment system?" or "what's our convention for API versioning?", Amplifier answers from the knowledge base instead of guessing.

The design is local-first. You generate knowledge from your repos, review it locally, and push when you're ready to share. Nothing leaves your machine until you say so.

## What's Included

The bundle provides two interfaces to the same knowledge base:

**The `team_knowledge` tool** -- your in-session interface. Search, look up details, list what's known, and publish new entries. This is what Amplifier uses during conversations to answer team questions.

**The `team-kb` CLI** -- your terminal interface for bulk operations. Generate knowledge by scanning repositories, push local changes to share with the team, pull updates from others, and manage team membership.

All knowledge is organized into three entity categories:

| Category | What It Captures | Examples |
|----------|-----------------|---------|
| **Capability** | A project, service, or area of expertise the team builds | "payment-service", "CI/CD pipeline", "search indexer" |
| **Person** | A team member and what they work on | "Alice -- owns auth service, expert in OAuth" |
| **Convention** | A team standard or agreed-upon practice | "API errors use RFC 7807 format", "all services log to structured JSON" |

## Getting Started

The typical workflow has three steps: join a team, generate knowledge from your repos, and push to share.

### Step 1 -- Join or Create a Team

From your terminal, initialize the knowledge base and join a team:

```
$ team-kb init
$ team-kb join my-team
```

This creates the local knowledge store at `~/.amplifier/team-knowledge/` and registers you as a member of `my-team`.

### Step 2 -- Generate from Your Repos

Point the CLI at your repositories and it scans for capabilities, conventions, and ownership signals:

```
$ team-kb generate --repo ~/dev/payment-service --repo ~/dev/shared-libs
Scanning payment-service...
  Found: 3 capabilities, 2 conventions
Scanning shared-libs...
  Found: 1 capability, 1 convention
Generated 7 entries. Review with: team-kb status
```

The generator reads code structure, README files, configuration patterns, and commit history to infer what exists and who works on it. Review the results before sharing:

```
$ team-kb status
Local changes (not yet pushed):
  [capability] payment-service     NEW
  [capability] payment-webhooks    NEW
  [capability] shared-http-client  NEW
  [convention] error-format-rfc7807  NEW
  [convention] structured-logging    NEW
  [person] you                       UPDATED
```

### Step 3 -- Push to Share

When you're satisfied with what was generated, push to make it available to the team:

```
$ team-kb push
Pushed 7 entries to my-team.
```

Now anyone on the team can search and discover what you've contributed.

## Key Features

### Searching and Discovering (Layer 0)

The first layer of discovery is broad: search across everything or list what's known. This is how you orient yourself.

> What does the team know about payments?

```
[Tool: team_knowledge]
  operation: search
  query: "payments"

Results:
  [capability] payment-service — Stripe-based payment processing, owned by Alice
  [capability] payment-webhooks — Webhook handlers for Stripe events, owned by Alice
  [convention] idempotency-keys — All payment endpoints require idempotency keys
```

> List all the team's conventions

```
[Tool: team_knowledge]
  operation: list
  category: convention

  error-format-rfc7807 — API errors follow RFC 7807 Problem Details
  structured-logging — All services emit structured JSON logs
  idempotency-keys — Payment endpoints require idempotency keys
  branch-naming — Feature branches use feat/TICKET-description format
```

### Looking Up Details (Layer 1)

Once you find something, look it up for full details -- description, ownership, dependencies, and usage guidance:

> Tell me more about the payment-service capability

```
[Tool: team_knowledge]
  operation: lookup
  name: "payment-service"

  Name: payment-service
  Type: capability
  Description: Stripe-based payment processing service handling
    charges, refunds, and subscription management.
  Owner: Alice
  Dependencies: shared-http-client, user-service
  Usage: Exposes REST API on port 8080. See docs/ for endpoint reference.
```

### Full Artifacts (Layer 2)

The deepest layer includes diagrams, activity history, and full convention content. This level of detail comes from the knowledge entries themselves and any linked artifacts in the repository.

> Show me the full convention for our error format

```
[Tool: team_knowledge]
  operation: lookup
  name: "error-format-rfc7807"

  Name: error-format-rfc7807
  Type: convention
  Description: All API errors follow RFC 7807 Problem Details format.
  Content:
    - Response body: { "type", "title", "status", "detail", "instance" }
    - type field uses URI format: /errors/payment-declined
    - 4xx errors include actionable detail text
    - 5xx errors use generic detail (no internal leaks)
```

### Publishing New Knowledge

You can also publish directly from a session when you discover something worth capturing:

> We just agreed that all new services need health check endpoints. Capture that as a convention.

```
[Tool: team_knowledge]
  operation: publish
  name: "health-check-endpoints"
  category: convention
  content:
    description: "All new services must expose /healthz and /readyz endpoints"
    type: "convention"
    usage: "/healthz returns 200 when process is alive. /readyz returns 200
      when service can handle traffic. Both return JSON with status field."
```

Published entries are saved locally first. Run `team-kb push` from your terminal to share them with the team.

### Keeping Knowledge Current

Pull the latest from your team, then update your local entries by re-scanning:

```
$ team-kb pull
Pulled 12 new/updated entries from my-team.

$ team-kb update --repo ~/dev/payment-service
Scanning payment-service...
  Updated: 1 capability (payment-service)
  New: 1 capability (payment-subscriptions)

$ team-kb push
Pushed 2 entries to my-team.
```

## Practical Example: Onboarding a New Team Member

Here's what the knowledge base looks like in practice. Jordan just joined the team and needs to get up to speed.

> I just joined the team. What does this team work on?

```
[Tool: team_knowledge]
  operation: list
  category: capability

  payment-service — Stripe-based payment processing (Alice)
  payment-webhooks — Webhook handlers for Stripe events (Alice)
  user-service — User registration and profile management (Bob)
  search-indexer — Elasticsearch indexing pipeline (Carol)
  shared-http-client — Common HTTP client with retry/circuit-breaker (Bob)
  ci-cd-pipeline — GitHub Actions deployment pipeline (Carol)
```

> Who should I talk to about the search infrastructure?

```
[Tool: team_knowledge]
  operation: search
  query: "search infrastructure"

  [person] Carol — Owns search-indexer and ci-cd-pipeline.
    Expertise: Elasticsearch, GitHub Actions, infrastructure automation.
    Notes: Prefers async communication. Available in #search-team channel.
```

> What conventions should I know about before I start writing code?

```
[Tool: team_knowledge]
  operation: list
  category: convention

  error-format-rfc7807 — API errors follow RFC 7807 Problem Details
  structured-logging — All services emit structured JSON logs
  idempotency-keys — Payment endpoints require idempotency keys
  branch-naming — Feature branches use feat/TICKET-description format
  health-check-endpoints — All services expose /healthz and /readyz
```

In five minutes, Jordan knows what the team builds, who owns what, and the standards to follow. No Slack archaeology required.

## Tips

**Generate before you write.** Run `team-kb generate` against your repos before manually creating entries. The scanner catches things you'd forget to document.

**Keep entries focused.** One capability per service or component, not one per repository. If a repo contains three distinct services, that's three capability entries.

**Never edit `~/.amplifier/team-knowledge/` directly.** Always use the CLI or the `team_knowledge` tool. Direct edits can corrupt the local store or create conflicts on push.

**Push regularly.** Local-first is great for review, but knowledge only helps the team when it's shared. Make pushing part of your routine, like committing code.

**Use conventions for decisions, not aspirations.** A convention should describe what the team *does*, not what it wishes it did. "All services use structured logging" is a convention. "We should probably add monitoring" is a TODO.

**Review generated entries.** The scanner infers from code signals -- it's good but not perfect. Spend a minute checking `team-kb status` before pushing to make sure descriptions are accurate and ownership is correct.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core capabilities that team knowledge builds on
- Explore [Skills](../concepts/skills.md) for adding domain knowledge to individual sessions
- See the [Recipes Bundle](./recipes.md) to automate team workflows like onboarding checklists
- Read about [Bundles](../concepts/bundles.md) to understand how Team Knowledge composes with other bundles
