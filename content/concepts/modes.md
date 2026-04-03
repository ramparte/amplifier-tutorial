---
id: modes
type: concept
title: "Modes"
---

# Modes

An agent with every tool available at all times is an agent with no guardrails.
Modes fix that. When you activate a mode, you change the agent's runtime
behavior — which tools it can use, what guidance it follows, how it approaches
your problem — without touching the bundle that defines it. Think of a mode as
a transparent overlay: the agent underneath is the same, but the rules it
operates under shift to match the task at hand.

## What is a Mode?

A mode is a **runtime behavior overlay** — a markdown file with YAML frontmatter
that modifies how Amplifier operates when active. It does three things
simultaneously:

1. **Injects context.** The mode's markdown body becomes a system reminder,
   shaping how the agent reasons and responds.
2. **Enforces tool policies.** Each tool gets a policy — safe, warn, confirm,
   or block — controlling what the agent can and can't do.
3. **Shows a visual indicator.** The prompt changes so you always know which
   mode is active.

Modes are *temporary*. They activate, they shape behavior, and they deactivate.
The bundle never changes. You can switch modes mid-session, clear them entirely,
or let one mode transition into another as your workflow progresses.

## How It Works

### Activating a Mode

The simplest way to enter a mode is the slash command:

> Enter brainstorm mode.

```
/brainstorm
→ Mode activated: brainstorm
  "Design refinement before any creative work"
```

That's it. The agent now operates under brainstorm rules — read-only tools are
safe, write tools are blocked, and the markdown body of the brainstorm mode
file guides every response.

### Tool Policies

The heart of a mode is its tool policy map. Every tool gets one of four
policies:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **safe** | Normal execution, no restrictions | Read-only tools in exploration modes |
| **warn** | First call blocked with a reminder; retry to proceed | Bash in planning modes — discouraged but allowed |
| **confirm** | User must approve each call | File writes in careful mode |
| **block** | Tool completely disabled | Write tools in brainstorm mode |

Any tool not explicitly listed falls to the mode's `default_action`. Most modes
set `default_action: block`, meaning only explicitly listed tools are available.
This is allowlist behavior — safer than trying to enumerate everything to deny.

Here's how the brainstorm mode configures its tools:

```yaml
mode:
  name: brainstorm
  tools:
    safe:
      - read_file
      - glob
      - grep
      - web_search
      - load_skill
      - LSP
    warn:
      - bash
  default_action: block
```

Reading and searching — safe. Bash — warned (you can still run it, but the
system nudges you first). Everything else — `write_file`, `edit_file`,
`apply_patch` — blocked entirely. The agent physically cannot modify files
while brainstorming. That's not a suggestion; it's an enforcement.

### Context Injection

The markdown body below the YAML frontmatter becomes the mode's guidance. When
a mode is active, this content is injected as a system reminder before each
turn. The agent sees it as authoritative instructions:

```markdown
---
mode:
  name: explore
  tools:
    safe: [read_file, glob, grep, web_search, LSP]
  default_action: block
---

EXPLORE MODE: Understand the codebase with zero side effects.

Your role:
- MAP the codebase structure
- TRACE code paths and dependencies
- ANSWER questions about how things work
- BUILD mental model before any action

Do NOT:
- Modify files
- Execute commands
- Make any changes whatsoever
```

The tool policies *enforce* the "Do NOT modify files" rule. The markdown body
*explains* the intent so the agent understands why.

## Available Modes

Amplifier bundles ship modes for common workflows. The Superpowers bundle
provides a complete development pipeline:

| Mode | Shortcut | Purpose |
|------|----------|---------|
| **brainstorm** | `/brainstorm` | Design refinement through collaborative dialogue |
| **write-plan** | `/write-plan` | Create detailed implementation plans with TDD tasks |
| **execute-plan** | `/execute-plan` | Subagent-driven development with review pipeline |
| **debug** | `/debug` | Systematic 4-phase debugging |
| **verify** | `/verify` | Evidence-based completion verification |
| **finish** | `/finish` | Complete branch — merge, PR, keep, or discard |

These modes form a pipeline. You brainstorm a design, write an implementation
plan, execute it with subagents, verify the results, and finish the branch.
Each mode constrains the agent to exactly the tools and mindset that phase
requires.

Other bundles add their own modes. The `modes` bundle provides lightweight
general-purpose modes like `explore` (read-only codebase navigation), `plan`
(analyze without implementing), and `careful` (full capability with
confirmation gates on destructive actions).

> List all available modes.

```
[Tool: mode] operation="list"
→ 16 modes available:
  brainstorm (superpowers), write-plan (superpowers), debug (superpowers),
  explore (modes), plan (modes), careful (modes), ...
```

### A Practical Session

Here's what a mode-driven workflow looks like in practice:

> I need to add WebSocket support to the notification system. /brainstorm

```
→ Mode activated: brainstorm
  Exploring project context...
  Let me ask some clarifying questions before we design this.
  What notification types need real-time delivery?
```

The agent reads code, asks questions, proposes approaches — but cannot write
files. After you validate the design section by section:

> Design looks good. Let's plan the implementation.

```
[Tool: mode] operation="set", name="write-plan"
→ Mode activated: write-plan
  Creating implementation plan with TDD tasks...
```

The mode transitions. Now the agent has the same read-only tools plus planning
structure. It produces a task-by-task implementation plan — still without
writing any code.

### The Mode Tool

Agents can also manage modes programmatically using the `mode` tool. This
enables agent-initiated transitions during automated workflows:

| Operation | What It Does |
|-----------|-------------|
| `set` | Activate a mode by name |
| `clear` | Deactivate the current mode |
| `list` | Show all available modes with sources |
| `current` | Show the currently active mode |

The `set` operation has a built-in safety gate — the first call is blocked
with a reminder, and the agent must call again to confirm. This prevents
accidental mode transitions during complex reasoning chains.

## Creating Custom Modes

A mode is a markdown file with YAML frontmatter. Place it in `.amplifier/modes/`
for project scope or `~/.amplifier/modes/` for global scope.

Here's a complete custom mode for security-focused code review:

```markdown
---
mode:
  name: security-review
  description: Security-focused code audit with no modification capability
  shortcut: security-review

  tools:
    safe:
      - read_file
      - glob
      - grep
      - LSP
      - load_skill
    confirm:
      - bash

  default_action: block
---

SECURITY REVIEW MODE: Audit code for vulnerabilities. No modifications.

Focus areas:
- Authentication and authorization flows
- Input validation and sanitization
- Secret management (hardcoded keys, tokens, credentials)
- SQL injection, XSS, CSRF vectors
- Dependency vulnerabilities

For each finding, report:
1. Severity (critical / high / medium / low)
2. Location (file:line)
3. Description of the vulnerability
4. Recommended fix

Do NOT fix issues directly. Document them for human review.
```

Save it as `.amplifier/modes/security-review.md` and it's immediately available
via `/security-review`. The YAML frontmatter defines the tool policies; the
markdown body defines the agent's mission.

Bundles can also ship modes by including a `modes/` directory. Any `.md` file
in that directory with valid `mode:` frontmatter becomes available when the
bundle is active.

## Best Practices

**Default to block.** Set `default_action: block` and explicitly allowlist the
tools your mode needs. An explore mode that accidentally allows `bash` defeats
its purpose. Allowlists are safer than denylists.

**Match policies to intent.** Use `safe` for tools the mode encourages, `warn`
for tools that are allowed but discouraged, `confirm` for tools that need
human oversight, and `block` for tools that violate the mode's purpose.

**Write the body as instructions, not documentation.** The markdown body is
injected as a system reminder. Write it in imperative voice: "Focus on X,"
"Do NOT do Y," "Report findings as Z." The agent reads it as orders.

**Keep modes focused.** A mode that tries to govern everything governs nothing.
"Explore" does one thing: read-only navigation. "Careful" does one thing:
confirm before writing. Resist the urge to combine unrelated constraints.

**Use transitions for pipelines.** Modes like brainstorm define
`allowed_transitions` to guide the natural flow from design to planning to
execution. This keeps the workflow structured without requiring the user to
remember what comes next.

## Key Takeaways

1. **Modes are runtime overlays.** They modify tool access, inject context,
   and shape behavior — without changing the underlying bundle.

2. **Tool policies are the enforcement mechanism.** Four levels — safe, warn,
   confirm, block — give precise control over what the agent can do in each
   mode.

3. **Context injection is the guidance mechanism.** The markdown body becomes
   a system reminder that tells the agent *how* to work, not just *what* tools
   it has.

4. **Modes compose into pipelines.** Brainstorm → write-plan → execute-plan →
   verify → finish. Each mode constrains the agent to exactly what that phase
   requires.

5. **Custom modes are just markdown files.** YAML frontmatter for policies,
   markdown body for guidance, drop into `.amplifier/modes/` — immediately
   available.

6. **The mode tool enables programmatic transitions.** Agents can set, clear,
   list, and query modes, supporting automated workflows with built-in safety
   gates.

---

## Related Concepts

- [Skills](./skills.md) — Loadable expertise that modes can reference
- [Hooks](./hooks.md) — Lifecycle observers that modes use for context injection and policy enforcement
- [Bundles](./bundles.md) — Packaging modes with tools, agents, and configuration