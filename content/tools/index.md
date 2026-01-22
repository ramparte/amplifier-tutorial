---
id: tools-index
type: section-index
title: Tools Reference
---

# Tools Reference

Tools are the fundamental building blocks that give Amplifier its capabilities. Each tool provides a specific function—from file operations to web searches—that agents can invoke to accomplish tasks. This section covers all available tools, their parameters, and best practices for effective use.

## Section Contents

| Page | Description |
|------|-------------|
| [File Operations](./file-operations.md) | Read, write, and edit files with precision |
| [Search Tools](./search-tools.md) | Grep and glob for finding content and files |
| [Bash Execution](./bash-execution.md) | Run shell commands safely |
| [Web Tools](./web-tools.md) | Search and fetch content from the web |
| [LSP Integration](./lsp-integration.md) | Language Server Protocol for code intelligence |
| [Task Delegation](./task-delegation.md) | Spawn agents for complex operations |
| [Todo Management](./todo-management.md) | Track multi-step task progress |
| [Skills Loader](./skills-loader.md) | Load domain knowledge on demand |

## Quick Tips

- **Prefer specialized tools over bash** - Use `read_file` instead of `cat`, `edit_file` instead of `sed`
- **Parallel execution** - Independent tool calls can run simultaneously for speed
- **Check before writing** - Always read a file before editing to understand context
- **Use glob for discovery** - Find files by pattern before targeted operations
- **Grep for content** - Search inside files with regex patterns

## Tool Categories

### File System
Core operations for interacting with the local filesystem. These are your most frequently used tools.

### Search & Discovery
Find files and content across your codebase quickly and efficiently.

### External Resources
Connect to the web, APIs, and external services when local context isn't enough.

### Code Intelligence
Semantic understanding of code through LSP—definitions, references, and type information.

### Orchestration
Manage complex workflows with task delegation and progress tracking.

## Where to Start

**New to Amplifier?** Begin with [File Operations](./file-operations.md) to understand the most common tool patterns.

**Building agents?** Jump to [Task Delegation](./task-delegation.md) to learn how agents spawn sub-agents.

**Searching code?** See [Search Tools](./search-tools.md) for grep vs glob guidance.

## Common Patterns

```yaml
# Read before edit
1. read_file → understand content
2. edit_file → make precise changes

# Find then act
1. glob → locate files
2. grep → find specific content
3. read_file → examine matches
```

## Next Steps

After mastering tools, explore [Concepts](../concepts/index.md) to understand how tools fit into the larger architecture.
