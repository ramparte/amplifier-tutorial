---
id: foundation
type: bundles
title: "Foundation Bundle"
---

# Foundation Bundle

## Overview

The Foundation bundle is the default, batteries-included bundle for Amplifier. It provides a comprehensive set of tools and specialized agents that cover the most common software development workflows. When you run Amplifier without specifying a bundle, you get Foundation.

Foundation follows the "thin bundle" pattern—it composes behaviors from smaller, focused modules rather than implementing everything itself. This means you get a powerful, well-integrated experience out of the box while maintaining the flexibility to customize or extend as needed.

### Design Philosophy

Foundation embodies several key principles:

- **Completeness without bloat**: Everything you need for day-to-day development, nothing you don't
- **Specialized agents**: Each agent does one thing exceptionally well
- **Composable workflows**: Agents can delegate to each other for complex tasks
- **Sensible defaults**: Works great out of the box, customizable when needed

## Tools Included

Foundation provides access to core tools that enable file operations, search, shell access, and more.

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `bash` | Shell command execution | Running builds, tests, git commands, system utilities |
| `read_file` | Read file contents | Viewing source code, configs, documentation |
| `write_file` | Create or overwrite files | Creating new files when necessary |
| `edit_file` | Surgical file edits | Making precise changes to existing files |
| `glob` | Find files by pattern | Locating files matching `**/*.py`, `src/**/*.ts` |
| `grep` | Search file contents | Finding code patterns, function definitions |
| `web_search` | Search the internet | Looking up documentation, APIs, solutions |
| `web_fetch` | Fetch URL content | Reading documentation pages, API specs |
| `task` | Launch sub-agents | Delegating specialized work to expert agents |
| `LSP` | Language Server Protocol | Code intelligence, go-to-definition, find references |
| `todo` | Task tracking | Managing multi-step workflows |
| `load_skill` | Load domain knowledge | Accessing specialized skills and patterns |

### Tool Usage Patterns

**File Operations**: Always prefer `edit_file` over `write_file` for existing files. The edit tool makes surgical changes while preserving the rest of the file.

```
# Good: Precise edit
edit_file(file_path="src/auth.py", old_string="timeout=30", new_string="timeout=60")

# Avoid: Rewriting entire file for small changes
write_file(file_path="src/auth.py", content="...")
```

**Search Operations**: Use `grep` for content search, `glob` for file discovery.

```
# Find all Python files
glob(pattern="**/*.py")

# Find function definitions
grep(pattern="def authenticate", type="py")
```

## Agents Included

Foundation includes a roster of specialized agents, each designed for specific development tasks.

| Agent | Purpose | Key Capabilities |
|-------|---------|------------------|
| `explorer` | Codebase reconnaissance | Deep code exploration, architecture mapping |
| `zen-architect` | Design and planning | Architecture decisions, module specifications |
| `modular-builder` | Implementation | Building code from specifications |
| `bug-hunter` | Debugging | Systematic bug investigation and fixes |
| `git-ops` | Git operations | Commits, PRs, branch management |
| `test-coverage` | Testing strategy | Coverage analysis, test case suggestions |
| `web-research` | Internet research | Documentation lookup, API research |
| `security-guardian` | Security review | Vulnerability assessment, security audits |
| `integration-specialist` | External integrations | APIs, MCP servers, dependencies |
| `post-task-cleanup` | Codebase hygiene | Removing artifacts, ensuring simplicity |
| `session-analyst` | Session debugging | Analyzing and repairing Amplifier sessions |

### Agent Descriptions

#### explorer

The explorer agent performs deep reconnaissance of codebases. Use it when you need a comprehensive survey of code structure, documentation, or configuration.

**Best for:**
- Understanding unfamiliar codebases
- Mapping module dependencies
- Finding relevant files for a task

**Example:**
```
task(agent="foundation:explorer", 
     instruction="Map the authentication flow in this codebase")
```

#### zen-architect

The zen-architect embodies ruthless simplicity. It designs solutions that are as simple as possible but no simpler.

**Best for:**
- Planning new features
- Designing system architecture
- Reviewing code for unnecessary complexity

**Modes:**
- ANALYZE: Break down problems, design solutions
- ARCHITECT: System design, module specification
- REVIEW: Code quality assessment

#### modular-builder

The modular-builder implements code from specifications. It works hand-in-hand with zen-architect—architect designs, builder implements.

**Best for:**
- Implementing new features
- Building modules from specs
- Creating self-contained components

#### bug-hunter

Bug-hunter uses hypothesis-driven debugging to systematically track down issues.

**Best for:**
- Investigating errors and exceptions
- Debugging test failures
- Tracing unexpected behavior

**Example:**
```
task(agent="foundation:bug-hunter",
     instruction="The API returns 500 on /users endpoint. Find and fix the cause.")
```

#### git-ops

Git-ops handles all version control operations with safety protocols and proper formatting.

**Best for:**
- Creating commits with proper messages
- Managing branches
- Creating and managing PRs
- GitHub API operations

**Important:** Always delegate git operations to this agent rather than using bash directly.

## Getting Started

### Basic Usage

Foundation is the default bundle. Simply run Amplifier:

```bash
amp
```

You're immediately working with Foundation's full toolkit.

### Explicit Bundle Selection

To explicitly use Foundation (useful in configurations):

```yaml
# .amplifier/config.yaml
bundle: foundation
```

### First Steps

1. **Explore a codebase**: Ask about the project structure
2. **Make changes**: Request file edits or new features
3. **Run commands**: Execute tests, builds, or other shell commands
4. **Search**: Find files or code patterns

### Example Session

```
User: What does this project do?

[Amplifier explores the codebase using explorer agent]

User: Add input validation to the login endpoint

[Amplifier uses zen-architect to plan, then modular-builder to implement]

User: Run the tests

[Amplifier executes pytest via bash tool]

User: Commit the changes

[Amplifier delegates to git-ops for proper commit formatting]
```

## When to Use

### Foundation is Ideal For

- **General development work**: Day-to-day coding, debugging, refactoring
- **Learning new codebases**: Explorer agent excels at reconnaissance
- **Full-stack projects**: Web, backend, CLI—Foundation handles it all
- **Projects without specialized needs**: When you don't need domain-specific agents

### Consider Alternatives When

- **Design-heavy work**: Add the `design-intelligence` bundle for UI/UX
- **Python-specific LSP needs**: Add `lsp-python` for enhanced Python intelligence
- **Recipe-based workflows**: Add `recipes` for multi-step automation
- **Memory persistence**: Add `dev-memory` for cross-session recall

### Bundle Composition

Foundation composes well with other bundles:

```yaml
# .amplifier/config.yaml
bundles:
  - foundation
  - lsp-python
  - dev-memory
```

This gives you Foundation's core capabilities plus Python LSP and persistent memory.

## Agent Workflows

### The Architect-Builder Pattern

For significant features, Foundation encourages a two-phase approach:

1. **Design Phase**: zen-architect analyzes requirements and creates specifications
2. **Build Phase**: modular-builder implements from those specifications

This separation ensures thoughtful design before implementation.

### The Debug Workflow

When encountering bugs:

1. **bug-hunter** investigates systematically
2. Uses hypothesis-driven debugging
3. Proposes and implements fixes
4. Validates the fix works

### The Cleanup Pattern

After completing major work:

1. **post-task-cleanup** reviews changes
2. Removes temporary artifacts
3. Ensures adherence to simplicity principles
4. Validates codebase hygiene

## Configuration Options

### Provider Settings

Foundation works with any configured LLM provider:

```yaml
# .amplifier/config.yaml
provider: anthropic
model: claude-sonnet-4-20250514
```

### Tool Restrictions

Disable specific tools if needed:

```yaml
# .amplifier/config.yaml
disabled_tools:
  - web_search
  - web_fetch
```

### Agent Customization

Override agent behavior with custom instructions:

```yaml
# .amplifier/config.yaml
agent_instructions:
  zen-architect: "Always prefer functional programming patterns"
```

## Try It Yourself

### Exercise 1: Codebase Exploration

Open a project you're unfamiliar with and ask:

```
Explore this codebase and explain its architecture.
What are the main components and how do they interact?
```

Watch how Foundation's explorer agent systematically maps the project.

### Exercise 2: Feature Implementation

In a project, request a new feature:

```
Add a rate limiting middleware to the API.
Limit requests to 100 per minute per IP.
```

Observe the architect-builder workflow in action.

### Exercise 3: Bug Investigation

If you have failing tests:

```
The authentication tests are failing. Investigate and fix the issue.
```

See bug-hunter's hypothesis-driven approach.

### Exercise 4: Git Workflow

After making changes:

```
Create a commit for these changes with a proper message.
Then create a PR with a description of what was changed.
```

Notice how git-ops handles formatting and safety.

### Exercise 5: Security Review

Before deploying:

```
Review the authentication module for security vulnerabilities.
Check for OWASP Top 10 issues.
```

Watch security-guardian perform a systematic audit.

## Best Practices

### Let Agents Specialize

Don't try to do everything in one prompt. Leverage specialized agents:

- Design questions → zen-architect
- Implementation → modular-builder
- Bugs → bug-hunter
- Git → git-ops

### Trust the Workflow

Foundation's agents coordinate effectively. Let zen-architect plan before modular-builder implements. This produces better results than rushing to code.

### Use Task Tracking

For complex work, the todo tool helps track progress:

```
Create a todo list for implementing user authentication.
```

This keeps multi-step work organized and visible.

### Embrace Simplicity

Foundation's philosophy is ruthless simplicity. When solutions seem complex, ask:

```
Is there a simpler way to achieve this?
```

The zen-architect excels at finding minimal solutions.

## Summary

Foundation is your everyday development companion. It provides:

- **Complete tooling**: File ops, search, shell, web, LSP
- **Expert agents**: Specialized for design, implementation, debugging, and more
- **Sensible workflows**: Architect-builder pattern, systematic debugging
- **Extensibility**: Composes with other bundles for specialized needs

Start with Foundation. Extend when you need more. That's the Amplifier way.
