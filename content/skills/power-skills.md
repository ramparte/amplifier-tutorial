---
id: power-skills
type: skill
title: Power Skills
---

# Power Skills

Most skills teach the agent how to think about a domain. Power skills go further — they take over, spawning dedicated agents that work in parallel to accomplish complex tasks. Amplifier ships with three built-in power skills that handle code review, large-scale changes, and session diagnostics.

These aren't auto-suggested. You invoke them explicitly with slash commands: `/code-review`, `/mass-change`, `/session-debug`. When triggered, they fork execution, spin up specialist agents, and return consolidated results. Think of them as pre-built workflows that happen to be invoked like skills.

## How Power Skills Work

Power skills differ from regular skills in two critical ways:

1. **They fork execution** — each spawned agent runs in its own context (`context: fork`), isolated from your main session
2. **They disable model invocation** — the spawned agents use `disable-model-invocation: true`, meaning they operate with their own model calls, not yours

```
┌─────────────────┐
│  Your Session    │
│                  │
│  > /code-review  │
│                  │
│  ┌───┬───┬───┐  │
│  │ A │ B │ C │  │  ← 3 forked agents (parallel)
│  └─┬─┘ └─┬─┘ └─┬┘
│    │     │     │
│    ▼     ▼     ▼
│  ┌─────────────┐ │
│  │  Merged     │ │  ← Results combined back
│  │  Results    │ │
│  └─────────────┘ │
└─────────────────┘
```

Each forked agent gets its own context window, its own tool access, and its own model calls. Your session waits for all agents to finish, then presents the merged output. You stay in control — the agents do the heavy lifting.

## /code-review

Spawns three parallel review agents, each examining your recent changes from a different angle. Instead of one generic review pass, you get three specialized perspectives.

### The Three Reviewers

| Agent | Focus | What It Catches |
|-------|-------|-----------------|
| **Reuse Analyst** | Code duplication and reuse opportunities | Repeated patterns, missed abstractions, existing utilities that could replace new code |
| **Quality Auditor** | Code quality and maintainability | Naming issues, complexity hotspots, missing error handling, test gaps |
| **Efficiency Reviewer** | Performance and resource efficiency | Unnecessary allocations, N+1 queries, blocking calls, cache opportunities |

### Example

> /code-review

```
[Tool: bash] git diff HEAD~3 --stat
 src/auth/login.py      | 47 ++++++++++++
 src/auth/session.py    | 23 ++++++
 src/api/middleware.py   | 15 ++++
 tests/test_auth.py     | 38 ++++++++++
 4 files changed, 123 insertions(+)
```

Amplifier spawns three agents. Each reviews the same diff independently:

```
── Reuse Analyst ──────────────────────────────────
⚠ src/auth/login.py:12 — Token validation logic duplicates
  src/auth/session.py:8. Extract to shared validator.
✓ Good reuse of existing middleware patterns in api/middleware.py.

── Quality Auditor ────────────────────────────────
⚠ src/auth/login.py:34 — bare except clause. Catch specific
  exceptions (AuthError, TokenExpired).
⚠ src/auth/session.py:19 — missing docstring on public method.
✓ Test coverage looks solid — all new paths have assertions.

── Efficiency Reviewer ────────────────────────────
⚠ src/auth/login.py:28 — database query inside a loop.
  Consider batch lookup.
✓ Session creation uses connection pooling correctly.
✓ No unnecessary allocations detected.
```

After all three complete, Amplifier asks if you want to apply fixes:

> Apply all suggested fixes

```
[Tool: edit_file] src/auth/login.py — extracted shared validator
[Tool: edit_file] src/auth/login.py — replaced bare except
[Tool: edit_file] src/auth/session.py — added docstring
[Tool: edit_file] src/auth/login.py — converted loop query to batch
```

Four files updated. Three reviewers, one command.

## /mass-change

When a change touches dozens of files — a dependency upgrade, an API migration, a rename that ripples across the codebase — doing it sequentially is slow. `/mass-change` decomposes the work into independent units and runs them in parallel.

### How It Works

1. **Analysis** — scans the codebase to understand the full scope of the change
2. **Decomposition** — breaks the change into 5–30 independent units, each touching a small set of files
3. **Parallel execution** — spawns one agent per unit, all working simultaneously
4. **Verification** — each agent runs tests for its unit before reporting completion

```
> /mass-change Migrate all datetime.utcnow() calls to datetime.now(timezone.utc)
```

```
── Analysis ───────────────────────────────────────
Scanning codebase for datetime.utcnow() usage...
Found 47 call sites across 23 files in 8 modules.

── Decomposition ──────────────────────────────────
Grouped into 8 independent units by module:
  Unit 1: src/auth/     (6 files, 11 call sites)
  Unit 2: src/api/      (4 files, 8 call sites)
  Unit 3: src/billing/  (3 files, 7 call sites)
  Unit 4: src/jobs/     (3 files, 6 call sites)
  Unit 5: src/notify/   (2 files, 5 call sites)
  Unit 6: src/export/   (2 files, 4 call sites)
  Unit 7: src/admin/    (2 files, 4 call sites)
  Unit 8: tests/        (1 file, 2 call sites)

── Parallel Execution (8 agents) ──────────────────
  ✓ Unit 1: src/auth/      — 11/11 migrated, tests pass
  ✓ Unit 2: src/api/       — 8/8 migrated, tests pass
  ✓ Unit 3: src/billing/   — 7/7 migrated, tests pass
  ✓ Unit 4: src/jobs/      — 6/6 migrated, tests pass
  ✓ Unit 5: src/notify/    — 5/5 migrated, tests pass
  ✓ Unit 6: src/export/    — 4/4 migrated, tests pass
  ✓ Unit 7: src/admin/     — 4/4 migrated, tests pass
  ✓ Unit 8: tests/         — 2/2 migrated, tests pass

All 47 call sites migrated. 23 files updated. Full test suite passing.
```

Each agent works in isolation — if Unit 3 has a tricky edge case, it doesn't block Units 1 through 8. The decomposition ensures units don't touch overlapping files, so there are no merge conflicts.

### When to Use It

- Dependency upgrades that change an API surface
- Renaming a concept across the entire codebase
- Migrating from one pattern to another (e.g., callbacks to async/await)
- Applying a new linting rule that requires code changes

### When Not to Use It

If the change can't be decomposed into independent units — for example, a refactor where every file depends on a new shared interface you're still designing — do it sequentially. `/mass-change` needs independence between units.

## /session-debug

When something in your Amplifier session isn't working — a tool fails silently, a hook misbehaves, the wrong provider is being selected — `/session-debug` spawns a diagnostic agent that investigates the issue from the inside.

### What It Checks

The session-analyst agent inspects:

- **Tool registration** — are all expected tools mounted and responding?
- **Provider connectivity** — can the configured provider reach its API?
- **Hook chain** — are hooks firing in the expected order?
- **Bundle conflicts** — are multiple bundles trying to configure the same thing?
- **Context health** — is the session context within token limits?
- **Module errors** — any modules that failed to mount or threw during execution?

### Example

> /session-debug The LSP tool isn't finding any references

```
── Session Analyst ────────────────────────────────

Checking tool registration...
  ✓ 14 tools registered
  ✓ LSP tool is mounted

Checking LSP configuration...
  ⚠ LSP server for Python: not running
    Expected: pyright-langserver at port 4826
    Status: process not found

  Root cause: The Python language server hasn't been started
  for this workspace.

  Fix: Run the LSP initialization:
```

```
[Tool: bash] cd /home/user/project && pyright-langserver --stdio &
```

```
── Session Analyst (continued) ─────────────────────

Retesting LSP...
  ✓ Python language server: connected
  ✓ findReferences: responding (tested on 3 symbols)

Issue resolved. LSP tool is now fully operational.
```

One command diagnosed the problem, identified the root cause, and fixed it.

## Invocation and Behavior

All three power skills share these characteristics:

| Property | Value |
|----------|-------|
| **Trigger** | Slash command only (`/code-review`, `/mass-change`, `/session-debug`) |
| **Auto-suggested** | No — never triggered automatically |
| **Execution model** | Forked context (`context: fork`) |
| **Model isolation** | `disable-model-invocation: true` per agent |
| **Parallelism** | Yes, except `/session-debug` (single agent) |
| **Session impact** | Forked agents don't pollute your main context |

Because forked agents run in isolation, they don't consume your session's context window. A `/mass-change` spawning 15 agents doesn't make your session 15x larger — each agent has its own window that's cleaned up when it finishes.

## Next Steps

For the conceptual foundation behind skills, see [Skills](../concepts/skills.md). To browse other available skills, check the [Skills Guide](./index.md). For building your own power skill, see [Custom Recipe](../advanced/custom-recipe.md) — power skills are recipes with forked execution.
