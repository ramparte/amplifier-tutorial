---
id: agents
type: concepts
title: "Agents"
---

# Agents

Agents are specialized AI configurations designed for specific tasks. When you need debugging expertise, you spawn the bug-hunter. When you need architecture advice, you spawn the zen-architect.

## What is an Agent?

An agent is a **sub-session** with its own:

- **Instructions** - Specialized system prompt
- **Tools** - Subset of available tools
- **Context** - Domain knowledge
- **Behavior** - How it approaches problems

Agents run as separate sessions, do their work, and report back.

## Using Agents

### Automatic Delegation

Amplifier automatically delegates to specialists:

```
You: Debug the authentication issue in src/auth.py

Amplifier: I'll delegate this to the bug-hunter agent...

[bug-hunter runs, investigates, reports back]

Amplifier: The bug-hunter found the issue: [summary]
```

### Manual Delegation

Request a specific agent:

```
You: Use the zen-architect to review this module's design

You: Ask the security-guardian to check src/api.py
```

### Via Task Tool

The AI uses the `task` tool to spawn agents:

```
[Tool: task]
agent: foundation:bug-hunter
instruction: "Debug the authentication failure in src/auth.py"
```

## Built-in Agents

### Foundation Agents

| Agent | Specialty | When to Use |
|-------|-----------|-------------|
| `zen-architect` | Design & architecture | Planning, code review, refactoring |
| `bug-hunter` | Debugging | Errors, test failures, unexpected behavior |
| `modular-builder` | Implementation | Building from specifications |
| `explorer` | Reconnaissance | Understanding codebases, mapping structure |
| `security-guardian` | Security review | Vulnerability assessment, auth review |
| `test-coverage` | Testing strategy | Identifying test gaps, writing tests |
| `git-ops` | Git operations | Commits, PRs, branch management |
| `web-research` | Web research | Finding docs, searching for solutions |
| `file-ops` | File operations | Bulk file changes, search |

### Design Intelligence Agents

From the `design-intelligence` bundle:

| Agent | Specialty |
|-------|-----------|
| `art-director` | Visual strategy, aesthetics |
| `component-designer` | UI component design |
| `layout-architect` | Page structure, information architecture |
| `animation-choreographer` | Motion and transitions |
| `responsive-strategist` | Multi-device design |
| `voice-strategist` | Tone, microcopy, UX writing |

## Agent Definition

Agents are defined in YAML:

```yaml
# agents/my-specialist.yaml
meta:
  name: my-specialist
  description: "Expert at analyzing performance issues"

# What tools this agent can use
tools:
  - read_file
  - bash
  - grep

# Specialized instructions
instructions: |
  You are a performance analysis expert.
  
  When analyzing performance:
  1. First profile the code to identify hotspots
  2. Check for common issues (N+1 queries, missing indexes)
  3. Measure before and after any changes
  4. Provide specific, actionable recommendations
  
  Always include benchmarks with your findings.

# Additional context to load
context:
  include:
    - ./context/performance-patterns.md
```

## Agent vs Bundle

| Aspect | Agent | Bundle |
|--------|-------|--------|
| **Scope** | Single task | Full session configuration |
| **Lifetime** | Spawned, runs, returns | Active for entire session |
| **Purpose** | Specialist work | Environment setup |
| **Relationship** | Agent runs within bundle | Bundle defines available agents |

## How Agents Work

```
Main Session
    │
    ├── You: "Debug this auth issue"
    │
    ├── Main AI: "I'll delegate to bug-hunter"
    │
    ├── [Spawns sub-session]
    │   │
    │   └── bug-hunter agent
    │       ├── Reads auth.py
    │       ├── Runs tests
    │       ├── Identifies bug
    │       └── Returns findings
    │
    └── Main AI: "The bug-hunter found..."
```

Key points:
- Sub-sessions are **isolated** - they don't see your full conversation
- They receive **focused instructions** - just what they need
- They **report back** - main session synthesizes results

## Creating Custom Agents

### Step 1: Create Agent File

```yaml
# .amplifier/agents/api-reviewer.yaml
meta:
  name: api-reviewer
  description: "Reviews REST API designs for best practices"

tools:
  - read_file
  - grep
  - web_search

instructions: |
  You are an API design expert specializing in REST.
  
  When reviewing APIs:
  - Check for RESTful resource naming
  - Verify proper HTTP method usage
  - Ensure consistent error responses
  - Look for versioning strategy
  - Check authentication patterns
  
  Reference industry standards (OpenAPI, JSON:API).
```

### Step 2: Register in Bundle

```yaml
# bundle.yaml
agents:
  - path: ./agents/api-reviewer.yaml
```

### Step 3: Use It

```
You: Use api-reviewer to check our user endpoints in src/api/users.py
```

## Agent Context

Agents can include additional context files:

```yaml
# agents/database-expert.yaml
meta:
  name: database-expert
  description: "PostgreSQL and query optimization expert"

context:
  include:
    - ./context/postgres-patterns.md
    - ./context/indexing-guide.md
    - ./context/query-optimization.md

instructions: |
  You have access to PostgreSQL best practices.
  Always check for missing indexes and N+1 queries.
```

## Try It Yourself

### Exercise 1: See Available Agents

```bash
amplifier

> /agents
```

### Exercise 2: Explicitly Use an Agent

```
> Use the explorer agent to map the structure of this project
```

### Exercise 3: Watch Delegation

```
> Debug why the tests are failing

# Watch the output - you'll see it delegate to bug-hunter
```

## Key Takeaways

1. **Agents are specialists** - Each has deep expertise in one area
2. **Delegation is automatic** - Amplifier routes to the right agent
3. **Agents are sub-sessions** - They run, return results, and end
4. **You can create your own** - Define YAML files with instructions and context

## Next

Learn about multi-step workflows:

→ [Recipes](recipes.md)
