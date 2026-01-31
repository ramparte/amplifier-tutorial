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
| [Filesystem](./filesystem.md) | Read, write, and edit files with precision |
| [Search (grep & glob)](./search.md) | Find content and files across your codebase |
| [Bash](./bash.md) | Run shell commands safely |
| [Web Tools](./web.md) | Search and fetch content from the web |
| [LSP (Code Intelligence)](./lsp.md) | Language Server Protocol for code understanding |
| [Task (Sub-Agents)](./task.md) | Spawn agents for complex operations |
| [Recipes Tool](./recipes-tool.md) | Declarative multi-step workflows |

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

**New to Amplifier?** Begin with [Filesystem](./filesystem.md) to understand the most common tool patterns.

**Building agents?** Jump to [Task (Sub-Agents)](./task.md) to learn how agents spawn sub-agents.

**Searching code?** See [Search (grep & glob)](./search.md) for grep vs glob guidance.

## Common Patterns

### Read Before Edit
1. `read_file` → understand content
2. `edit_file` → make precise changes

### Find Then Act
1. `glob` → locate files
2. `grep` → find specific content
3. `read_file` → examine matches

## Next Steps

After mastering tools, explore [Concepts](../concepts/index.md) to understand how tools fit into the larger architecture.
