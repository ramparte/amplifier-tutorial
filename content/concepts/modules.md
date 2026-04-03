---
id: modules
type: concept
title: "Modules"
---

# Modules

## What is a Module?

A module is a swappable capability unit that plugs into the Amplifier kernel. If the kernel is the engine block, modules are everything you bolt onto it — the fuel injector (provider), the transmission (orchestrator), the dashboard instruments (hooks), the navigation system (tools), and the memory (context).

Every module follows a contract: a small interface that the kernel understands. As long as your module speaks that contract, the kernel doesn't care who wrote it, when, or why. This is what makes Amplifier modular in practice, not just in name. You can swap your LLM provider from Anthropic to OpenAI by changing one line in a YAML file. You can add file system access to your agent by mounting a tool. You can completely change how the agent loop works by swapping the orchestrator.

The kernel itself is tiny and stable. It provides **mechanisms** — module loading, event emission, session lifecycle — but makes zero **policy** decisions. Which LLM to use? That's a provider module. How to run the agent loop? That's an orchestrator module. What to log? That's a hook module. The kernel just wires them together.

## How It Works

Modules flow from configuration to runtime through a clear pipeline:

**1. Bundle declares modules.** Your bundle YAML lists which modules the session needs:

```yaml
---
bundle:
  name: my-assistant
  version: 1.0.0

tools:
  - module: tool-filesystem
  - module: tool-bash
  - module: tool-web
---
```

**2. Kernel loads and mounts them.** When a session starts, the kernel reads the bundle, discovers each module by name, calls its `mount(coordinator, config)` entry point, and registers the result in the right slot — tools go in the tool registry, hooks go in the hook registry, and so on.

**3. Session uses them.** During execution, the orchestrator drives the loop. It calls the provider for LLM completions, dispatches tool calls the LLM requests, fires events that hooks observe, and reads/writes messages through the context manager. Every step touches modules.

```
Bundle YAML ──→ Kernel loads ──→ Session uses
  (config)       (mount)          (execute)
```

The beauty of this pipeline is that modules are isolated from each other. A tool doesn't know which provider is active. A hook doesn't care which orchestrator is running. They all communicate through the kernel's contracts.

## The Five Types

Amplifier defines exactly five module types. No more, no less. Each serves a distinct architectural role.

### 1. Provider — LLM Backends

Providers connect your agent to language models. Their contract is simple: take a completion request, return a response.

```
complete(request) → response
```

That's it. Everything else — authentication, rate limiting, streaming, format translation — is the provider's internal concern. From the kernel's perspective, all providers look identical.

The ecosystem currently includes **11 providers**: Anthropic, OpenAI, Azure OpenAI, Gemini, Ollama, vLLM, GitHub Copilot, and a mock provider for testing, plus community-built providers for AWS Bedrock, Perplexity, and OpenAI Realtime. Whether you're calling Claude through an API or running Llama locally through Ollama, the rest of your agent doesn't change.

### 2. Tool — Agent Capabilities

Tools give the LLM the ability to act in the world. The critical thing about tools is that the **LLM decides** when to use them. You mount a set of tools, the LLM sees their names and descriptions, and it chooses which to call based on the conversation.

```
execute(input) → result
```

The ecosystem includes **12+ tools**: filesystem operations, bash execution, web search and fetching, code search (grep/glob), agent delegation (task), LSP code intelligence, Python quality checking, todo tracking, DOT graph analysis, MCP server integration, and more. Each one gives the agent a specific capability — reading files, running commands, searching the web — that the LLM can invoke when it decides the moment is right.

### 3. Orchestrator — The Main Engine

The orchestrator is the single most powerful module type. It controls **the loop** — the fundamental cycle of calling the LLM, processing tool calls, and deciding what happens next. Swap the orchestrator and you radically change how your agent behaves.

There are currently **3 orchestrators**:

- **loop-basic** — Sequential request/response. Simple and predictable.
- **loop-streaming** — Real-time streaming with extended thinking support. What you see in the CLI.
- **loop-events** — Event-driven orchestration with full hook integration. The most capable.

Most users never think about the orchestrator because the default just works. But if you need custom retry logic, multi-agent coordination, or a fundamentally different execution strategy, this is where you'd build it.

### 4. Hook — Lifecycle Observers

Hooks observe what's happening in the session and react to events. The critical distinction: hooks are **code-decided**, not LLM-decided. The LLM never chooses to invoke a hook. Instead, hooks fire automatically when lifecycle events occur — a message is sent, a tool is called, a response arrives.

```
__call__(event, data) → HookResult
```

The ecosystem includes **12+ hooks**: JSONL logging, real-time streaming UI, git status context injection, approval gates for sensitive operations, privacy redaction, session backup, cost-aware scheduling, and more. Hooks are how you add observability, safety, and policy without touching the agent's core behavior.

### 5. Context — Memory Management

Context modules manage the conversation's memory. They store messages, provide access to history, and handle compaction when conversations get long.

```
add_message() / get_messages() / set_messages() / clear()
```

There are currently **2 context modules**:

- **context-simple** — In-memory with automatic compaction. The default for most sessions.
- **context-persistent** — File-backed storage that survives across sessions. For long-running work.

## Tool vs. Hook: The Triggering Difference

This is the single most important design distinction in Amplifier's module system. Both tools and hooks extend what the agent can do, but they differ in **who decides when they run**:

| | Tool | Hook |
|---|------|------|
| **Triggered by** | The LLM decides | Code fires automatically |
| **Visible to LLM?** | Yes — it sees the name, description, and schema | No — completely invisible |
| **Returns to LLM?** | Yes — results flow back into the conversation | No — side effects only |
| **Example** | `bash` — LLM decides to run a shell command | `hooks-logging` — every event is logged to disk |

Here's the intuition: if you want the agent to **choose** when to do something, make it a tool. If you want something to **always happen** regardless of what the agent decides, make it a hook.

A `read_file` tool makes sense because the agent should decide when it needs to see a file. But a logging hook makes sense because you want every event logged, whether the agent "wants" that or not.

## Using Modules

In practice, you rarely think about individual modules. Your bundle declares what you need, and the kernel handles the rest. But there are times when you want to see what's loaded or change things on the fly.

> What modules are available?

```bash
amplifier module list
```

> I want to add web search to my current bundle.

In your bundle YAML, add the tool:

```yaml
tools:
  - module: tool-web
    source: git+https://github.com/microsoft/amplifier-module-tool-web@main
```

> Can I swap providers mid-session?

Yes. This is one of the most practical things about the module architecture. Say you're working with Claude and want to try the same task with GPT-4:

```
> /model openai/gpt-4.1

Switched to openai/gpt-4.1

> Now summarize the changes we discussed.

[Response now comes from GPT-4.1 instead of Claude,
 with full access to the same conversation history
 and the same tools]
```

The context, tools, hooks, and orchestrator all stay the same. Only the provider changes. This is what "swappable" means in practice — the contracts make modules interchangeable within their type.

## Creating Your Own

Amplifier uses Python's structural typing (protocols), so you never need to inherit from a base class. If your object has the right methods, it's a valid module.

Here's a complete custom tool:

```python
from amplifier_core.models import ToolResult

class WordCounter:
    """Counts words in text."""

    @property
    def name(self) -> str:
        return "word_counter"

    @property
    def description(self) -> str:
        return "Count the number of words in a given text"

    async def execute(self, input: dict) -> ToolResult:
        text = input.get("text", "")
        count = len(text.split())
        return ToolResult(output=f"Word count: {count}")

async def mount(coordinator, config):
    tool = WordCounter()
    await coordinator.mount("tools", tool, name="word_counter")

    async def cleanup():
        pass
    return cleanup
```

Register it in `pyproject.toml`:

```toml
[project.entry-points."amplifier.modules"]
tool-word-counter = "my_tool:mount"
```

That's it. The kernel discovers the module by its entry point name, calls `mount()`, and the tool appears in the agent's capabilities. No framework classes to extend, no decorators to apply. Just implement the contract.

## Best Practices

**Start with existing modules.** The ecosystem has 11 providers, 12+ tools, 3 orchestrators, 12+ hooks, and 2 context managers. Before building something custom, check whether a module already does what you need.

**Choose the right type.** The most common mistake is building a hook when you need a tool (or vice versa). Ask: "Should the LLM decide when this runs?" If yes, it's a tool. If no, it's a hook.

**Keep modules focused.** A module should do one thing well. If your tool is growing to handle multiple unrelated tasks, split it into multiple tools. The LLM is better at choosing between focused tools than navigating a Swiss Army knife.

**Handle errors gracefully.** A module should never crash the kernel. Return error information in your result rather than raising exceptions that propagate upward.

**Respect the contract.** Protocols exist for a reason. Don't add hidden side channels between modules. If your tool needs to communicate with a hook, go through the coordinator's event system.

## Key Takeaways

1. **Modules are swappable units** that plug into the kernel through contracts. Change a module, change the behavior — without touching anything else.

2. **Exactly five types**, each with a clear role: Provider (LLM), Tool (capabilities), Orchestrator (the loop), Hook (observers), and Context (memory).

3. **Tool vs. Hook is about who decides.** The LLM triggers tools; code triggers hooks. This is the most important design decision when building modules.

4. **Bundle YAML drives everything.** Declare your modules in the bundle, and the kernel loads, mounts, and wires them automatically.

5. **The ecosystem is rich.** With 11 providers, 12+ tools, 3 orchestrators, 12+ hooks, and growing community contributions, you can compose powerful agents from existing pieces.

6. **Structural typing means no inheritance.** Implement the right methods and your class is a valid module. Duck typing at its best.

7. **Swapping is practical, not theoretical.** You can switch providers mid-session, add tools to a running bundle, or change orchestrators — and everything else keeps working.
