---
id: modules
type: concepts
title: "Modules"
---

# Modules

Modules are the building blocks of Amplifier. Each module provides a specific capability that can be loaded, configured, and swapped.

## Module Types

Amplifier has five types of modules:

| Type | Purpose | Example |
|------|---------|---------|
| **Provider** | AI backend | Anthropic Claude, OpenAI GPT, Ollama |
| **Tool** | Capability | Read files, run bash, search web |
| **Hook** | Observer/modifier | Logging, redaction, approval |
| **Orchestrator** | Execution strategy | Basic loop, streaming, events |
| **Context** | Memory management | Simple (in-memory), persistent |

## Providers

Providers connect Amplifier to AI models:

```bash
# List available providers
amplifier provider list

# Switch providers
amplifier provider use anthropic
amplifier provider use openai
amplifier provider use ollama
```

### Available Providers

| Provider | Models | Use Case |
|----------|--------|----------|
| `anthropic` | Claude 3.5, Claude 3 | Best for coding (recommended) |
| `openai` | GPT-4, GPT-4o | Alternative, good general use |
| `azure-openai` | Azure-hosted GPT | Enterprise compliance |
| `ollama` | Llama, Mistral, etc. | Local, private, free |
| `gemini` | Gemini Pro | Google ecosystem |

### Provider Configuration

```yaml
# In settings.yaml or bundle
providers:
  - module: provider-anthropic
    config:
      model: claude-sonnet-4-20250514
      max_tokens: 8192
```

## Tools

Tools are capabilities the AI can invoke:

```bash
# In a session, list tools
/tools
```

### Core Tools

| Tool | What It Does |
|------|--------------|
| `read_file` | Read file contents |
| `write_file` | Create or overwrite files |
| `edit_file` | Make targeted string replacements |
| `bash` | Execute shell commands |
| `grep` | Search file contents with regex |
| `glob` | Find files by pattern |
| `web_search` | Search the internet |
| `web_fetch` | Fetch content from URLs |
| `task` | Spawn sub-agents |
| `todo` | Manage task lists |

### Tool Contract

Every tool implements this interface:

```python
class Tool(Protocol):
    name: str                    # Unique identifier
    description: str             # What the tool does (shown to AI)
    input_schema: dict          # JSON Schema for parameters
    
    async def execute(self, input: dict) -> str:
        """Execute the tool and return result."""
        ...
```

### Adding Tools

In a bundle:

```yaml
tools:
  # Official tool
  - module: tool-filesystem
  
  # Custom tool from local path
  - module: my-tool
    source: ./modules/my-tool
    
  # Tool with configuration
  - module: tool-bash
    config:
      allowed_commands: ["ls", "cat", "grep"]
```

## Hooks

Hooks observe and optionally modify the agent lifecycle:

### Hook Events

```
session:start → turn:start → provider:request → provider:response
                          ↓
                     tool:call → tool:result
                          ↓
              turn:end → session:end
```

### Available Hooks

| Hook | Purpose |
|------|---------|
| `hooks-logging` | Log events to JSONL file |
| `hooks-redaction` | Remove sensitive data from logs |
| `hooks-approval` | Require human approval for actions |
| `hooks-streaming-ui` | Stream responses to terminal |
| `hooks-status-context` | Inject environment info |
| `hooks-todo-reminder` | Remind about task lists |

### Hook Actions

Hooks can:

1. **Observe** - Just watch what happens (logging)
2. **Enrich** - Add data to events (status context)
3. **Modify** - Change data (redaction)
4. **Cancel** - Block an action (approval)

### Hook Configuration

```yaml
hooks:
  - module: hooks-logging
    config:
      output_dir: ./logs
      
  - module: hooks-approval
    config:
      require_approval_for:
        - bash
        - write_file
```

## Orchestrators

Orchestrators control how the AI executes:

| Orchestrator | Behavior |
|--------------|----------|
| `loop-basic` | Simple request/response loop |
| `loop-streaming` | Stream responses as they generate |
| `loop-events` | Event-driven with hooks |

### Choosing an Orchestrator

```yaml
orchestrator:
  module: loop-streaming  # For interactive use
  # module: loop-basic    # For simple scripts
  # module: loop-events   # For complex workflows
```

## Context Managers

Context managers handle conversation memory:

| Context | Behavior |
|---------|----------|
| `context-simple` | In-memory only (lost on exit) |
| `context-persistent` | Saved to disk (resumable) |

### Context Configuration

```yaml
context:
  module: context-persistent
  config:
    storage_dir: ~/.amplifier/sessions
```

## Module Resolution

When Amplifier loads a module, it searches:

1. **Bundle path** - `./modules/` relative to bundle
2. **Installed modules** - From `amplifier module install`
3. **Registry** - Official module registry

## Try It Yourself

### Exercise 1: List Your Modules

```bash
# See what's loaded
amplifier run "List all tools you have access to. Group by category."
```

### Exercise 2: Use Different Providers

```bash
# Compare responses
amplifier run "Write a haiku about coding" --provider anthropic
amplifier run "Write a haiku about coding" --provider openai
```

### Exercise 3: Explore Hooks

```bash
# Check your session log
ls ~/.amplifier/sessions/
cat ~/.amplifier/sessions/[latest]/events.jsonl | head -5
```

## Key Takeaways

1. **Modules are swappable** - Change providers without changing tools
2. **Each type has a contract** - Providers provide, tools execute, hooks observe
3. **Configuration is separate** - Same module, different config
4. **Composition through bundles** - Bundles select and configure modules

## Next

Learn about specialized AI configurations:

→ [Agents](agents.md)
