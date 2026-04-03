---
id: writing-agents
type: contributing
title: "Writing Agents"
---

# Writing Agents

Agents are bundles. Same Markdown file, same YAML frontmatter. There is no special
"agent format" to learn. The only difference is a frontmatter convention: bundles use
a `bundle:` key, agents use a `meta:` key. Everything else -- the file structure, the
Markdown body as system prompt, the `@mention` syntax -- works identically.

This guide covers how to write effective agents, with emphasis on the one thing that
matters most: the description.

## The Agent File Format

An agent is a Markdown file in `agents/` inside a bundle. Here's a complete one:

```markdown
---
meta:
  name: api-reviewer
  description: >
    Reviews REST API designs for consistency, RESTfulness, and documentation
    coverage. Use when designing new endpoints, auditing existing APIs, or
    reviewing OpenAPI specs. Examines HTTP methods, status codes, naming
    conventions, pagination, error formats, and documentation completeness.
    Returns a structured report with an overall grade and specific recommendations.
  model_role: coding
---

# API Reviewer

You are an expert in REST API design. When given an API to review, evaluate:

1. **RESTfulness** -- HTTP methods, status codes, resource naming
2. **Consistency** -- naming conventions, pagination patterns, error format
3. **Documentation** -- OpenAPI coverage, example requests and responses

Return a structured report with:
- Overall grade (A-F)
- Issues with severity (critical / warning / note)
- Specific recommendations with code examples

Always examine the actual code, not just documentation. Read route definitions,
middleware, and response handlers to verify behavior matches documentation.
```

That's it. Save it as `agents/api-reviewer.md`, register it in a behavior, and
it's ready for delegation.

## The `meta:` Section

The frontmatter distinguishes agents from bundles:

| Field | Purpose |
|-------|---------|
| `meta.name` | The agent's identifier, used in delegation calls |
| `meta.description` | When and why to use this agent (the most important field) |
| `meta.model_role` | Preferred model routing: `coding`, `reasoning`, `research`, etc. |

```yaml
meta:
  name: security-auditor
  description: >
    Audits code for security vulnerabilities, hardcoded secrets, injection
    risks, and authentication weaknesses. Use when reviewing PRs, onboarding
    to a new codebase, or before releases.
  model_role: coding
```

The `model_role` tells Amplifier's routing which model class to prefer. Common
choices:

| Role | Best For |
|------|----------|
| `coding` | Implementation, code review, debugging |
| `reasoning` | Architecture decisions, complex analysis |
| `research` | Information gathering, documentation lookup |
| `fast` | Quick classification, parsing, bulk operations |

When omitted, the agent inherits the session's default routing.

## The Description: Most Important Field

The description is the single most consequential thing you write. It determines
**whether** Amplifier delegates to your agent, **when** it's selected over
alternatives, and **what** instructions the orchestrator crafts. A bad description
means your agent sits unused or gets called for the wrong tasks.

### Good vs Bad Descriptions

**Bad -- vague and passive:**

```yaml
description: "Helps with code."
```

Why it fails: every agent "helps with code." The orchestrator can't distinguish
this from explorer, modular-builder, or bug-hunter.

**Bad -- too narrow:**

```yaml
description: "Checks Python files for PEP 8 violations."
```

Why it fails: the agent won't be selected for related tasks like reviewing type
hints, checking import organization, or auditing docstring coverage.

**Good -- clear domain with trigger conditions:**

```yaml
description: >
  Reviews Python code quality including PEP 8 style, type annotation coverage,
  import organization, and docstring completeness. Use when reviewing PRs,
  auditing code before release, or establishing coding standards for a project.
  Examines actual source files and returns a prioritized report with specific
  line-level recommendations.
```

Why it works: states the domain (Python code quality), lists what it covers
(style, types, imports, docs), describes trigger situations (PR review, audit,
standards), and sets expectations for output format (prioritized report).

### The Description Formula

Write descriptions that answer four questions:

1. **WHAT** does this agent do? (domain and capabilities)
2. **WHEN** should it be used? (trigger situations)
3. **WHY** choose this over alternatives? (distinguishing expertise)
4. **HOW** does it deliver results? (output format expectations)

```yaml
description: >
  [WHAT] Analyzes test suites for coverage gaps, flaky tests, and structural
  issues. [WHEN] Use when evaluating test health, planning test improvements,
  or investigating CI failures. [WHY] Goes beyond line coverage to assess test
  quality -- meaningful assertions, edge case coverage, test isolation.
  [HOW] Returns a structured report with coverage map, risk areas, and
  prioritized recommendations.
```

You don't need to label the sections -- just make sure all four questions are
answered naturally in the text.

## The Context Sink Pattern

This is the architectural reason agents exist. When an agent investigates your
codebase, it might read 40 files and consume 20,000 tokens of working context.
Your session receives only the distilled summary -- maybe 500 tokens.

Agents that need heavy reference material should carry it themselves using
`@mentions` in the Markdown body:

```markdown
---
meta:
  name: api-reviewer
  description: ...
---

# API Reviewer

@my-bundle:context/api-standards.md

@my-bundle:context/rest-best-practices.md

You are an expert in REST API design...
```

The `@mentioned` files load into the **agent's** context when it spawns -- not
into your main session. This is the context sink in action: heavy reference
material lives with the specialist, not in your conversation.

Use this pattern for agents that need:
- Coding standards or style guides
- API specifications or schemas
- Domain-specific reference material
- Long lists of rules or checklists

Your main session stays lean while the agent has everything it needs.

## Agent Authoring Checklist

Before shipping an agent, verify each of these:

**Clear domain.** Can you state the agent's purpose in one sentence? If you need
two sentences, you might need two agents.

**Good description.** Answers WHAT, WHEN, WHY, HOW. Specific enough to distinguish
from foundation agents. Includes trigger conditions.

**Appropriate model_role.** Match the work to the model: `coding` for implementation,
`reasoning` for analysis, `research` for gathering, `fast` for bulk operations.

**Self-contained instructions.** The Markdown body tells the agent everything it
needs to know -- goals, constraints, output format. Don't assume it inherits
context from the parent session.

**Context files via @mentions.** If the agent needs reference material, load it
into the agent's context, not the parent's. Use the context sink pattern.

**Registered in a behavior.** The agent file exists in `agents/` and is listed
in your behavior's `agents.include`:

```yaml
# behaviors/my-capability.yaml
agents:
  include:
    - my-bundle:api-reviewer
    - my-bundle:security-auditor
```

**Output format specified.** Tell the agent what to return -- structured report,
summary paragraph, list of findings. Agents that know their output format
produce dramatically better results.

## Testing Your Agent

Test agents by delegating to them from a live session:

> amplifier run --bundle ./my-bundle/bundle.md

Then in the session:

> Delegate to api-reviewer to review the endpoints in src/api/

```
[Tool: delegate] agent="my-bundle:api-reviewer", instruction="Review REST API endpoints in src/api/..."
```

Watch for these signals:

**Good signs:**
- Agent reads relevant files systematically
- Response follows the format you specified
- Findings are specific and actionable
- Token usage is reasonable (not reading the entire repo)

**Warning signs:**
- Agent reads unrelated files (instructions too vague)
- Response is generic advice (system prompt lacks specificity)
- Agent asks clarifying questions instead of working (missing context)
- Delegation fails entirely (agent not registered in behavior)

### Iterating on Descriptions

If the orchestrator doesn't select your agent when it should, the description
needs work. Test by describing scenarios and checking which agent gets selected:

> Review the API code quality in src/api/

If `foundation:explorer` fires instead of your `api-reviewer`, your description
doesn't signal clearly enough that API review is this agent's domain. Add more
specific trigger conditions.

## Practical Examples

### A Focused Agent (Good)

```markdown
---
meta:
  name: migration-planner
  description: >
    Plans database migration strategies for schema changes. Use when adding
    columns, changing types, splitting tables, or migrating data between
    schemas. Evaluates migration safety (can it run without downtime?),
    generates migration scripts, and identifies rollback strategies.
  model_role: reasoning
---

# Migration Planner

You plan safe database migrations. For every schema change:

1. Assess if the migration can run without downtime
2. Generate forward migration SQL
3. Generate rollback migration SQL
4. Identify data integrity risks
5. Recommend a deployment sequence

Always check for: foreign key dependencies, index impact on large tables,
default value implications, and NOT NULL constraints on populated columns.
```

### A Diffuse Agent (Bad)

```markdown
---
meta:
  name: helper
  description: "Helps with various development tasks."
---

# Helper

You help with development. Do whatever is asked.
```

This agent will never be selected -- its description is indistinguishable from
the base session itself. No clear domain, no trigger conditions, no expected
output. Every field is an opportunity wasted.

## Key Takeaways

1. **Agents ARE bundles** -- same file format, `meta:` instead of `bundle:` in
   frontmatter. No new format to learn.

2. **The description is everything** -- it determines selection, timing, and
   instruction quality. Answer WHAT, WHEN, WHY, HOW.

3. **Context sink pattern** -- load heavy reference material into the agent via
   `@mentions`, keeping your main session lean.

4. **One agent, one job** -- sharp specialists outperform generalists. If you
   can't state the purpose in one sentence, split into multiple agents.

5. **Self-contained instructions** -- the Markdown body should tell the agent
   everything: goals, constraints, output format. Don't rely on inherited context.

6. **Test by delegating** -- run your bundle, delegate to the agent, and check
   that it reads the right files, follows your format, and produces actionable
   output.
