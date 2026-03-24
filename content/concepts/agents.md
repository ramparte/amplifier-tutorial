---
id: agents
type: concepts
title: "Understanding Agents"
---

# Understanding Agents

Agents are one of Amplifier's most powerful features for handling complex, multi-step
tasks. They enable delegation, parallel execution, and specialized expertise—all while
maintaining the simplicity of the bundle format you already know.

## What is an Agent?

**Key Insight: Agents ARE bundles. Same file format.**

An agent is simply a bundle that gets spawned as a sub-session to handle a specific task.
There's no special "agent format" to learn. If you can write a bundle, you can write an
agent. The only difference is how it's used:

- **Bundle**: Loaded into your current session, adds capabilities
- **Agent**: Spawned as a separate sub-session, works independently

```yaml
# This is both a valid bundle AND a valid agent
name: code-reviewer
version: "1.0.0"
description: "Reviews code for quality and best practices"

instructions:
  - role: system
    content: |
      You are a code review specialist. Analyze code for:
      - Security vulnerabilities
      - Performance issues
      - Best practice violations
      - Maintainability concerns
```

When you load this as a bundle, its instructions enhance your session.
When you spawn it as an agent, it runs independently and reports back.


## How Agents Work

Agents operate as isolated sub-sessions with their own context and tools:

```
┌─────────────────┐         ┌─────────────────┐
│  Main Session   │         │   Sub-Session   │
│     (you)       │         │    (agent)      │
│                 │         │                 │
│  ┌───────────┐  │  task   │  ┌───────────┐  │
│  │ Your      │──┼────────▶│  │ Agent     │  │
│  │ Context   │  │         │  │ Context   │  │
│  └───────────┘  │         │  └───────────┘  │
│                 │         │                 │
│  Continues...   │◀────────┼─ Final Report   │
│                 │  result │                 │
└─────────────────┘         └─────────────────┘
```

**Key characteristics:**

1. **Stateless**: Each agent invocation starts fresh with no memory of previous calls
2. **Isolated**: Agents have their own context window, separate from yours
3. **One-shot**: You send instructions, they work, they report back once
4. **Autonomous**: Agents decide how to accomplish the task you give them

The agent receives:
- Your instruction (the task to perform)
- Its own bundle configuration (instructions, tools, context)
- Access to the filesystem and configured tools

The agent returns:
- A single final report with results
- No intermediate communication (they work autonomously)


## Using Agents

Agents are invoked through the `task` tool with an instruction and agent type:

```
Use the task tool:
  agent: "foundation:code-reviewer"
  instruction: "Review the authentication module in src/auth/ for security issues"
```

### Writing Effective Instructions

Since agents work autonomously with a single instruction, clarity is essential:

**Good instruction:**
```
Analyze src/api/handlers.py for security vulnerabilities.
Focus on: input validation, SQL injection, XSS risks.
Return: A list of issues with severity, line numbers, and fix suggestions.
```

**Poor instruction:**
```
Check the code.
```

### Parallel Execution

One of the most powerful patterns is running multiple agents simultaneously:

```
# These run in parallel when called in the same message
task(agent="foundation:test-coverage", instruction="Analyze test coverage for src/auth/")
task(agent="foundation:security-guardian", instruction="Security audit of src/auth/")
task(agent="foundation:zen-architect", instruction="Review architecture of src/auth/")
```

All three agents work concurrently, dramatically reducing total time.


## Built-in Agents

Amplifier Foundation provides specialized agents for common development tasks:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `foundation:zen-architect` | Architecture design and code planning | Starting new features, system design |
| `foundation:modular-builder` | Implementation from specifications | Building code from architect specs |
| `foundation:bug-hunter` | Systematic debugging | Tracking down errors and failures |
| `foundation:explorer` | Deep codebase reconnaissance | Understanding unfamiliar code |
| `foundation:test-coverage` | Test analysis and gap identification | Ensuring adequate test coverage |
| `foundation:security-guardian` | Security audits and vulnerability detection | Pre-deployment security checks |
| `foundation:git-ops` | Git and GitHub operations | Commits, PRs, branch management |
| `foundation:web-research` | Internet research and documentation lookup | Finding external information |
| `foundation:file-ops` | Precise file operations | Batch file changes, targeted edits |
| `foundation:integration-specialist` | External service integration | API connections, MCP setup |

### Agent Selection Guidelines

Choose agents based on the task type:

- **Analysis tasks** → `explorer`, `test-coverage`, `security-guardian`
- **Implementation tasks** → `modular-builder`, `file-ops`
- **Design tasks** → `zen-architect`
- **Debugging tasks** → `bug-hunter`
- **External tasks** → `web-research`, `integration-specialist`


## Creating Your Own Agents

Since agents are just bundles, creating one is straightforward:

### Step 1: Define the Bundle

```yaml
# .amplifier/agents/my-agent.yaml
name: my-custom-agent
version: "1.0.0"
description: "Specialized agent for my domain"

instructions:
  - role: system
    content: |
      You are a specialist in [domain].
      
      Your task execution process:
      1. Analyze the instruction provided
      2. Gather necessary information using your tools
      3. Perform the requested work
      4. Return a clear, structured report
      
      Always include in your final report:
      - Summary of what was done
      - Key findings or results
      - Any issues encountered
      - Recommendations for next steps
```

### Step 2: Add Tools (Optional)

```yaml
tools:
  - tool-bash
  - tool-files
  - tool-grep
```

### Step 3: Add Context (Optional)

```yaml
context:
  - path: ./domain-knowledge.md
  - path: ./coding-standards.md
```

### Step 4: Register in Your Collection

Add to your collection's `agents` section or place in `.amplifier/agents/`.

### Agent Design Best Practices

1. **Single responsibility**: Each agent should excel at one thing
2. **Clear output contract**: Define what the agent should return
3. **Appropriate tools**: Only include tools the agent actually needs
4. **Good instructions**: Guide the agent's approach and output format
5. **Domain context**: Include relevant knowledge files


## Agents vs Bundles

Understanding when to use each:

| Aspect | Bundle | Agent |
|--------|--------|-------|
| **Loading** | Into current session | As separate sub-session |
| **Context** | Shares your context | Has its own context |
| **Interaction** | Continuous, conversational | One-shot task/report |
| **Best for** | Adding capabilities | Delegating tasks |
| **Memory** | Persists in session | Stateless per invocation |
| **Parallelism** | N/A | Can run multiple simultaneously |

### When to Use Bundles

- Adding tools and capabilities to your session
- Loading persistent context (coding standards, domain knowledge)
- Enhancing the main agent's abilities
- When you need conversational back-and-forth

### When to Use Agents

- Delegating specific, well-defined tasks
- Running multiple analyses in parallel
- Isolating complex work from your main context
- When the task can be described in a single instruction

### The Same File, Two Uses

The same YAML file can serve both purposes:

```yaml
# code-quality.yaml
name: code-quality
description: "Code quality analysis"

instructions:
  - role: system
    content: "Analyze code for quality issues..."

tools:
  - tool-files
  - tool-grep
```

- **As bundle**: `amp --bundle code-quality` → Adds quality analysis to your session
- **As agent**: `task(agent="code-quality", ...)` → Spawns quality analyzer


## Advanced Patterns

### Chained Agents

Use one agent's output as input for another:

```
1. zen-architect analyzes and creates spec
2. modular-builder implements from spec  
3. test-coverage verifies implementation
4. security-guardian audits result
```

### Specialist Teams

Create domain-specific agent teams:

```yaml
# Your collection could include:
agents:
  - api-designer      # Designs REST APIs
  - schema-validator  # Validates data schemas
  - docs-generator    # Generates documentation
```

### Context Handoff

Pass relevant context in your instruction:

```
instruction: |
  Review the following code changes:
  
  Files modified: src/auth.py, src/session.py
  PR description: Adds JWT token refresh
  Related issue: #142
  
  Focus on security implications of the token handling.
```


## Key Takeaways

1. **Agents ARE bundles** — Same YAML format, different execution model

2. **Delegation, not conversation** — Send clear instructions, get back reports

3. **Parallelism is powerful** — Run multiple agents simultaneously for speed

4. **Stateless by design** — Each invocation starts fresh

5. **Single responsibility** — Best agents do one thing exceptionally well

6. **Clear instructions matter** — Agents work autonomously from your instruction

7. **Built-in agents cover common needs** — Use Foundation agents before building custom

8. **Same file, two uses** — Any bundle can be used as an agent and vice versa

---

## Next Steps

- [Advanced Topics](../advanced/index.md) — Build custom bundles and agents
- [Bundles Guide](../bundles/index.md) — Learn about bundle composition
- [Task Tool Reference](../tools/task.md) — Complete task tool documentation
