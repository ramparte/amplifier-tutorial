---
id: tools-index
type: section-index
title: Tools Reference
---

# Tools Reference

Tools are the primary way Amplifier agents interact with the world. Each tool provides a specific capability—from reading files to executing commands to searching the web. Understanding tools is essential for building effective agents.

This section covers all built-in tools, their parameters, and best practices for tool usage.

## Section Contents

| Page | Description |
|------|-------------|
| [File Operations](./file-operations.md) | Reading, writing, and editing files |
| [Search Tools](./search-tools.md) | Grep, glob, and content discovery |
| [Shell Execution](./shell-execution.md) | Running bash commands safely |
| [Web Tools](./web-tools.md) | Fetching URLs and web search |
| [LSP Integration](./lsp-integration.md) | Language Server Protocol for code intelligence |
| [Task Delegation](./task-delegation.md) | Spawning sub-agents for complex work |
| [Todo Management](./todo-management.md) | Tracking multi-step tasks |
| [Custom Tools](./custom-tools.md) | Creating your own tools |

## Quick Tips

- **Prefer specialized tools over bash** — Use `read_file` instead of `cat`, `edit_file` instead of `sed`
- **Parallel calls** — Independent tool calls can be made simultaneously for efficiency
- **Error handling** — Tools return structured errors; always check for failure conditions
- **Context limits** — Large file reads are automatically truncated; use pagination for big files
- **LSP vs grep** — Use LSP for semantic code understanding, grep for text pattern matching

## Tool Categories

| Category | Tools | Use Case |
|----------|-------|----------|
| File I/O | read_file, write_file, edit_file | Direct file manipulation |
| Search | grep, glob | Finding files and content |
| Execution | bash | System commands and scripts |
| Web | web_fetch, web_search | External information |
| Code Intel | LSP operations | Semantic code navigation |
| Orchestration | task, todo | Multi-step coordination |

## Where to Start

**New to Amplifier?** Begin with [File Operations](./file-operations.md) — it's the most commonly used tool and establishes patterns you'll use everywhere.

**Building agents?** Jump to [Task Delegation](./task-delegation.md) to understand how agents can spawn sub-agents for complex workflows.

**Coming from other AI tools?** Check [Shell Execution](./shell-execution.md) to understand Amplifier's safety guardrails and execution model.

## Common Patterns

```yaml
# Reading before editing (required pattern)
1. read_file → understand current content
2. edit_file → make precise changes

# Search then navigate
1. grep → find relevant files
2. read_file → examine matches
3. LSP → understand code structure
```

## Related Sections

- [Concepts: Tool Architecture](../concepts/tool-architecture.md)
- [Advanced: Custom Tool Development](../advanced/custom-tools.md)
- [Bundles: Tool Bundles](../bundles/tool-bundles.md)
