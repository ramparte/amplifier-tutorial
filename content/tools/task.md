---
id: tool-task
type: tools
title: "Task Tool"
---

# Task Tool (Sub-Agents)

The Task Tool is Amplifier's delegation system that allows the main agent to spawn specialized sub-agents to handle complex, multi-step tasks autonomously. Each sub-agent operates independently with its own set of tools and expertise.

## What is Delegation?

Delegation in Amplifier means launching a specialized agent to handle a specific task. Instead of doing everything itself, the main agent can:

- **Spawn** a sub-agent with a specific instruction
- **Wait** for the sub-agent to complete its work autonomously
- **Receive** a final report with the results
- **Continue** with the next steps based on those results

Think of it like assigning work to a specialist colleague who will handle the task independently and report back when done.

**Key Characteristics:**
- Sub-agents are **stateless** - they run once and return results
- Sub-agents work **autonomously** - no back-and-forth during execution
- Sub-agents have **specialized tools** - each type has different capabilities
- Multiple sub-agents can run **in parallel** for faster completion

## When to Delegate

Delegation is powerful but not always necessary. Use the task tool when:

### ✅ ALWAYS Delegate

**Git Operations** - ALWAYS use `foundation:git-ops`:
```
- Creating branches
- Committing changes
- Creating pull requests
- Checking git status/history
- Merging branches
- Resolving conflicts
```

**Large Codebase Exploration** - Use `foundation:explorer`:
```
- Understanding unfamiliar codebases
- Finding architectural patterns
- Mapping dependencies
- Locating specific functionality across many files
```

**Complex Debugging** - Use `foundation:bug-hunter`:
```
- Investigating error messages
- Tracing execution flows
- Finding root causes of failures
- Analyzing test failures
```

### ✅ Often Beneficial

**Design & Review** - Use `foundation:zen-architect`:
```
- Reviewing code architecture
- Suggesting improvements
- Design pattern recommendations
- Code quality assessments
```

**Modular Implementation** - Use `foundation:modular-builder`:
```
- Building new features from scratch
- Creating multiple related files
- Implementing complex components
```

**Testing** - Use `foundation:test-coverage`:
```
- Writing comprehensive test suites
- Achieving high coverage
- Test strategy planning
```

**Security** - Use `foundation:security-guardian`:
```
- Security vulnerability scans
- Dependency audits
- Best practice checks
```

### ❌ DON'T Delegate

**Simple File Operations:**
- Reading 1-2 specific files → Use `read_file` directly
- Writing a single file → Use `write_file` directly
- Quick edits → Use `edit_file` directly

**Quick Searches:**
- Finding a specific class → Use `grep` directly
- Searching known file paths → Use `glob` directly

**Simple Commands:**
- Running tests → Use `bash` directly
- Installing packages → Use `bash` directly

## Available Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `foundation:explorer` | Codebase reconnaissance | Understanding unfamiliar code, finding patterns, mapping architecture |
| `foundation:git-ops` | Git operations | **ALWAYS** for commits, branches, PRs, merges, git history |
| `foundation:bug-hunter` | Debugging & investigation | Tracing errors, finding bugs, analyzing failures |
| `foundation:zen-architect` | Design & code review | Architecture review, design patterns, quality assessment |
| `foundation:modular-builder` | Feature implementation | Building new features, creating multiple files |
| `foundation:test-coverage` | Testing | Writing tests, improving coverage, test strategy |
| `foundation:security-guardian` | Security checks | Vulnerability scans, security audits |
| `foundation:file-ops` | Complex file operations | Batch file operations, directory restructuring |
| `foundation:web-research` | External research | Fetching docs, researching libraries, finding examples |
| `foundation:integration-specialist` | Integration tasks | Connecting systems, API integration |
| `foundation:ecosystem-expert` | Ecosystem knowledge | Tool recommendations, best practices |

## Parallel Execution

**One of the most powerful features** is running multiple agents simultaneously. This dramatically speeds up complex workflows.

### How to Run in Parallel

Send **a single message** with **multiple task tool calls**:

```
User: "Analyze the auth module - check architecture, tests, and security"

Agent: I'll launch three agents in parallel to analyze different aspects:
- foundation:explorer for architecture
- foundation:test-coverage for tests
- foundation:security-guardian for security

[Makes 3 task tool calls in one message]
```

### Benefits

- **Speed**: 3 tasks that take 2 minutes each = 2 minutes total (not 6)
- **Efficiency**: Better resource utilization
- **Independence**: Each agent works autonomously without blocking others

### Example Scenarios

**Feature Development:**
```
1. foundation:explorer - Understand existing code
2. foundation:modular-builder - Implement feature
3. foundation:test-coverage - Write tests

All three can run in parallel!
```

**Code Quality Check:**
```
1. foundation:zen-architect - Review architecture
2. foundation:security-guardian - Security audit
3. foundation:bug-hunter - Find potential bugs

Launch all simultaneously for fast assessment!
```

## Session Resumption

Sub-agents typically run once and complete. However, if you need to continue working with a previous agent session:

### Using session_id

When an agent finishes, it returns a `session_id`:

```json
{
  "session_id": "recipe_20251118_143022_a3f2",
  "result": "Analysis complete..."
}
```

To resume this session:

```xml
<invoke name="task">
  <parameter name="agent">foundation:explorer</parameter>
  <parameter name="session_id">recipe_20251118_143022_a3f2</parameter>
  <parameter name="instruction">Now explore the payment module as well</parameter>
</invoke>
```

### When to Resume

- **Iterative exploration**: Continue exploring related areas
- **Follow-up questions**: Ask for more details about previous findings
- **Extending work**: Build on what was already discovered

### Note

Most workflows don't need resumption - launching a fresh agent with comprehensive instructions is usually simpler and more reliable.

## Best Practices

### 1. Write Detailed Instructions

❌ **Bad**: "Check the auth code"

✅ **Good**: "Explore the authentication module in src/auth/. Document the authentication flow, identify all user-facing endpoints, check for security best practices (password hashing, token management), and list any technical debt or improvement opportunities. Return a structured report with findings."

### 2. Specify What to Return

Always tell the agent **exactly what information** you need back:

```
"Return a JSON structure with:
- List of security vulnerabilities found
- Severity rating for each
- Recommended fixes
- Files affected"
```

### 3. Clarify Intent (Code vs Research)

Tell the agent if you want:
- **Research only**: "Investigate and report findings"
- **Code changes**: "Implement the feature and write tests"
- **Both**: "Research the best approach, then implement it"

### 4. Use Parallel Execution Aggressively

Don't serialize tasks that could run in parallel:

❌ **Bad**: Launch explorer, wait, then launch test-coverage, wait, then launch security
✅ **Good**: Launch all three in one message

### 5. Git Operations = Always Delegate

Never use `bash` for git commands. **Always** use `foundation:git-ops`:

❌ `bash("git commit -m 'Update'")` 
✅ `task(agent="foundation:git-ops", instruction="Commit changes with message 'Update'")`

### 6. Trust Agent Results

Agent outputs should generally be trusted. They have specialized tools and expertise for their domain.

### 7. Provide Context

Include relevant information in your instruction:
```
"The user wants to add OAuth2 authentication. 
Current auth uses simple JWT tokens in src/auth/.
Target: Add Google and GitHub OAuth providers.
Check existing OAuth libraries we're already using."
```

## Try It Yourself

### Exercise 1: Simple Delegation

Try delegating a git status check:

```
Agent: Can you check what git status shows?

You: [Use task tool with foundation:git-ops]
```

### Exercise 2: Parallel Exploration

Explore both frontend and backend architecture simultaneously:

```
Agent: I need to understand both our React frontend and Python backend

You: [Launch two explorer agents in parallel]
```

### Exercise 3: Full Workflow

Implement a new feature end-to-end:

```
1. Explorer - Understand existing code
2. Zen-architect - Design the feature
3. Modular-builder - Implement it
4. Test-coverage - Write tests
5. Git-ops - Commit changes

Try running some of these in parallel!
```

## Common Errors

### "Session spawning not available"

**Cause**: The task tool requires session spawning capability which isn't available in all environments.

**Solution**: This is an environment limitation. Sub-agents work in the main Amplifier interface.

### "Agent not found"

**Cause**: Invalid agent name or typo.

**Solution**: Use exact agent names from the Available Agents table (e.g., `foundation:git-ops`, not `git-ops`).

### "Task timeout"

**Cause**: Agent task took too long or got stuck.

**Solution**: 
- Break down into smaller tasks
- Provide more specific instructions
- Check if the agent has access to required resources

### Agent Returns Empty Result

**Cause**: Instruction was too vague or agent couldn't find what was requested.

**Solution**: 
- Be more specific about what to look for
- Provide file paths or starting points
- Check if the target code/files actually exist

## Summary

The Task Tool enables:

- ✅ **Specialization** - Right agent for each job
- ✅ **Autonomy** - Agents work independently
- ✅ **Parallelism** - Multiple tasks simultaneously
- ✅ **Efficiency** - Complex workflows completed faster

**Remember**: Delegate complex multi-step tasks, handle simple operations directly.

**Golden Rule**: Git operations ALWAYS use `foundation:git-ops`!
