---
id: agents
type: concept
title: "Agents"
---

# Agents

You've been using Amplifier as a single conversation -- you type, it responds, tools
fire. But what happens when the task outgrows one context window? That's where agents
come in: specialized sub-sessions that work independently and report back with
distilled results.

## What is an Agent?

**Agents are bundles.** Same Markdown file, same YAML frontmatter. There is no special
"agent format." The only difference is a frontmatter convention: bundles use a `bundle:`
key, agents use a `meta:` key with a name and description.

```markdown
---
meta:
  name: code-reviewer
  description: "Reviews code for quality, security, and maintainability issues."
---

# Code Reviewer

You are a code review specialist. Analyze code for:
- Security vulnerabilities
- Performance issues
- Best practice violations

Return a structured report with findings, severity levels, and fix suggestions.
```

That's a complete agent. It lives in `agents/code-reviewer.md` inside a bundle and
becomes available as `my-bundle:code-reviewer`. The Markdown body becomes the agent's
system prompt -- its personality and operating instructions.

| File | Frontmatter Key | Purpose |
|------|----------------|---------|
| `bundle.md` | `bundle:` with name, version, description | Session configuration |
| `agents/explorer.md` | `meta:` with name, description | Sub-session persona |

## How It Works

When Amplifier delegates to an agent, three things happen in sequence:

1. The `delegate` tool fires and **looks up the agent config** by name
2. It calls `session.spawn()`, creating a **new AmplifierSession** with merged config from the parent bundle and the agent definition, linked back via `parent_id`
3. The agent **works autonomously** -- reading files, running searches, calling tools -- then returns a final summary to your session

```
You: "Explore the auth module"
        |
  [delegate tool] → session.spawn()
        |
  New AmplifierSession (parent_id = yours)
        |
  Agent works: [read_file] [grep] [glob] ...
  (15-30 tool calls, ~20,000 tokens consumed)
        |
  Returns distilled summary (~500 tokens)
        |
  Your session continues with the summary
```

### The Context Sink Pattern

This is THE key architectural insight. When the explorer agent investigates your
auth module, it might read 40 files and build up 20,000 tokens of working context.
But your session only receives the final summary -- maybe 500 tokens.

The agent **absorbs the token cost of exploration** so your main session stays lean.
Without delegation, those 20,000 tokens consume your context window. With delegation,
you pay only for the summary.

```
Without agents:         With agents:

Your context:           Your context:
  [exploration x40]       [500-token summary]
  [20,000 tokens]
  ... running low         Agent's context (discarded):
                            [20,000 tokens of work]
```

Agents act as sinks: they do the expensive exploration, compress it, and free
your context budget for the actual work.

## Using Agents

Agents are invoked through the `delegate` tool with an agent name and instruction.

> Explore the authentication module in src/auth/ and explain how login flows work.

[Tool: delegate] agent="foundation:explorer", instruction="Explore src/auth/. Map the login flow end-to-end, identify entry points, middleware, and token handling."

### Context Control

You control what the agent knows about your conversation with two parameters:

**context_depth** -- how much history:

| Value | Agent Sees | Use When |
|-------|-----------|----------|
| `none` | Clean slate, just the instruction | Independent tasks (default, best choice) |
| `recent` | Last N turns | Follow-up tasks needing recent context |
| `all` | Full history | Tasks needing complete background |

**context_scope** -- what kinds of content:

| Value | Includes | Use When |
|-------|---------|----------|
| `conversation` | Text messages only | Lightweight, most common |
| `agents` | Text + previous agent results | Building on earlier agent work |
| `full` | Text + agent results + all tool outputs | Maximum context, rare |

Most of the time, `none` depth is correct. Agents with clean slates and clear
instructions outperform agents given messy conversation history.

### Parallel Dispatch

The real power shows when you dispatch multiple agents simultaneously. Say you just
cloned an unfamiliar repo:

> Give me a full assessment -- architecture, test coverage, and security.

Amplifier dispatches three agents in parallel:

[Tool: delegate] agent="foundation:explorer", instruction="Map the architecture. Identify main modules, responsibilities, key abstractions, and how they connect."

[Tool: delegate] agent="foundation:test-coverage", instruction="Analyze the test suite. Report coverage gaps, test quality, and which modules lack tests."

[Tool: delegate] agent="foundation:security-guardian", instruction="Security audit. Check for hardcoded secrets, injection vulnerabilities, auth weaknesses, and dependency issues."

All three work concurrently. Three two-minute tasks finish in two minutes total,
not six. Your session receives three focused summaries and synthesizes the
overall assessment.

### Multi-Agent Patterns

**Parallel dispatch** -- independent tasks run simultaneously (the example above).

**Chain with accumulated knowledge** -- one agent's output feeds the next:

1. `explorer` maps the codebase → architectural summary
2. `zen-architect` designs a feature → spec (informed by summary)
3. `modular-builder` implements → code (informed by spec)
4. `test-coverage` writes tests → suite (informed by code)

Each agent receives previous results via `agents` context scope.

**Session resumption** -- pass a previous agent's `session_id` back to delegate.
The agent picks up with its full context intact -- useful for iterative exploration.

## Foundation Agents

Amplifier Foundation ships with specialized agents for common tasks:

| Agent | What It Does |
|-------|-------------|
| `foundation:explorer` | Deep codebase reconnaissance -- maps architecture, traces flows |
| `foundation:zen-architect` | Architecture design, code review, design pattern guidance |
| `foundation:modular-builder` | Implements features from specifications across multiple files |
| `foundation:bug-hunter` | Systematic debugging -- traces errors, finds root causes |
| `foundation:git-ops` | Git and GitHub -- commits, PRs, branches, merges |
| `foundation:session-analyst` | Analyzes session history and agent interactions |
| `foundation:security-guardian` | Security audits, vulnerability detection, dependency checks |
| `foundation:web-research` | Internet research, documentation lookup, library evaluation |
| `foundation:file-ops` | Precise batch file operations, targeted edits, restructuring |
| `foundation:test-coverage` | Test analysis, coverage gaps, test strategy |

Quick selection guide: **analysis** → explorer, test-coverage, security-guardian.
**Implementation** → modular-builder, file-ops. **Design** → zen-architect.
**Debugging** → bug-hunter. **External lookups** → web-research.

## Creating Your Own

Since agents are Markdown files with `meta:` frontmatter, creating one is trivial.

**1. Create the file** in `agents/` inside your bundle:

```markdown
---
meta:
  name: api-reviewer
  description: "Reviews REST API designs for consistency, RESTfulness, and
    documentation coverage. Use when designing or auditing API endpoints."
---

# API Reviewer

You are an expert in REST API design. Evaluate:
1. **RESTfulness** -- HTTP methods, status codes, resource naming
2. **Consistency** -- naming conventions, pagination, error format
3. **Documentation** -- OpenAPI coverage, example requests/responses

Return a structured report with an overall grade (A-F), issues with severity,
and specific recommendations with code examples.
```

The `meta.description` is important -- it's shown when listing agents and helps
Amplifier decide when to use this agent. Include usage guidance in the description.

**2. Register in your behavior:**

```yaml
# behaviors/my-capability.yaml
agents:
  include:
    - my-bundle:api-reviewer
```

**3. Use it:**

> Review the API endpoints in src/api/ for REST best practices.

[Tool: delegate] agent="my-bundle:api-reviewer", instruction="Review REST API endpoints in src/api/."

For agents that need reference material, use `@mentions` in the agent's Markdown
body. The material loads into the agent's context when spawned -- not into your
main session. This is the context sink pattern: heavy references live with the
specialist.

## Best Practices

**Write self-contained instructions.** Include file paths, goals, constraints, and
output format directly in the instruction. Don't assume the agent knows your context.

**Use parallel dispatch aggressively.** If three tasks don't depend on each other,
dispatch all three at once. The wall-clock savings compound.

**Default to `none` context depth.** Clean slates with clear instructions beat
messy inherited history almost every time.

**One agent, one job.** Agents that try to do everything produce mediocre results.
The best agents are sharp specialists.

**Trust the context sink.** Delegate exploration rather than doing it in your main
session "just to see." Your context window is a budget -- spend it on synthesis
and decisions, not intermediate exploration.

## Key Takeaways

1. **Agents ARE bundles** -- same Markdown format, different frontmatter (`meta:`
   instead of `bundle:`). No new format to learn.

2. **Context sink pattern** -- agents absorb ~20,000 tokens of exploration and
   return ~500 tokens of distilled summary. This is why delegation scales.

3. **Delegation via `delegate` tool** -- control context with `context_depth`
   (none/recent/all) and `context_scope` (conversation/agents/full).

4. **Parallel dispatch is the power move** -- independent tasks run concurrently,
   cutting wall-clock time proportionally.

5. **Foundation covers common cases** -- explorer, zen-architect, modular-builder,
   bug-hunter, git-ops, security-guardian, test-coverage, and more ship ready to use.

6. **Creating agents is trivial** -- a Markdown file with `meta:` frontmatter,
   registered in a behavior's `agents.include`.

7. **Self-contained instructions win** -- agents work best with clear, complete
   instructions rather than inherited conversation context.
