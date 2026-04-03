---
id: first-conversation
type: quickstart
title: "Your First Conversation"
---

# Your First Conversation

You've got Amplifier installed. Now let's actually use it. This page walks you through a complete first session — from launching the CLI to having a multi-turn conversation where Amplifier reads your files, runs commands, and delegates to specialized agents on your behalf.

## What You'll Learn

By the end of this page, you'll know how to:

- Start an interactive session and a single-shot command
- Read Amplifier's tool indicators to understand what it's doing behind the scenes
- Have a multi-turn conversation that builds on previous context
- Trigger agent delegation for deeper investigation
- Use slash commands for session control
- End a session cleanly

---

## Launch Interactive Mode

Open a terminal, navigate to a project directory (any project — a personal repo, a work project, even a folder with a few scripts), and start Amplifier:

```bash
amplifier
```

That's it. No subcommand, no flags. You'll land at an interactive prompt where you can type naturally. Amplifier remembers everything you say during the session, so each message builds on what came before.

## Step 1: Ask Something Simple

Start with a straightforward question about your surroundings. You're orienting Amplifier to your workspace — and orienting yourself to how it responds.

> What's in the current directory? Give me a quick overview.

Behind the scenes, Amplifier reaches for its tools to answer you. You'll see indicators like these flash by:

```
[Tool: glob] **/* — found 47 files
[Tool: read_file] README.md
[Tool: read_file] pyproject.toml
```

Then it responds with a structured summary: file counts by type, the project's purpose (pulled from your README), key entry points, and dependencies. It's not guessing — it actually read your files.

Notice what happened. You asked a natural question, and Amplifier decided *which* tools to use, *what* to read, and *how* to summarize it. You didn't need to specify any of that. The tool indicators tell you exactly what it did, so there's no mystery.

## Step 2: Dig Into a Specific File

Now pick something from the overview that looks interesting and ask about it. This is where multi-turn context kicks in — Amplifier remembers the overview it just gave you.

> Explain the main entry point. What happens when someone runs this project?

```
[Tool: read_file] src/main.py
[Tool: grep] "def main" — 3 matches
[Tool: read_file] src/config.py
```

Amplifier reads the file, traces the execution flow, and explains it in plain language — what gets imported, what runs first, how configuration loads. If it needs to chase an import into another file, it does that automatically.

This is the rhythm of working with Amplifier: ask, watch it gather information, read the explanation, then follow up on whatever catches your eye.

## Step 3: Trigger Agent Delegation

Here's where things get interesting. Ask Amplifier to do something that requires deeper investigation — the kind of task that benefits from a specialist.

> This codebase is new to me. Can you map out the architecture and tell me how the main components fit together?

For a broad exploration like this, Amplifier delegates to the **explorer** agent — a specialist tuned for mapping codebases:

```
[Tool: task] Delegating to foundation:explorer
  └─ "Map the architecture and explain how main components fit together"
```

The explorer works in its own sub-session: reading files, tracing imports, scanning directory structure, building a mental model. When it finishes, the results flow back to your main conversation. You'll see a structured summary with cited file paths, component relationships, and key observations.

Agent delegation happens automatically when Amplifier judges a task benefits from a specialist. You can also request it explicitly:

> Use bug-hunter to investigate the error handling in the auth module.

```
[Tool: task] Delegating to foundation:bug-hunter
  └─ "Investigate error handling in the auth module"
```

To see what agents are available in your current session, type `/agents` at the prompt.

## Step 4: Keep the Conversation Going

The real power shows up in multi-turn conversations. Each message builds on everything that came before. After the explorer maps your codebase, you might say:

> The database layer seems tightly coupled to the API routes. What would it take to add a service layer in between?

Amplifier doesn't need you to re-explain the codebase. It already has the explorer's findings in context. It might read a few more files to confirm specifics, then give you a concrete plan — which files to change, what the new layer looks like, what the migration path is.

Follow up again:

> Show me what the service layer would look like for the user module specifically.

```
[Tool: read_file] src/routes/users.py
[Tool: read_file] src/models/user.py
```

Now it's writing code, informed by everything it learned in the previous turns. That's the value of a sustained conversation — context accumulates and every answer gets more precise.

## Single-Shot Mode

Not everything needs a conversation. For quick one-off tasks, use `amplifier run` with a prompt in quotes:

```bash
amplifier run "How many Python files are in this project?"
```

Amplifier processes the prompt, uses whatever tools it needs, gives you the answer, and exits. No interactive session, no follow-up — just the result.

This is handy for quick checks, scripting, and CI/CD pipelines. You can also pipe content in:

```bash
amplifier run "Explain what this does" < src/utils.py
```

## Slash Commands

During an interactive session, slash commands give you control without interrupting the conversation:

| Command | What It Does |
|---------|-------------|
| `/help` | Show all available slash commands |
| `/tools` | List the tools Amplifier can use |
| `/agents` | List available specialist agents |
| `/status` | Show current session info |
| `/think` | Enter read-only planning mode (no file changes) |
| `/do` | Exit planning mode (allow changes again) |
| `/save` | Save a conversation transcript |
| `/clear` | Clear conversation context and start fresh |

Try typing `/tools` during your session to see what's available. It's a good way to understand what Amplifier can do.

## Ending a Session

When you're done, exit cleanly with either:

- Type `exit` at the prompt
- Press `Ctrl+C`

Your session is saved automatically. Come back later and pick up where you left off:

```bash
amplifier continue
```

Or resume a specific past session:

```bash
amplifier session list
amplifier session resume <session-id>
```

---

## Verify

After your first session, you should be able to confirm:

- **You started a session** with just `amplifier` and got an interactive prompt
- **You saw tool indicators** like `[Tool: read_file]` and `[Tool: grep]` showing what happened behind the scenes
- **You had a multi-turn conversation** where later answers built on earlier context
- **You saw agent delegation** (or understand how to trigger it with exploration or debugging requests)
- **You exited cleanly** with `exit` or `Ctrl+C`

## Try It Out

Open Amplifier in a project you know well and try these prompts:

> What does this project do? Summarize it from the code, not just the README.

> Find the most complex function in this codebase and explain what it does.

> Use zen-architect to evaluate whether this project's structure follows good practices.

> What tests exist? Run them and tell me if anything fails.

Each of these exercises a different capability: file reading, code analysis, agent delegation, and command execution.

## Common Issues

**"It gave a vague answer without reading any files."**
Your prompt was probably too abstract. Instead of "tell me about this project," try "read the files in src/ and explain what this project does." Giving Amplifier a concrete starting point — a directory, a file, a function name — produces much better results.

**"It read the wrong files."**
Be specific about where to look. "Explain the auth system" is ambiguous — it might start anywhere. "Explain how authentication works starting from src/auth/middleware.py" points it exactly where you want.

**"The agent delegation seemed slow."**
Agents do real work — reading many files, tracing relationships, building understanding. A thorough exploration of a large codebase takes time. That's normal. For quick questions, talk to Amplifier directly without triggering delegation.

**"I lost my session."**
Sessions auto-save. Run `amplifier session list` to find recent sessions and `amplifier session resume <id>` to pick one up. Or just `amplifier continue` to resume the most recent one.

**"It wants to make changes but I just want to explore."**
Type `/think` to enter read-only planning mode. Amplifier will analyze and suggest but won't modify any files. Type `/do` to allow changes again when you're ready.

## Next Steps

You've had your first real conversation. Here's where to go from here:

1. **[Key Commands](./key-commands.md)** — Master the CLI commands for daily use
2. **[Your First Bundle](./first-bundle.md)** — Customize what tools and agents are available
3. **[Core Concepts](../concepts/index.md)** — Understand the architecture underneath
4. **[Agents](../concepts/agents.md)** — Learn about the specialist agents and when to use them
