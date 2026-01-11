# Architecture - Internals Supplement

This content is injected into architecture/concepts pages explaining what happens behind the scenes.

## What Just Happened?

When you run Amplifier, here's what happens internally.

### Session Lifecycle

```
amplifier run "Hello"
         │
         ▼
┌─────────────────────────────────────┐
│ 1. Session Created                  │
│    - Unique ID generated            │
│    - Working directory recorded     │
│    - Timestamp logged               │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│ 2. Bundle Loaded                    │
│    - Parse bundle.md/YAML           │
│    - Load included bundles          │
│    - Mount modules (tools, hooks)   │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│ 3. Orchestrator Loop                │
│    - Send prompt to provider        │
│    - Process tool calls             │
│    - Emit events to hooks           │
│    - Repeat until complete          │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│ 4. Session Saved                    │
│    - events.jsonl written           │
│    - transcript.md generated        │
│    - State persisted for resume     │
└─────────────────────────────────────┘
```

### Where Sessions Live

All session data is stored locally:

```
~/.amplifier/
├── sessions/
│   └── [session-id]/
│       ├── events.jsonl      # Full event log (source of truth)
│       ├── transcript.md     # Human-readable conversation
│       └── state.json        # Session metadata
├── config/
│   └── settings.yaml         # Your configuration
└── cache/
    └── [bundle-cache]/       # Downloaded bundles
```

### Inspecting a Session

```bash
# Find your sessions
ls ~/.amplifier/sessions/

# Read the transcript
cat ~/.amplifier/sessions/[id]/transcript.md

# See raw events (large, use with care)
head -20 ~/.amplifier/sessions/[id]/events.jsonl
```

### Event Flow

Every action emits events that hooks can observe:

```
session:start    → Session begins
prompt:start     → User message received
provider:request → LLM API called
provider:response → LLM responds
tool:start       → Tool execution begins
tool:complete    → Tool returns result
content_block:*  → Streaming content
prompt:complete  → Turn finished
session:end      → Session closes
```

Hooks subscribe to these events for logging, approval gates, UI updates, etc.
