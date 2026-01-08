---
id: architecture-deep-dive
type: concepts
title: "Architecture Deep Dive"
---

# Architecture Deep Dive

How Amplifier's pieces fit together - from the lead developer's perspective.

## The Big Picture

Amplifier has multiple ways to infuse capabilities into the system. Each serves a different purpose and offers different levels of control.

```
┌─────────────────────────────────────────────────────────┐
│                     Your Session                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Orchestrator (the engine)            │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐          │   │
│  │  │ Provider│  │ Context │  │  Hooks  │          │   │
│  │  │  (LLM)  │  │ (memory)│  │ (events)│          │   │
│  │  └─────────┘  └─────────┘  └─────────┘          │   │
│  │                    │                              │   │
│  │              ┌─────┴─────┐                       │   │
│  │              │   Tools   │                       │   │
│  │              │(LLM calls)│                       │   │
│  │              └─────┬─────┘                       │   │
│  │                    │                              │   │
│  │              ┌─────┴─────┐                       │   │
│  │              │  Agents   │                       │   │
│  │              │(sub-sess) │                       │   │
│  │              └───────────┘                       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  All packaged in: BUNDLES                               │
└─────────────────────────────────────────────────────────┘
```

## Agents: Amplifier Driving Amplifier

> "Agents are 'just' bundles that we spawn a sub-session and the main session drives as the 'user' instead of us."

**Key insight:** Agents aren't a special concept - they're bundles that Amplifier talks to *as if it were the user*.

```
Main Session (you're the user)
    │
    ├── Spawns sub-session with "zen-architect" bundle
    │   └── Main session sends prompts as "user"
    │   └── Agent responds, main session continues
    │
    └── You see the combined result
```

This is why we use the **task tool** to spawn agents - you're giving Amplifier a "task" to delegate to a specialized instance of itself.

Each agent bundle is focused on specific tasks:
- `zen-architect` - Design and code review
- `bug-hunter` - Debugging
- `explorer` - Codebase reconnaissance

## Tools: LLM-Driven Capabilities

> "Tools are code that can be called by the LLMs... the LLM decides to call a tool instead of generating a response."

**Key insight:** The LLM chooses when to use tools. You don't call them - the model does.

The flow:
1. You send a message
2. LLM decides: "I need to read a file"
3. LLM returns a tool call request (not a response)
4. Tool executes (no models involved)
5. Result goes back to LLM
6. LLM decides what to do next
7. Repeat until LLM generates a final response

```
You: "What's in config.yaml?"
         │
         ▼
LLM: "I should read that file"
         │
         ▼
Tool Call: read_file("config.yaml")
         │
         ▼
Tool Result: "port: 8080\nhost: localhost"
         │
         ▼
LLM: "The config has port 8080 and host localhost"
         │
         ▼
You see the response
```

## Hooks: Code-First Control

> "Hooks are code that listens to lifecycle events... NO LLM making decisions IF they should be called."

**Key insight:** Hooks always run on their events. Full programmatic control.

Unlike tools (LLM decides), hooks fire automatically:

| Event | Hook Fires |
|-------|------------|
| Session starts | `session:start` hooks run |
| Tool called | `tool:call` hooks run |
| Response generated | `turn:end` hooks run |

Use hooks for:
- Logging everything (observability)
- Injecting context (status, time, reminders)
- Approval gates (pause before dangerous actions)
- Redaction (remove sensitive data)

```python
# A hook runs on events, not LLM decisions
@hook("tool:call")
def log_tool_usage(event):
    print(f"Tool used: {event.tool_name}")
    # This ALWAYS runs when a tool is called
```

## Orchestrators: The Engine

> "Orchestrators are the main engine that drives your 'Amplifier Session' and can make the experience radically different."

**Key insight:** The orchestrator is code-first. It decides how everything connects.

Our main orchestrators are agentic loops, but possibilities include:
- **Recipe runner** - Execute YAML workflows
- **Observer pattern** - Other instances "observe" a limited main session and push context in
- **Custom flows** - Any pattern you can code

The orchestrator:
1. Receives your input
2. Calls the provider (LLM)
3. Handles tool calls
4. Fires hook events
5. Manages the conversation loop
6. Returns the response

Different orchestrators = radically different behaviors.

## Context Modules: Memory Management

> "Context modules are the managers of your context, storing/loading/'compacting' your history."

Context modules handle:
- **Storage** - Where conversation history lives
- **Loading** - Retrieving past context
- **Compacting** - Summarizing long histories
- **Memory systems** - Persistent knowledge

This is where "memory" implementations plug in.

## Providers: LLM Backends

> "Providers are generally vendor level and support many/all of their models."

Provider patterns:
- **Vendor provider** - One provider for all Anthropic models
- **Model-specific** - One provider per model
- **Aggregator** - One provider routing to many backends

Providers handle the actual LLM API calls.

## Bundles: The Packaging

> "Bundles let us group together the various parts. It could be as lightweight as a bundle around a tool module or as fully featured as the foundation bundle."

Bundles are **composition**:

```yaml
# A minimal bundle
bundle:
  name: my-tool
tools:
  - module: my-tool

# A full-featured bundle
bundle:
  name: foundation
includes:
  - bundle: behavior-git-ops
  - bundle: behavior-security
  - bundle: behavior-testing
tools:
  - module: tool-filesystem
  - module: tool-bash
agents:
  - path: ./agents/zen-architect.yaml
```

**Behavior bundles** are a convention (not code):
- Smaller, focused bundles
- Compose into larger bundles
- Foundation bundle includes many behavior bundles

## Skills: Bundles All the Way Down

> "Skills is 'just' a bundle. It has tools, context files, agent(s), etc."

**Key insight:** Skills aren't a core Amplifier concept. They're a *pattern* we use.

The skills bundle:
- Provides a `load_skill` tool
- Loads knowledge packages on demand
- Each skill = structured context + patterns

This pattern extends Amplifier for:
- MCP integration
- Anthropic plugins
- Domain knowledge packages

We'll likely move agents to follow this same pattern - agents aren't "core" either, they're built on top through foundation.

## Control Spectrum

From most LLM-driven to most code-driven:

```
More LLM Control                              More Code Control
      │                                              │
      ▼                                              ▼
   Tools ──────── Agents ──────── Hooks ──────── Orchestrators
      │              │              │                │
   LLM decides   LLM drives     Events fire      Code drives
   when to call  sub-sessions   automatically    everything
```

Choose based on your needs:
- **Need flexibility?** → Tools, Agents
- **Need reliability?** → Hooks, Orchestrators
- **Need both?** → Combine them

## Summary

| Component | Who Decides | Runs When |
|-----------|-------------|-----------|
| **Tools** | LLM | LLM requests it |
| **Agents** | LLM (via task) | LLM spawns sub-session |
| **Hooks** | Code | Events fire |
| **Orchestrators** | Code | Always (it's the engine) |
| **Context** | Code | Load/save operations |
| **Providers** | Code | LLM API calls |
| **Bundles** | You | Configuration time |

Everything composes through bundles. Start simple, add capabilities as needed.
