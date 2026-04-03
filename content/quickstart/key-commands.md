---
id: key-commands
type: quickstart
title: "Key Commands"
---

# Key Commands

You've got Amplifier installed and you've had your first conversation. Now let's build muscle memory around the commands you'll use every day. This page walks through each one with real examples so you can see how they feel in practice.

## What You'll Learn

- How to start Amplifier in interactive and one-shot modes
- How to resume previous sessions without losing context
- The slash commands that control workflows inside a session
- Keyboard shortcuts that keep you moving fast
- How to keep the CLI updated and reset when things go sideways

## Starting Sessions

There are two ways to kick things off: interactive mode for exploratory work, and one-shot mode for quick tasks.

### Interactive mode

Just type `amplifier` with no arguments:

```bash
amplifier
```

> Welcome to Amplifier. Type a message to begin.

You're now in a chat session. Type naturally, use slash commands, and keep going as long as you need. This is your daily driver for development work — debugging, building features, exploring code.

### One-shot mode

When you have a specific task and don't need a back-and-forth conversation, pass your prompt directly:

```bash
amplifier run "Add error handling to the parse_config function in src/config.py"
```

Amplifier reads the file, makes the changes, and exits. You get a session ID in the output in case you want to pick up where it left off.

> Session s_a3f29b complete.

This is great for scripting, CI pipelines, or quick fire-and-forget tasks.

### Running with a specific bundle

Bundles control which tools and agents are available. If you need capabilities from a particular bundle without changing your default, pass it directly:

```bash
amplifier run --bundle recipes "Plan a database migration from PostgreSQL to MySQL"
```

> [Tool: recipes] Loading recipe capabilities...
>
> Here's a phased migration plan...

The `--bundle` flag is temporary — your default bundle stays the same for next time. This is handy when you want the recipes agents for one task but normally use the dev bundle.

## Managing Sessions

Sessions persist automatically. Every conversation — whether interactive or one-shot — gets a session ID. You can come back to any of them.

### Resume your last session

The simplest resume: pick up exactly where you left off.

```bash
amplifier resume
```

> Resuming session s_a3f29b...
>
> Welcome back. Last time we were adding error handling to parse_config.

This loads the full conversation history so Amplifier remembers what you were working on, what files it changed, and what decisions you made together.

### Resume a specific session

If you've been juggling multiple tasks, pass the session ID directly:

```bash
amplifier resume s_7c41de
```

> Resuming session s_7c41de...
>
> We were debugging the authentication middleware. The failing test was in test_auth.py line 84.

You can find session IDs from the output of previous runs or from `amplifier session list`.

### Finding past sessions

```bash
amplifier session list
```

This shows your recent sessions with timestamps and the opening prompt, so you can find the right one to resume.

## Slash Commands — Workflows Inside a Session

Once you're in an interactive session, slash commands give you control without leaving the conversation. Think of them as mode switches and power tools.

### Getting help

Lost? Start here:

> /help

This prints every slash command available in your current session, grouped by category. The list depends on your active bundle — some bundles add extra commands.

### Modes — changing how Amplifier thinks

Modes shift Amplifier's behavior for specific tasks. Activate one with `/mode`:

> /mode brainstorm

> Brainstorm mode active. I'll explore ideas broadly without committing to implementations.

Now Amplifier generates options, asks questions, and avoids jumping straight to code. When you're done brainstorming:

> /mode off

> Mode deactivated. Back to normal operation.

Here are the modes you'll use most:

**brainstorm** — Divergent thinking. Amplifier explores possibilities instead of building.

> /mode brainstorm
>
> What are three different approaches to caching user sessions?

**debug** — Systematic root-cause analysis. Amplifier follows evidence methodically instead of guessing.

> /mode debug
>
> The API returns 500 errors intermittently but only under load.

**verify** — Double-check mode. Amplifier validates its own work and looks for issues.

> /mode verify
>
> Check that the refactoring we just did didn't break any existing behavior.

**execute-plan** — Heads-down implementation. Amplifier follows a plan step by step with minimal detours.

> /mode execute-plan
>
> Implement the caching strategy we agreed on in brainstorm mode.

**finish** — Wrap-up mode. Amplifier reviews changes, runs checks, and prepares work for commit.

> /mode finish
>
> Let's make sure everything is clean and ready to merge.

To see every mode available in your session:

> /modes

> Available modes: brainstorm, debug, verify, execute-plan, finish

### Code review

The `/code-review` command runs a structured review of your changed code — checking for bugs, style issues, missed edge cases, and opportunities to simplify:

> /code-review

> [Tool: bash] Running git diff to find changes...
>
> [Tool: read_file] Reading modified files...
>
> **Code Review Summary:**
> - src/auth.py: The token expiry check on line 47 doesn't handle the case where `expires_at` is None
> - src/handlers.py: Good error handling, but the retry logic could be extracted into a shared utility
> - tests/test_auth.py: Missing test for expired token edge case

This is one of the most useful commands when you're wrapping up a feature. It catches things you'd normally find in a PR review — before you open the PR.

### Mass changes

When you need to make the same kind of change across many files — renaming a pattern, updating an API, migrating a dependency — `/mass-change` parallelizes the work:

> /mass-change

> What change do you need to make?

> Rename all instances of `userId` to `user_id` across the Python codebase to match our style guide.

> [Tool: grep] Finding all occurrences...
>
> Found 47 occurrences across 12 files. I'll handle these in parallel batches.

The command researches the scope of the change, plans the approach, and then executes across files in parallel. Each change gets its own isolated context so mistakes in one file don't cascade.

### Session debugging

If something feels off — a tool isn't working, responses seem wrong, or you're getting unexpected errors — the `/session-debug` command diagnoses the problem:

> /session-debug

> Checking session health...
>
> - Provider: Anthropic (connected)
> - Bundle: dev (loaded, 12 tools active)
> - Working directory: /home/user/project (exists, writable)
> - Issue detected: bash tool timeout is set to 5s, which may be too low for test runs

This saves you from guessing. Instead of restarting or reinstalling, you get a targeted diagnosis.

## Keyboard Shortcuts

Two shortcuts you'll use constantly:

**Ctrl+C** — Exit the current session. If Amplifier is mid-response, the first Ctrl+C interrupts the current operation. A second Ctrl+C exits the session entirely.

**Up arrow** — Scroll through your input history, just like in a regular shell. This is invaluable when you're iterating: tweak your last prompt and send it again.

## Keeping Amplifier Updated

Amplifier moves fast. New tools, bug fixes, and model improvements land regularly. Update with:

```bash
amplifier update
```

> Updating Amplifier...
>
> Updated from v0.8.2 to v0.8.5. See changelog for details.

Run this periodically — especially if you hit a bug that might already be fixed.

### Resetting when things go wrong

If Amplifier gets into a weird state — stale caches, corrupted config, modules not loading — reset it:

```bash
amplifier reset
```

This clears cached data and resets configuration to defaults. Your session history is preserved. It's less drastic than a full reinstall but fixes most "something is off" problems.

If `reset` doesn't do it, see the [Clean Reinstall](./installation.md#clean-reinstall) section on the Installation page.

## Try It Out

Work through these exercises to get the commands into your fingers:

1. **Start interactive mode** — run `amplifier` and ask it to list files in your current directory. Then exit with Ctrl+C.

2. **Try one-shot mode** — run `amplifier run "What language are the files in this directory?"` and see it analyze your project.

3. **Resume the session** — run `amplifier resume` to pick up the one-shot session. Ask a follow-up question about the files it found.

4. **Use a mode** — in an interactive session, type `/mode brainstorm` and ask "What would be a good way to organize this project?" Then deactivate with `/mode off`.

5. **Check for updates** — run `amplifier update` to make sure you're on the latest version.

## Next Steps

You now have the core commands down. Here's where to go from here:

1. **[Your First Bundle](./first-bundle.md)** — Learn how bundles control what tools and agents you have access to
2. **[Core Concepts](../concepts/index.md)** — Understand the architecture underneath: bundles, tools, agents, and how they compose
3. **[Tools Reference](../tools/index.md)** — Deep dive into the tools available in your sessions
