---
id: architecture
type: concept
title: "Architecture"
---

# Architecture

You've seen Amplifier work -- you type a prompt, tools fire, and a response appears. But what's actually running under the hood? Understanding the architecture explains *why* things work the way they do: why you can swap providers without changing bundles, why modules never break each other, and why the whole system stays stable as the ecosystem grows.

## What is the Architecture?

Amplifier's architecture is a layered stack with a tiny, stable kernel at the center and everything else -- tools, providers, orchestrators, hooks -- living in replaceable modules at the edges. The design is borrowed from the Linux kernel: the core provides **mechanisms** (how things work), while modules provide **policies** (what to do).

The kernel is implemented in Rust (~2,600 lines) with Python bindings via PyO3. It's small enough for one person to understand completely. Everything you actually interact with -- the CLI, the tools, the AI models -- lives in modules and libraries that surround it.

## How It Works

Here's the full stack, from the user interface down to the modules:

```
+------------------------------------------------------------------+
|  APPLICATIONS                                                     |
|  amplifier-app-cli, amplifierd, amplifier-chat, amplifier-voice   |
+----------------------------+-------------------------------------+
                             |
+----------------------------v-------------------------------------+
|  LIBRARIES (Foundation)                                           |
|  amplifier-foundation: bundles, module resolution, utilities      |
+----------------------------+-------------------------------------+
                             |
+----------------------------v-------------------------------------+
|  KERNEL (amplifier-core)                    ~2,600 lines of Rust  |
|  * Session lifecycle         * Event system                       |
|  * Coordinator               * Hook registry                     |
|  * Module contracts          * Cancellation tokens                |
|  * Mount plan execution      * PyO3 Python bindings               |
+----------------------------+-------------------------------------+
                             |
         +-------------------+-------------------+
         |         |         |         |         |
    +----v---+ +---v----+ +-v------+ +v------+ +v--------+
    |Provider| |  Tool  | |Orchestr| | Hook  | | Context |
    |        | |        | |  ator  | |       | | Manager |
    +--------+ +--------+ +--------+ +-------+ +---------+
     Claude     bash       loop-      logging   simple
     OpenAI     read_file  streaming  redaction  persistent
     Gemini     web_search loop-basic approval
     Ollama     grep       loop-events
```

Each layer has a clear boundary. Applications consume the Foundation library. The Foundation library uses the kernel. Modules only depend on the kernel -- never on the Foundation or applications. This means a tool module works identically whether it's running in the CLI, the web daemon, or a test harness.

### The Layers Explained

**Applications** are the user-facing interfaces. The CLI (`amplifier-app-cli`) is the one most people use, but `amplifierd` exposes the same kernel over HTTP/SSE, and `amplifier-chat` and `amplifier-voice` add web and voice interfaces on top of it.

**Libraries** sit between applications and the kernel. `amplifier-foundation` handles bundle resolution, module discovery, and shared utilities. This is where bundles get compiled into mount plans that the kernel can execute.

**The Kernel** (`amplifier-core`) is the stable center. It manages sessions, coordinates modules, dispatches hooks, and emits events. It never decides *which* modules to load or *how* to orchestrate a conversation -- those are policy decisions that belong in modules.

**Modules** are the replaceable edges. Every capability the agent has -- talking to an LLM, reading files, running shell commands, logging events -- is delivered through a module.

### Mechanism, Not Policy

The kernel's design principle is simple: provide capabilities, don't make decisions.

| Kernel Provides (Mechanism) | Modules Decide (Policy)     |
|-----------------------------|-----------------------------|
| Module loading              | Which modules to load       |
| Event emission              | What to log, and where      |
| Session lifecycle           | Orchestration strategy      |
| Hook registration           | Security and filtering rules|

The litmus test: *"Could two teams want different behavior here?"* If yes, it's policy, and it belongs in a module -- not the kernel.

This is why one team can use Claude with streaming orchestration and file-safety hooks, while another uses GPT with basic orchestration and no hooks at all. The kernel doesn't care. It provides the same mechanisms to both.

## The Five Module Types

Every capability in Amplifier flows through one of five module contracts. Each uses Python's Protocol (structural typing -- no inheritance required):

**Provider** -- The AI brain. Providers connect to language models via `complete()`. Swap Anthropic for OpenAI or Ollama without touching anything else.

**Tool** -- Hands for the AI. Tools expose `execute()` and are *chosen by the LLM* based on the conversation. When you ask "find all TODOs in my code," the agent picks the `grep` tool.

**Orchestrator** -- The main engine. This is not just a strategy pattern -- the orchestrator *is* the agent loop. It receives user input, calls the provider, processes tool calls, fires hooks, and decides when the turn is complete. Everything flows through it.

**Hook** -- Lifecycle observers. Hooks fire *automatically* on code events -- the agent never chooses to invoke a hook. Logging, redaction, approval gates, cost tracking -- all hooks.

**Context** -- Memory management. Context modules control what the agent remembers via `add_message()`, `get_messages()`, and `compact()`. Simple context keeps everything in memory; persistent context survives across sessions.

### Tool vs Hook: Who Decides?

This distinction matters for module design:

| Aspect            | Tool                           | Hook                          |
|-------------------|--------------------------------|-------------------------------|
| **Triggered by**  | LLM decides to call it         | Code events, automatic        |
| **Returns value** | Yes, result goes back to LLM   | No, side effects only         |
| **Blocks agent**  | Yes, agent waits for result    | No, runs quickly              |
| **Example**       | `read_file` reads a file on demand | `hooks-logging` records every event |

The question to ask: *"Should the AI decide when this runs?"* If yes, it's a Tool. If it should always happen automatically, it's a Hook.

### The Orchestrator is THE Engine

It's worth emphasizing: the orchestrator isn't a pluggable strategy sitting alongside the agent. It *is* the agent loop. Here's what `loop-streaming` does on every turn:

```
User input arrives
    |
    v
[Hook: pre_turn] -----> Hooks observe/modify
    |
    v
Provider.complete() ---> Streams response from LLM
    |
    v
Tool calls found? --+---> No ----> Return response to user
                    |
                    v Yes
              Execute tools
              [Hook: pre_tool / post_tool]
                    |
                    v
              Feed results back to Provider
              (loop continues)
```

Different orchestrators implement fundamentally different execution strategies. `loop-basic` does simple request/response. `loop-streaming` adds real-time token streaming with extended thinking. `loop-events` adds event-driven hook integration. Swapping orchestrators changes *how your agent thinks*.

## The Event System

Every significant action in Amplifier emits an event. These events are written to a per-session JSONL file that serves as the **source of truth** for what happened during a session.

```python
{"event": "session.started", "ts": "2026-04-02T10:00:00Z", "session_id": "abc123"}
{"event": "turn.started",   "ts": "2026-04-02T10:00:01Z", "turn": 1}
{"event": "tool.called",    "ts": "2026-04-02T10:00:02Z", "tool": "read_file", "args": {"path": "src/main.py"}}
{"event": "tool.result",    "ts": "2026-04-02T10:00:02Z", "tool": "read_file", "status": "success"}
{"event": "turn.completed", "ts": "2026-04-02T10:00:05Z", "turn": 1}
```

Hooks subscribe to these events using pattern matching. The `hooks-logging` module writes them to disk. The `hooks-streaming-ui` module renders them as live terminal output. The `hooks-approval` module intercepts specific events and pauses for human confirmation. Same event stream, different policies.

## Session Lifecycle

A session is the execution context that ties everything together. It holds mounted modules, conversation state, and configuration. Here's the lifecycle:

```
create --> configure --> run --> complete
  |           |          |         |
  |     Mount plan     Agent     Cleanup
  |     compiled       loop      modules,
  |     from bundle    runs      flush events
  |
  Session ID assigned,
  event log opened
```

In code, this maps to:

```python
async with AmplifierSession(config) as session:     # create + configure
    response = await session.execute("List files")   # run
                                                     # complete (on exit)
```

The `async with` block handles the full lifecycle: creating the session, mounting modules from the config, running the agent loop, and cleaning up when done.

## Bundle Composition

Bundles are the composition layer between human intent and the kernel's mount plan. A bundle is a Markdown file with YAML frontmatter that declares what modules, behaviors, and agents to include:

```
Bundle (Markdown + YAML)
    |
    +--> Behaviors (reusable capability sets)
    |       +--> Tools, hooks, context injections
    |
    +--> Agents (specialized personas)
    |       +--> System prompt + curated tool access
    |
    +--> Includes (other bundles)
            +--> Recursive composition
```

The Foundation library compiles bundles into a **mount plan** -- a flat configuration dictionary that tells the kernel exactly which modules to load and how to configure them. The kernel never sees bundles; it only sees mount plans.

This separation means you can create a mount plan by hand, generate one programmatically, or let the Foundation compile one from bundles. The kernel doesn't care where the plan came from.

## Creating Your Own

Understanding the architecture helps you decide *where* to extend:

- **Need a new capability?** Write a Tool module. Implement `execute()`, register via `mount()`.
- **Need to observe or guard behavior?** Write a Hook module. Subscribe to events you care about.
- **Need a different AI model?** Write a Provider module. Implement `complete()`.
- **Need a different execution strategy?** Write an Orchestrator module. Own the entire agent loop.
- **Need different memory behavior?** Write a Context module. Control `add_message()` and `compact()`.

Every module follows the same pattern: implement a `mount(coordinator, config)` function that registers your component with the coordinator and returns a cleanup function.

```python
async def mount(coordinator, config):
    tool = MyTool()
    await coordinator.mount("tools", tool, name="my_tool")

    async def cleanup():
        pass  # Release resources
    return cleanup
```

## Best Practices

**Respect the layers.** Modules depend on `amplifier-core` only -- never on `amplifier-foundation` or applications. This keeps modules portable across any Amplifier interface.

**Use the litmus test.** Before putting logic in the kernel, ask: "Could two teams want different behavior?" If yes, it's a module.

**Let the orchestrator orchestrate.** Don't fight the agent loop. If you need custom execution flow, write a new orchestrator rather than hacking around the existing one.

**Events are your observability layer.** Don't log manually inside tools. Emit events and let hooks handle logging, metrics, and alerting. This keeps concerns separated.

**Keep modules isolated.** Modules shouldn't know about each other. They communicate through the kernel's coordination mechanisms -- hooks, events, and the coordinator. If two modules need to share data, that's a design signal to reconsider.

## Key Takeaways

1. **Layered stack**: Applications -> Foundation -> Kernel -> Modules. Each layer has a clear boundary and dependency direction.

2. **Tiny kernel, large ecosystem**: The Rust kernel (~2,600 lines) provides mechanisms. All behavior lives in replaceable modules.

3. **Five module types**: Provider (LLM), Tool (capabilities), Orchestrator (the loop), Hook (observers), Context (memory). Each has a protocol contract.

4. **Mechanism, not policy**: The kernel never decides what to do -- only how things can be done. Litmus test: "Could two teams differ?" -> Module.

5. **The orchestrator is the engine**: It's not a strategy pattern. It *is* the agent loop that drives everything.

6. **Tools vs Hooks**: Tools are LLM-decided ("I need to read this file"). Hooks are code-decided ("Log every event automatically").

7. **Events are the source of truth**: JSONL session logs capture everything. Hooks subscribe to events for observability without coupling.

8. **Bundles compile to mount plans**: The composition layer (bundles, behaviors, agents) is resolved by the Foundation library into flat mount plans the kernel executes.

## Next Steps

With this architectural understanding, go deeper into each layer:

- **[Modules](./modules.md)** -- The five module types in detail, with creation patterns
- **[Hooks](./hooks.md)** -- Event subscription, hook chains, and lifecycle points
- **[Bundles](./bundles.md)** -- Composition, includes, behaviors, and the thin bundle pattern
- **[Agents](./agents.md)** -- Specialized personas and session delegation
