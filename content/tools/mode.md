---
id: tool-mode
type: tool
title: "Mode Tool"
---

# Mode Tool

Amplifier doesn't work the same way all the time. Sometimes you want free-form brainstorming with no guardrails. Other times you want strict code review with specific policies enforced. Modes are how you shift between these ways of working — they inject context, enforce tool policies, and change how Amplifier behaves for the duration of an activity.

The Mode tool is the mechanism for activating, deactivating, and inspecting these modes. It's simple on the surface — four operations — but the gate policy and context injection it controls have real effects on your session.

## What Is a Mode?

A mode is a named configuration that, when active, does two things:

1. **Injects context** — instructions, constraints, and framing that guide how Amplifier approaches your requests
2. **Enforces tool policies** — rules about which tools are available, restricted, or modified while the mode is active

For example, a "brainstorm" mode might inject context that says "explore ideas freely, don't write code yet" and restrict file-writing tools. A "code-review" mode might inject review checklists and enable stricter diagnostics.

Modes are defined in bundles and discovered automatically. You don't create them through the Mode tool — you activate modes that already exist in your configured bundles.

## Core Capabilities

The Mode tool has four operations:

### list — See What's Available

Before you can activate a mode, you need to know what modes exist. `list` shows every mode available in your current session:

> What modes can I use?

```
[Tool: mode] list
  Available modes:
    brainstorm    — Free-form ideation, no code writing
    code-review   — Structured code review with checklists
    planning      — Architecture and design planning
    debug         — Focused debugging with diagnostic tools
    writing       — Documentation and content creation
```

The available modes depend on which bundles are loaded. Different projects may have different modes.

### current — Check What's Active

> What mode am I in right now?

```
[Tool: mode] current
  Active mode: brainstorm
  Since: 2 minutes ago
```

If no mode is active, you'll get a clear "no active mode" response. Only one mode can be active at a time.

### set — Activate a Mode

This is where the gate policy matters. Activating a mode changes how your session behaves, so Amplifier uses a **confirmation gate** to prevent accidental transitions.

> Switch to code-review mode

```
[Tool: mode] set  name: "code-review"
  ⚠ Gate policy: warn
  Mode "code-review" will change session behavior:
    - Injects code review checklist context
    - Enables strict diagnostics
  Retry to confirm activation.
```

The first `set` request is **blocked by default**. This is the "warn" gate policy — it tells you what's about to change and asks you to confirm. Retry the same request to proceed:

> Yes, switch to code-review mode

```
[Tool: mode] set  name: "code-review"
  ✓ Mode "code-review" activated.
  Context injected: code review checklist, quality standards
  Tool policies active: diagnostics enabled after every edit
```

The two-step confirmation exists because mode transitions aren't free — they change context injection and tool policies, which affect everything that follows. You don't want to accidentally enter a restrictive mode when you meant to keep working freely.

### clear — Deactivate the Current Mode

> Exit code-review mode

```
[Tool: mode] clear
  ✓ Mode "code-review" deactivated.
  Session returned to default behavior.
```

Clearing removes the injected context and lifts any tool policy restrictions. You're back to baseline.

## The Gate Policy

The gate policy is the most important thing to understand about modes. By default, the policy is **"warn"** — meaning the first activation attempt is blocked with a warning, and you must retry to confirm.

This matters because:

- **Context injection is sticky.** Once a mode injects context, that context colors every response until you clear the mode. Accidentally entering a mode means working with the wrong framing.
- **Tool policies restrict capabilities.** A mode might disable file writing or enforce mandatory checks. Entering the wrong mode can block you from tools you need.
- **Mode transitions are meaningful.** Switching from brainstorm to code-review isn't just a label change — it fundamentally shifts how Amplifier operates.

The warn gate catches the most common mistake: typing a mode name when you meant something else, or confirming a mode switch suggested by an agent without understanding what it does.

### How Agents Handle Gates

When agents activate modes programmatically, the same gate policy applies. An agent's first `set` call gets the warning; it must retry to confirm. This prevents automated scripts from accidentally switching modes without the gate check.

In practice, well-designed agent workflows handle this by:
1. Calling `set` and receiving the warning
2. Evaluating whether the mode transition is appropriate
3. Retrying to confirm, or choosing not to proceed

## Context Injection in Practice

When a mode is active, its injected context is prepended to the system prompt. This means Amplifier "sees" the mode's instructions before it processes your request. The effect is subtle but pervasive:

**Without a mode active:**
> Review this function for bugs

Amplifier gives a general review based on its training.

**With code-review mode active:**
> Review this function for bugs

Amplifier follows the injected checklist — checking error handling, edge cases, type safety, performance implications, and security concerns in a structured format. The mode's context told it *how* to review, not just *that* you asked for a review.

This is why modes are more powerful than just asking for a different style. The injected context persists across your entire conversation until the mode is cleared, providing consistent behavior without you having to repeat instructions.

## Tool Policy Enforcement

Modes can modify which tools are available and how they behave:

| Policy Type | Example | Effect |
|------------|---------|--------|
| **Restrict** | Brainstorm mode disables `write_file` | Prevents premature implementation during ideation |
| **Require** | Debug mode requires `diagnostics` after edits | Enforces verification discipline automatically |
| **Modify** | Review mode changes `grep` defaults | Searches test files by default for coverage checks |

You don't configure these policies through the Mode tool — they're defined in the mode's bundle configuration. The tool simply activates and deactivates the mode; the policies come along for the ride.

## Using Modes via /mode

In addition to the tool interface, you can activate modes with the `/mode` command in conversation:

> /mode brainstorm

This is equivalent to calling the Mode tool's `set` operation. The same gate policy applies — you'll get a warning on the first attempt and need to confirm.

The `/mode` shorthand is convenient for quick switches. Use the tool interface when you need to inspect available modes (`list`) or check the current state (`current`) programmatically.

## Practical Examples

### Brainstorming Session

> /mode brainstorm

```
⚠ Mode "brainstorm" will enable free-form ideation. Retry to confirm.
```

> /mode brainstorm

```
✓ Mode "brainstorm" activated.
```

> How should we restructure the payment system?

Amplifier explores ideas freely, suggests multiple approaches, and avoids jumping to implementation. When you're done ideating:

> /mode clear

```
✓ Brainstorm mode deactivated.
```

Now you can switch to implementation with a clean slate.

### Checking Before Acting

When you're unsure what mode you're in — especially in a long conversation — check first:

> What mode am I in?

```
[Tool: mode] current
  No active mode.
```

> What modes are available?

```
[Tool: mode] list
  Available modes: brainstorm, code-review, planning, debug, writing
```

This inspect-then-decide pattern prevents confusion in long sessions where you might have forgotten a mode was still active.

## Tips

**Always check current mode in long sessions.** A mode activated 50 messages ago is still active. If responses seem oddly constrained or oddly loose, check `current` — you might be in a mode you forgot about.

**Clear before switching.** While `set` will replace the active mode, explicitly clearing first makes the transition clearer in your conversation history and avoids any context bleed.

**Let the gate protect you.** The two-step confirmation feels like friction, but it's there for a reason. Mode transitions change real behavior. The one time the gate stops an accidental switch is worth the extra confirmation every other time.

**Use modes for sustained activities, not one-off requests.** If you need one code review, just ask for it. If you're doing a two-hour review session, activate code-review mode. Modes pay off over multiple interactions.

**Check `list` in new projects.** Different bundles provide different modes. When you start working in a new codebase, `list` tells you what's available — you might discover modes you didn't know existed.

## Next Steps

- Learn about [Bundles](../concepts/bundles.md) to understand where modes are defined and how they're configured
- See [Hooks](../concepts/hooks.md) for the mechanism that injects mode context into your session
- Explore the [Bash Tool](./bash.md) and other tools to see how tool policies interact with mode restrictions
