---
id: tool-task
type: tools
title: "Task Tool"
---

# Task Tool (Sub-Agents)

The Task tool is Amplifier's delegation system. It launches specialized agents to handle
complex tasks autonomously, allowing you to break down sophisticated workflows into
manageable, parallelizable units of work.

## What is Delegation?

Delegation means spawning a sub-agent with its own context, tools, and focus area.
Think of it as hiring a specialist:

```
You (main agent) --> Task tool --> Sub-agent (specialist)
                                   |
                                   v
                              Works autonomously
                                   |
                                   v
                              Returns final report
```

Key characteristics:

- **Stateless invocations**: Each agent call starts fresh with no memory of previous calls
- **Autonomous execution**: Sub-agents work independently until completion
- **Single response**: You receive one final report, not a conversation
- **Specialized tools**: Each agent type has access to different tool sets

The main agent remains focused on orchestration while specialists handle domain-specific work.

## When to Delegate

Delegation is appropriate when a task requires:

### Exploration Tasks

When you need to understand a codebase, find patterns, or gather information:

```
Use foundation:explorer when:
- Mapping module dependencies
- Finding all usages of a pattern
- Understanding system architecture
- Gathering context across many files
```

### Git Operations (ALWAYS Delegate)

**Critical rule**: Never use bash directly for git commands. Always delegate to git-ops:

```
Use foundation:git-ops for:
- Creating commits (generates proper messages)
- Branch operations
- Creating/managing PRs
- Conflict resolution
- Multi-repo sync
```

The git-ops agent has safety protocols and follows commit message standards.

### Debugging

When encountering errors, test failures, or unexpected behavior:

```
Use foundation:bug-hunter when:
- User reports an error
- Tests are failing
- Behavior doesn't match expectations
- You need hypothesis-driven investigation
```

### Large Context Tasks

When the work requires more context than fits in a single conversation:

```
Delegate when:
- Refactoring spans many files
- Analysis requires reading extensive code
- The task has multiple independent phases
```

### When NOT to Delegate

Avoid the task tool for:

- Reading a specific file (use read_file directly)
- Finding a class definition (use glob or grep)
- Simple file edits (use edit_file directly)
- Tasks requiring back-and-forth conversation

## Available Agents

| Agent | Purpose | Key Capabilities |
|-------|---------|------------------|
| `foundation:explorer` | Codebase reconnaissance | Deep file analysis, pattern discovery |
| `foundation:git-ops` | Git and GitHub operations | Commits, PRs, branches, safety checks |
| `foundation:bug-hunter` | Systematic debugging | Hypothesis-driven investigation |
| `foundation:zen-architect` | Design and review | Architecture, specifications, code review |
| `foundation:modular-builder` | Implementation | Build from specifications |
| `foundation:web-research` | External information | Web search, documentation lookup |
| `foundation:test-coverage` | Test analysis | Coverage gaps, test suggestions |
| `foundation:security-guardian` | Security review | Vulnerability assessment, audits |
| `foundation:file-ops` | File operations | Read, write, edit, search files |
| `foundation:session-analyst` | Session debugging | Analyze/repair Amplifier sessions |

### Specialized Domain Agents

| Agent | Purpose |
|-------|---------|
| `amplifier:amplifier-expert` | Amplifier ecosystem guidance |
| `core:core-expert` | Kernel internals and protocols |
| `foundation:foundation-expert` | Bundle composition patterns |
| `lsp-python:python-code-intel` | Python semantic analysis |

## Parallel Execution

One of the Task tool's most powerful features is parallel agent execution.
Launch multiple independent agents simultaneously:

```yaml
# CORRECT: Multiple independent calls in one message block
- task(agent="foundation:explorer", instruction="Map auth module")
- task(agent="foundation:explorer", instruction="Map database layer")
- task(agent="foundation:web-research", instruction="Find JWT best practices")
```

All three agents run concurrently, dramatically reducing total execution time.

### When to Parallelize

Parallelize when tasks are **independent**:

```
Good candidates for parallel execution:
- Exploring different parts of a codebase
- Running multiple analyses
- Gathering information from different sources
- Building independent modules
```

### When NOT to Parallelize

Use sequential execution when tasks have **dependencies**:

```
Run sequentially when:
- Task B needs output from Task A
- You need to decide next steps based on results
- Operations must happen in a specific order
```

Example of dependent tasks (must be sequential):

```
1. zen-architect designs the module specification
2. modular-builder implements from that specification
3. test-coverage analyzes the implementation
```

## Session Resumption

Each task invocation returns a `session_id`. You can resume interrupted sessions:

```yaml
# Initial call
task:
  agent: foundation:explorer
  instruction: "Deep analysis of payment system"
# Returns: session_id: "abc123..."

# Resume if interrupted
task:
  session_id: "abc123..."
  instruction: "Continue from where you left off"
```

### When to Use Session Resumption

- Agent was interrupted mid-task
- You need to provide additional context to the same agent
- Long-running analysis needs continuation

### Limitations

- Sessions are stateless between invocations by default
- Resumption requires the original session_id
- Not all agents support meaningful resumption

## Best Practices

### 1. Write Detailed Instructions

Sub-agents have no context beyond what you provide. Be explicit:

```yaml
# BAD: Vague instruction
instruction: "Fix the bug"

# GOOD: Complete context
instruction: |
  Investigate the KeyError in src/auth/login.py:45.
  The error occurs when users log in with OAuth.
  Check the token validation flow and session handling.
  Return: root cause, affected code paths, and fix recommendation.
```

### 2. Specify Expected Output

Tell agents exactly what you need back:

```yaml
instruction: |
  Analyze the caching implementation.
  
  Return in your final report:
  1. List of all cache keys used
  2. TTL settings for each cache type
  3. Potential race conditions identified
  4. Recommendations for improvement
```

### 3. Choose the Right Agent

Match the task to the specialist:

| Task | Agent |
|------|-------|
| "What does this code do?" | `explorer` |
| "Commit these changes" | `git-ops` |
| "Why is this failing?" | `bug-hunter` |
| "Design a new feature" | `zen-architect` |
| "Implement this spec" | `modular-builder` |

### 4. Trust Agent Results

Agent outputs are generally reliable. Use their findings directly rather than
re-verifying everything manually.

### 5. Summarize for Users

Agent results are not shown to users automatically. Always summarize:

```
# After receiving agent report
"The explorer agent found that authentication spans 12 modules.
The main entry point is src/auth/handler.py. Key finding: 
the session manager has a potential race condition."
```

## Try It Yourself

### Exercise 1: Parallel Exploration

Ask Amplifier to explore two different areas of a codebase simultaneously:

```
"Explore both the API layer and the database models in parallel.
I need to understand how they connect."
```

Watch as two explorer agents work concurrently.

### Exercise 2: The Full Pipeline

Request a feature that uses multiple agents:

```
"Design and implement a rate limiting feature for the API.
Use zen-architect for design, then modular-builder for implementation."
```

Observe the handoff between agents.

### Exercise 3: Debugging Delegation

Report a bug and let bug-hunter investigate:

```
"The /api/users endpoint returns 500 errors intermittently.
Use bug-hunter to find the root cause."
```

## Common Errors

### "Session spawning not available"

The app layer hasn't registered session spawning capability. This typically means:
- You're in a restricted environment
- The bundle configuration doesn't include task delegation
- Check your bundle's capabilities

### "Agent not found"

The specified agent doesn't exist or isn't available:

```
# Check agent name format
agent: "collection:agent-name"

# Common mistakes:
- "explorer" (missing collection prefix)
- "foundation/explorer" (wrong separator)
- "Foundation:explorer" (case sensitive)
```

### "Instruction required"

Every task call needs an instruction:

```yaml
# This will fail
task:
  agent: foundation:explorer

# This works
task:
  agent: foundation:explorer
  instruction: "Analyze the src/ directory structure"
```

### Timeout Errors

Complex tasks may timeout. Solutions:
- Break into smaller sub-tasks
- Provide more focused instructions
- Use session resumption for long-running work

---

## Summary

The Task tool enables powerful multi-agent workflows:

- **Delegate** complex work to specialized agents
- **Parallelize** independent tasks for efficiency
- **Resume** interrupted sessions when needed
- **Trust** agent results and summarize for users

Master delegation to unlock Amplifier's full potential for sophisticated,
multi-step software engineering tasks.
