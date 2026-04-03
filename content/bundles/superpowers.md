---
id: superpowers
type: bundle
title: "Superpowers Bundle"
---

# Superpowers Bundle

Most AI coding assistants will happily write code the moment you ask. No tests, no design, no review -- just output. It works until it doesn't, and when it doesn't, you're debugging AI-generated code you never fully understood in the first place.

The Superpowers bundle takes a different position: test-driven development is non-negotiable. Every feature starts with a failing test. Every implementation passes through a two-stage review. Every claim of "it works" requires evidence. This isn't extra process for the sake of process -- it's the methodology that makes AI-assisted development reliable enough to trust.

## What is the Superpowers Bundle?

Superpowers is a TDD-focused development methodology bundle. It layers a complete software engineering workflow on top of Foundation, enforcing discipline through modes, specialized agents, and approval gates.

The core philosophy rests on four principles:

- **Test-driven development is non-negotiable.** No production code without a failing test first. RED-GREEN-REFACTOR, always.
- **Systematic over ad-hoc.** Every phase has a defined process. Brainstorming has structure. Debugging has phases. Verification has evidence requirements.
- **Evidence over claims.** "Should work" is not evidence. Run the test. Read the output. Then -- and only then -- claim success.
- **Complexity reduction.** Break large problems into small, testable pieces. A task that can't be tested in isolation is a task that hasn't been decomposed enough.

## What's Included

Superpowers provides six modes, five agents, two recipes, and a methodology framework:

| Category | What It Provides |
|----------|-----------------|
| **Modes** | 6 runtime behavior overlays that enforce workflow discipline |
| **Agents** | 5 specialized agents for implementation, review, design, and planning |
| **Recipes** | Full development cycle (autopilot) and subagent-driven development |
| **Skills** | TDD methodology, integration testing discipline, reference tables |
| **Philosophy** | Core principles, anti-rationalization rules, the Gate Function |

The bundle includes Foundation and builds on its tools, agents, and delegation system -- everything Foundation provides, plus the methodology layer that makes development systematic.

## Getting Started

Include Superpowers in your bundle configuration:

```yaml
# .amplifier/config.yaml
bundles:
  - foundation
  - superpowers
```

Once loaded, Superpowers watches every prompt for mode applicability. Before starting any work, it checks: does a mode apply here? If even 1% likely, it suggests it.

> I need to add pagination to the users API endpoint.

```
This looks like feature work. I'd recommend starting with /brainstorm
to nail down the design, then /write-plan to break it into tasks,
then /execute-plan to implement with full TDD and review.

Want to start brainstorming?
```

The simplest way to begin is to describe what you want to build. Superpowers figures out which mode applies and guides you through the process.

## Modes

Modes are runtime behavior overlays -- they change how Amplifier works when activated. Each mode enforces specific rules and constraints.

### /brainstorm -- Collaborative Design

Activates before any coding begins. You and the assistant explore requirements, constraints, trade-offs, and edge cases through back-and-forth conversation. No code is written. The output is a validated design document.

> /brainstorm -- I want to add a caching layer to our API

```
Let's think through this. A few questions before we design anything:

1. What's your current request latency? Where's the bottleneck?
2. Are responses user-specific or shared across users?
3. What's your invalidation strategy -- time-based, event-based, or both?
4. What infrastructure do you already have? Redis? Memcached?
```

The assistant owns the conversation. When the design is validated section by section, it delegates to the `brainstormer` agent to produce the final design document.

### /write-plan -- Task Decomposition

Takes a validated design and breaks it into bite-sized, implementable tasks. Each task includes the exact test to write first and the exact code to make it pass. No vague descriptions -- every task is concrete enough for a fresh agent to execute without ambiguity.

> /write-plan -- based on the caching design we just finished

```
[Tool: task] Delegating to plan-writer...
  Created implementation plan: 8 tasks
  Task 1: Add CacheConfig dataclass with TTL, max_size, backend fields
  Task 2: Implement CacheBackend protocol with get/set/delete/clear
  Task 3: Build InMemoryCache implementing CacheBackend
  ...
  Each task has: failing test first, implementation code, acceptance criteria
```

### /execute-plan -- Three-Agent Pipeline

This is where code gets written -- but not by you. Every task goes through a three-agent pipeline:

1. **Implementer** writes the failing test, then makes it pass (RED-GREEN-REFACTOR)
2. **Spec-reviewer** checks: does the implementation match the spec?
3. **Code-quality-reviewer** checks: is the code clean, maintainable, and idiomatic?

Each task gets a fresh agent with clean context. You orchestrate; the agents execute.

> /execute-plan -- run the caching implementation plan

```
[Task 1/8] CacheConfig dataclass
  [Tool: task] implementer → RED: wrote test_cache_config_defaults... FAIL ✓
  [Tool: task] implementer → GREEN: implemented CacheConfig... PASS ✓
  [Tool: task] implementer → REFACTOR: extracted defaults to constants
  [Tool: task] spec-reviewer → Spec compliance: PASS (all fields match design)
  [Tool: task] code-quality-reviewer → Quality: PASS (clean, typed, documented)
```

### /debug -- Systematic Root Cause Analysis

When something breaks, don't guess. Debug mode enforces a four-phase framework: reproduce, hypothesize, verify, then fix. You own the investigation (phases 1-3), but the fix itself must be delegated.

> /debug -- the cache is returning stale data after invalidation

```
Phase 1 — Reproduce
  Running invalidation test... confirmed: stale data persists
Phase 2 — Hypothesize
  Three candidates: race condition, async flush delay, key mismatch
Phase 3 — Verify
  Added logging to delete path... key mismatch confirmed.
  Cache key uses request URL, but invalidation uses entity ID.
Phase 4 — Fix
  [Tool: task] Delegating fix to implementer...
```

### /verify -- Evidence-Based Completion

Before claiming anything is done, verify mode demands proof. Run the tests. Check the output. Confirm the behavior. "Should work" gets rejected -- only evidence counts.

> /verify -- is the caching implementation complete?

```
Running full test suite...     142 passed, 0 failed ✓
Running integration tests...   18 passed, 0 failed ✓
Checking coverage...           cache module: 94% ✓
Manual smoke test...           GET /users → 200 (cache HIT, 3ms) ✓

Verification: COMPLETE with evidence.
```

### /finish -- Clean Completion

Wraps up the development branch. Verifies everything one final time, then presents options: merge to main, create a PR, keep the branch, or discard.

> /finish

```
Final verification: all tests pass, no lint errors, no type errors.
Options: 1) Merge to main  2) Create PR  3) Keep branch  4) Discard
```

## Key Agents

Five agents, each with a specific role in the pipeline:

| Agent | Role | When Used |
|-------|------|-----------|
| **brainstormer** | Writes design documents from validated conversations | After `/brainstorm` conversation completes |
| **plan-writer** | Creates detailed implementation plans with TDD tasks | After `/write-plan` conversation completes |
| **implementer** | Implements tasks following strict RED-GREEN-REFACTOR | Every task in `/execute-plan` |
| **spec-reviewer** | Reviews implementation against the spec | After implementer, every task |
| **code-quality-reviewer** | Reviews code quality, style, and best practices | After spec-reviewer, every task |

The two-stage review is critical. Spec compliance checks "did you build the right thing?" Code quality checks "did you build it well?" These are separate concerns that require separate reviewers. The spec-reviewer always runs first -- there's no point reviewing code quality on something that doesn't meet the spec.

## Recipes

### Full Development Cycle (Autopilot)

The autopilot recipe runs the entire pipeline end-to-end with approval gates at key transitions:

```
recipes execute @superpowers:recipes/superpowers-full-development-cycle.yaml
```

This recipe walks through brainstorm → write-plan → execute-plan → verify → finish, pausing for your approval between stages. The gates ensure you stay in control while the system does the heavy lifting.

### Subagent-Driven Development

For executing an existing plan in the current session:

```
recipes execute @superpowers:recipes/subagent-driven-development.yaml
```

This recipe iterates through plan tasks, spinning up a fresh implementer for each one, followed by both reviewers. It's the inner loop of execute-plan, packaged as a standalone recipe.

## Two Tracks: Autopilot vs. Manual

Superpowers supports two working styles:

**Autopilot (recipe-driven)** -- For feature development, use the full development cycle recipe. It handles mode transitions, approval gates, and agent delegation automatically. You describe what you want, approve at checkpoints, and get tested, reviewed code.

**Manual (mode-by-mode)** -- For ad-hoc work, activate modes directly. Need to brainstorm? Type `/brainstorm`. Found a bug? Type `/debug`. Fine-grained control when you don't need the full pipeline. Most feature work benefits from autopilot; most exploratory or maintenance work benefits from manual.

## Methodology Calibration

Not every task needs the full ceremony. Superpowers calibrates rigor to task size:

| Task | Appropriate Process |
|------|-------------------|
| Fix a typo in a docstring | Just fix it. No mode needed. |
| Add a small utility function | `/execute-plan` with TDD. Skip brainstorm if obvious. |
| Build a new feature | Full cycle: `/brainstorm` → `/write-plan` → `/execute-plan` → `/verify` → `/finish` |
| Refactor a subsystem | Full cycle with extra design attention in brainstorm. |
| Debug a production issue | `/debug` immediately. Four phases, no shortcuts. |

The rule is simple: match the process to the risk. A typo fix doesn't need a design document. A new authentication system does.

## The Gate Function

Every claim of completion -- at any level -- must pass through the same four-step gate:

1. **Identify** -- What specific command or test proves this works?
2. **Run** -- Execute that command. Actually run it, don't imagine the output.
3. **Read** -- Read the actual output. Don't assume it says what you expect.
4. **Verify** -- Does the output prove the claim? Only then assert success.

This applies everywhere: individual tests, task completion, mode transitions, final verification. The gate function is why Superpowers catches issues that ad-hoc development misses -- it replaces hope with evidence.

## Tips

- **Trust the process, especially when it feels slow.** Writing the failing test first feels like overhead until it catches a design flaw in minute two instead of hour two.

- **Don't skip brainstorm for "obvious" features.** The questions you didn't think to ask are the ones that create bugs. If the design truly is obvious, the conversation will be short.

- **Let the three-agent pipeline work.** The implementer, spec-reviewer, and code-quality-reviewer exist because a single agent writing and reviewing its own code has blind spots. Fresh eyes catch what familiarity misses.

- **Use `/debug` before guessing.** When something breaks, the instinct is to start changing things. Resist it. Reproduce first. Hypothesize second. Verify third. Fix last.

- **Approval gates are checkpoints, not obstacles.** When the recipe pauses for approval, read what it produced. This is your chance to course-correct before the system commits more effort.

## Next Steps

- Learn about [Foundation Bundle](./foundation.md) -- the base layer Superpowers builds on
- Explore [Recipes](./recipes.md) -- the engine behind autopilot workflows
- Read about [Modes](../concepts/modules.md) -- how runtime behavior overlays work
- See the [Task Tool](../tools/task.md) -- the delegation mechanism agents use
