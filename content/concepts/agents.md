---
id: agents
type: concepts
title: "Understanding Agents"
---

# Understanding Agents

Agents are specialized AI assistants that handle complex, multi-step tasks autonomously. They're one of Amplifier's most powerful features for managing sophisticated workflows without overwhelming your main conversation context.

## What is an Agent?

**Key Insight: Agents ARE bundles. Same file format, same structure.**

An agent is simply a bundle that's designed to be launched as a sub-session via the `task` tool. There's no special "agent format" - any bundle can potentially act as an agent if it's configured for autonomous task execution.

```yaml
# This IS an agent - it's just a bundle!
name: code-reviewer
version: 1.0.0
description: Reviews code for quality and best practices

instructions: |
  You are a code review specialist. Analyze the provided code
  for bugs, style issues, and improvement opportunities.
  
  Provide actionable feedback with specific line references.

tools:
  - read_file
  - grep
  - glob
```

The magic isn't in the format - it's in how agents are invoked. When you call an agent via the `task` tool, Amplifier:

1. Spawns a fresh sub-session
2. Loads the agent's bundle configuration
3. Executes the task autonomously
4. Returns results to your main session

## How Agents Work

```
┌─────────────────┐         ┌─────────────────┐
│  Main Session   │  task   │   Sub-Session   │
│     (you)       │ ──────▶ │    (agent)      │
│                 │         │                 │
│ - Your context  │         │ - Fresh context │
│ - Your tools    │         │ - Agent tools   │
│ - Conversation  │         │ - Single task   │
└─────────────────┘         └─────────────────┘
        │                           │
        │         result            │
        ◀───────────────────────────┘
```

### The Sub-Session Model

Each agent runs in complete isolation:

- **Fresh context**: Agents start with no memory of your conversation
- **Scoped tools**: Only tools defined in the agent's bundle are available
- **Single mission**: Agents focus on one specific task
- **One response**: Agents return a single final result

This isolation is intentional. It means:

- Your main session stays uncluttered
- Agents can focus deeply on their specialty
- Complex tasks don't pollute your context window
- Multiple agents can run in parallel

### Information Flow

```
You: "Review the authentication module"
        │
        ▼
┌───────────────────────────────────────┐
│ task tool invocation                  │
│                                       │
│ agent: "foundation:code-reviewer"     │
│ instruction: "Review src/auth/*.py    │
│              for security issues"     │
└───────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────┐
│ Agent Sub-Session                     │
│                                       │
│ - Reads files                         │
│ - Analyzes patterns                   │
│ - Checks security practices           │
│ - Generates detailed report           │
└───────────────────────────────────────┘
        │
        ▼
Final report returned to main session
```

## Using Agents

Agents are invoked through the `task` tool. The key parameters are:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `agent` | Yes | Agent identifier (e.g., `foundation:zen-architect`) |
| `instruction` | Yes | Detailed task description |
| `session_id` | No | Resume a previous agent session |

### Writing Good Instructions

Since agents start with zero context, your instructions must be comprehensive:

```yaml
# Bad - too vague
instruction: "Fix the bug"

# Good - complete context
instruction: |
  Fix the KeyError in src/processor.py around line 45.
  
  Context:
  - The error occurs when processing empty input arrays
  - Expected behavior: return empty result, not crash
  - Related files: src/processor.py, src/validator.py
  
  Return:
  - What you changed and why
  - How to verify the fix
```

### Parallel Agent Execution

Launch multiple agents simultaneously for independent tasks:

```
┌─────────────┐
│ Main Session│
└──────┬──────┘
       │
       ├──────────▶ Agent A (code review)
       │
       ├──────────▶ Agent B (test coverage)
       │
       └──────────▶ Agent C (documentation)
```

All three run concurrently, returning results as they complete.

## Built-in Agents

Amplifier Foundation provides specialized agents for common workflows:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `foundation:zen-architect` | Architecture design & code planning | Starting new features, major refactors |
| `foundation:modular-builder` | Implementation from specs | Building code after architecture phase |
| `foundation:bug-hunter` | Systematic debugging | Tracking down errors and failures |
| `foundation:explorer` | Deep codebase reconnaissance | Understanding unfamiliar code |
| `foundation:git-ops` | Git and GitHub operations | Commits, PRs, branch management |
| `foundation:test-coverage` | Test analysis & gap identification | Ensuring adequate test coverage |
| `foundation:web-research` | Internet search & fetching | Finding external documentation |
| `foundation:security-guardian` | Security review & auditing | Pre-deployment checks |
| `foundation:integration-specialist` | External API & MCP integration | Connecting to services |
| `foundation:post-task-cleanup` | Codebase hygiene | After completing major tasks |

### Specialized Domain Agents

| Agent | Purpose |
|-------|---------|
| `lsp:code-navigator` | Semantic code navigation via LSP |
| `lsp-python:python-code-intel` | Python-specific code intelligence |
| `recipes:recipe-author` | Creating and validating recipes |
| `design-intelligence:*` | UI/UX design specialists |

## Creating Your Own Agents

Any bundle can become an agent. Here's the pattern:

### 1. Define the Specialty

```yaml
# .amplifier/agents/pr-reviewer.yaml
name: pr-reviewer
version: 1.0.0
description: Reviews pull requests for quality and completeness

instructions: |
  You are a PR review specialist. When given a PR or branch:
  
  1. Analyze the changes for:
     - Code quality and style
     - Test coverage
     - Documentation updates
     - Breaking changes
  
  2. Provide structured feedback:
     - Summary of changes
     - Issues found (critical/minor)
     - Suggestions for improvement
     - Approval recommendation

tools:
  - read_file
  - grep
  - glob
  - bash  # for git operations
```

### 2. Scope the Tools

Only include tools the agent actually needs:

```yaml
# Minimal toolset for a documentation agent
tools:
  - read_file
  - write_file
  - glob

# Richer toolset for a debugging agent  
tools:
  - read_file
  - grep
  - glob
  - bash
  - LSP
```

### 3. Write Clear Instructions

Agent instructions should specify:

- **Role**: What the agent specializes in
- **Process**: How to approach tasks
- **Output format**: What to return
- **Boundaries**: What NOT to do

### 4. Register for Discovery

Place agents where they can be found:

```
.amplifier/
  agents/
    pr-reviewer.yaml     # Project-specific
    
~/.amplifier/
  agents/
    my-helper.yaml       # Personal agents
```

## Agents vs Bundles

The distinction is about **usage pattern**, not file format:

| Aspect | Bundle (as context) | Bundle (as agent) |
|--------|---------------------|-------------------|
| Loading | Included in session | Spawns sub-session |
| Context | Shared with main | Isolated |
| Lifecycle | Persistent | Task-scoped |
| Invocation | `@mention` or auto | `task` tool |
| Response | Ongoing conversation | Single result |

### When to Use Each

**Use as Context (bundle):**
- Configuration that applies to all work
- Instructions that guide ongoing behavior
- Tool definitions needed throughout session

**Use as Agent (task):**
- Discrete, completable tasks
- Work requiring deep focus
- Parallel execution opportunities
- Tasks benefiting from isolation

### The Spectrum

```
Pure Context ◀─────────────────────────▶ Pure Agent

 AGENTS.md      Shared        Task-specific
 Project        behaviors     specialists
 config
```

Most bundles live somewhere in between, usable as either context or agents depending on the situation.

## Agent Design Patterns

### The Specialist Pattern

Create agents for specific domains:

```yaml
# Each agent is an expert in one area
security-reviewer    # Security-focused review
performance-analyst  # Performance optimization
api-designer        # API design review
```

### The Pipeline Pattern

Chain agents for complex workflows:

```
analyze ──▶ design ──▶ implement ──▶ review
```

Each stage is a separate agent, passing context forward.

### The Parallel Pattern

Fan out independent work:

```
                    ┌──▶ lint
                    │
main task ──────────┼──▶ test
                    │
                    └──▶ docs
```

Results merge back for final synthesis.

## Key Takeaways

1. **Agents ARE bundles** - Same YAML format, different invocation pattern

2. **Sub-sessions provide isolation** - Fresh context, focused execution

3. **Instructions must be complete** - Agents have no memory of your conversation

4. **Parallel execution is powerful** - Launch independent agents simultaneously

5. **Tools are scoped** - Agents only access what their bundle defines

6. **Create specialists** - Build agents for your specific workflows

7. **Choose the right pattern** - Context for ongoing guidance, agents for discrete tasks

---

## Next Steps

- [Using the Task Tool](../guides/task-tool.md) - Deep dive into agent invocation
- [Creating Custom Agents](../guides/custom-agents.md) - Build your own specialists
- [Recipe-Based Workflows](../guides/recipes.md) - Orchestrate multi-agent pipelines
