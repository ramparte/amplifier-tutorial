---
id: tools-overview
type: tools
title: "Tools Reference"
---

# Tools Reference

Tools are capabilities that Amplifier can use to accomplish tasks.

## What Are Tools?

Tools let the AI interact with the world:

- **Read and write files**
- **Run shell commands**
- **Search the web**
- **Navigate code semantically**
- **Spawn sub-agents**

When you ask Amplifier to "create a file" or "search for X", it uses tools to do the actual work.

## Core Tools

| Tool | Purpose | Guide |
|------|---------|-------|
| [Filesystem](filesystem.md) | Read, write, edit files | `read_file`, `write_file`, `edit_file` |
| [Bash](bash.md) | Run shell commands | Build, test, git, packages |
| [Search](search.md) | Find files and content | `grep`, `glob` |
| [Web](web.md) | Internet search and fetch | `web_search`, `web_fetch` |
| [Task](task.md) | Spawn sub-agents | Delegate to specialists |

## Bundle-Provided Tools

| Tool | Bundle Required | Guide |
|------|-----------------|-------|
| [LSP](lsp.md) | `lsp-python` or similar | Code intelligence |
| [Recipes](recipes-tool.md) | `recipes` | Multi-step workflows |

## Quick Reference

### File Operations

```
read_file     Read file or list directory
write_file    Create or overwrite file
edit_file     Make targeted string replacements
```

### Shell Commands

```
bash          Run any shell command
```

### Search

```
grep          Search file contents (regex)
glob          Find files by pattern
```

### Web

```
web_search    Search the internet
web_fetch     Fetch URL content
```

### Agents

```
task          Spawn specialized sub-agents
todo          Manage task lists (internal)
```

### Code Intelligence (requires bundle)

```
LSP           Go to definition, find references, hover
```

### Workflows (requires bundle)

```
recipes       Execute multi-step YAML workflows
```

## Viewing Available Tools

In any session:

```bash
amplifier

> /tools
```

Or ask directly:

```
> What tools do you have available?
```

## Tool Selection Philosophy

Amplifier prefers specialized tools over primitives:

1. **Specialized tools first** - LSP over grep for code navigation
2. **Purpose-built tools second** - `read_file` over `bash cat`
3. **Primitives as fallback** - `bash` when nothing else fits

## Adding Custom Tools

See [Advanced: Custom Tools](../advanced/custom-tool.md) for creating your own tools.
