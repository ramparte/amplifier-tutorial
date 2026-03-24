---
id: modules
type: concepts
title: "Understanding Modules"
---

# Understanding Modules

## What is a Module?

A **module** is a pluggable component that extends your agent's capabilities. Think of modules as LEGO blocks—each one serves a specific purpose, and you can mix and match them to build the exact agent you need.

Modules are the foundation of the framework's architecture. Instead of building a monolithic agent with hardcoded behaviors, you compose your agent from independent, reusable modules. This design makes your agent:

- **Flexible**: Swap modules without rewriting code
- **Testable**: Test each module in isolation
- **Maintainable**: Update one module without breaking others
- **Extensible**: Add new capabilities by adding modules

Every module in the system implements a specific interface (or "contract") that defines how it communicates with the rest of the agent. This contract ensures that modules work together seamlessly, regardless of who wrote them or when they were created.

### The Module Lifecycle

When an agent starts, it:

1. **Loads** all configured modules
2. **Initializes** them with configuration
3. **Registers** them with the appropriate systems
4. **Executes** them during the agent loop
5. **Cleans up** when the agent shuts down

Modules can maintain state between invocations, access shared resources, and communicate with each other through well-defined interfaces.

## The 5 Module Types

The framework defines five core module types, each serving a distinct architectural role:

| Type | Purpose | Contract |
|------|---------|----------|
| Provider | LLM backends | complete() |
| Tool | Agent capabilities | execute() |
| Orchestrator | Main engine | The loop |
| Context | Memory | messages |
| Hook | Observers | events |

### Provider Modules

**Purpose**: Connect to language model backends

Providers abstract away the details of different LLM APIs (OpenAI, Anthropic, local models, etc.). They implement a single `complete()` method that takes a conversation history and returns the next response.

**Example use cases**:
- OpenAI GPT-4 provider
- Anthropic Claude provider
- Local Llama model provider
- Custom fine-tuned model provider

**Key characteristics**:
- Stateless request/response
- Handles authentication and rate limiting
- Transforms between framework format and API format
- Supports streaming responses

```python
class Provider:
    def complete(self, messages: list) -> Response:
        """Generate next response from conversation history"""
        pass
```

### Tool Modules

**Purpose**: Give your agent capabilities to interact with the world

Tools are functions that the LLM can call to perform actions—reading files, searching the web, executing code, querying databases, etc. The LLM decides which tool to use based on the conversation context.

**Example use cases**:
- File system operations (read, write, edit)
- Web search and scraping
- Database queries
- API integrations
- Code execution

**Key characteristics**:
- LLM-triggered (agent decides when to use)
- Synchronous or asynchronous execution
- Structured input parameters
- Returns results to the LLM

```python
class Tool:
    def execute(self, **params) -> dict:
        """Execute the tool with given parameters"""
        pass
```

### Orchestrator Modules

**Purpose**: Control the main agent loop

The orchestrator is the "brain" of your agent. It manages the conversation flow, decides when to call the LLM, processes tool calls, handles errors, and determines when the agent's work is complete.

**Example use cases**:
- Simple loop (call LLM → execute tools → repeat)
- Tree search orchestrator (explore multiple paths)
- Reflection orchestrator (self-critique and improve)
- Multi-agent orchestrator (coordinate multiple agents)

**Key characteristics**:
- Runs the main agent loop
- Manages state and conversation history
- Decides when to stop
- Handles errors and retries

```python
class Orchestrator:
    def run(self) -> Result:
        """Execute the main agent loop"""
        while not self.is_complete():
            response = self.provider.complete(self.context.messages)
            self.process_response(response)
        return self.result
```

### Context Modules

**Purpose**: Manage conversation memory and state

Context modules store and organize the conversation history. They decide what information to keep, what to summarize, and what to discard. This is crucial for long conversations where the full history exceeds the LLM's context window.

**Example use cases**:
- Simple in-memory context (store all messages)
- Sliding window context (keep last N messages)
- Semantic context (keep most relevant messages)
- Persistent context (save to database)

**Key characteristics**:
- Stores conversation messages
- Provides access to history
- May implement compression or summarization
- Can persist across sessions

```python
class Context:
    @property
    def messages(self) -> list:
        """Get current conversation history"""
        pass
    
    def add_message(self, message: dict):
        """Add message to history"""
        pass
```

### Hook Modules

**Purpose**: Observe and react to agent events

Hooks are passive observers that get notified when specific events occur in the agent's lifecycle. Unlike tools (which the LLM chooses to use), hooks are triggered by code events automatically.

**Example use cases**:
- Logging and debugging
- Performance monitoring
- Cost tracking
- Safety guardrails
- Analytics and telemetry

**Key characteristics**:
- Code-triggered (not LLM-triggered)
- Cannot be chosen by the agent
- Observe without blocking
- Multiple hooks can respond to same event

```python
class Hook:
    def on_event(self, event: Event):
        """Handle an agent event"""
        pass
```

## Tool vs Hook

Understanding the difference between Tools and Hooks is crucial for proper module design:

| | Tool | Hook |
|--|------|------|
| Triggered by | LLM decides | Code events |
| Choosable | Yes, agent picks from available tools | No, always runs on events |
| Blocks execution | Yes, agent waits for result | No, runs async or returns quickly |
| Returns value | Yes, result goes to LLM | No, side effects only |
| Example | "search_web" tool called when agent needs info | Logging hook records all LLM calls |

### When to Use a Tool

Create a Tool when:
- The agent should **decide** when to use it
- The action **produces a result** the LLM needs
- It's an **explicit capability** you want to advertise

**Example**: A `read_file` tool that the agent calls when it needs to see file contents.

### When to Use a Hook

Create a Hook when:
- The action should happen **automatically** on events
- It's an **observation** or side effect, not a core capability
- You want to **monitor or modify** agent behavior

**Example**: A `cost_tracker` hook that logs every LLM API call to track spending.

### Can It Be Both?

Sometimes! You might have a Hook that monitors for unsafe behavior AND a Tool that lets the agent explicitly check safety. The key is understanding the use case:

- **Tool version**: "Is this action safe?" (agent decides to check)
- **Hook version**: "Log every action and alert on unsafe patterns" (always runs)

## Module Configuration

Modules are typically configured in a YAML or JSON file:

```yaml
modules:
  provider:
    type: openai
    model: gpt-4
    
  tools:
    - name: file_ops
      enabled: true
    - name: web_search
      api_key: ${SEARCH_API_KEY}
      
  orchestrator:
    type: simple_loop
    max_iterations: 20
    
  context:
    type: sliding_window
    window_size: 10
    
  hooks:
    - name: logger
      level: info
    - name: cost_tracker
      alert_threshold: 10.00
```

## Creating Custom Modules

To create a custom module:

1. **Choose the right type** based on architectural role
2. **Implement the contract** (required interface)
3. **Register with the framework** (add to config)
4. **Test in isolation** before integration

Each module type has a base class you extend:

```python
from framework import ToolModule

class MyCustomTool(ToolModule):
    name = "my_tool"
    description = "What this tool does"
    
    parameters = {
        "param1": {"type": "string", "description": "..."},
        "param2": {"type": "number", "description": "..."}
    }
    
    def execute(self, param1: str, param2: int) -> dict:
        # Your implementation
        result = self.do_something(param1, param2)
        return {"result": result}
```

## Key Takeaways

1. **Modules are pluggable components** that extend agent capabilities through well-defined interfaces

2. **Five module types** serve different architectural roles: Provider (LLM), Tool (capabilities), Orchestrator (loop), Context (memory), Hook (observers)

3. **Tools vs Hooks**: Tools are chosen by the LLM and return results; Hooks are triggered by code events for observation

4. **Composition over inheritance**: Build agents by combining modules rather than writing monolithic code

5. **Each module has a contract**: Implement the required interface, and your module works with the framework

6. **Modules enable reusability**: Write once, use across multiple agents and projects

7. **Testing is easier**: Test each module independently before integration

8. **Configuration-driven**: Change agent behavior by swapping modules in config, not code

The modular architecture is what makes the framework powerful and flexible. By understanding these five module types and their roles, you can build sophisticated agents that are maintainable, testable, and extensible.

Start simple—use built-in modules first, then create custom modules as you understand the patterns. The framework handles the complexity of wiring modules together, so you can focus on building great capabilities.
