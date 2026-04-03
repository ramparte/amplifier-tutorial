---
id: tool-bash
type: tool
title: "Bash Tool"
---

# Bash Tool

Every tool in Amplifier is designed for a specific job — reading files, searching code, navigating symbols. But sometimes you just need to run a shell command. That's what the Bash tool is for: a general-purpose fallback that gives you direct access to the shell when no specialized tool fits.

The key word there is *fallback*. Amplifier's specialized tools are almost always a better choice — they have structured output, better error handling, and domain-specific intelligence built in. Bash is for everything else: running your test suite, installing a package, checking git status, or spinning up a dev server.

## Core Capabilities

The Bash tool takes three parameters:

- **command** (required) — the shell command to execute
- **timeout** (default: 30 seconds) — how long to wait before killing the process
- **run_in_background** (default: false) — returns immediately with a PID instead of waiting

That's it. Simple by design — the complexity lives in what you ask it to do.

## Running Commands

Most of the time, you just ask Amplifier to do something and it reaches for bash when appropriate:

> Run the test suite

```
[Tool: bash] pytest tests/ --verbose
===== test session starts =====
tests/test_auth.py::test_login PASSED
tests/test_auth.py::test_logout PASSED
tests/test_api.py::test_create_user PASSED
===== 3 passed in 1.42s =====
```

> Install the requests library

```
[Tool: bash] pip install requests
Successfully installed requests-2.31.0
```

> What branch am I on?

```
[Tool: bash] git branch --show-current
feature/auth-redesign
```

Notice that you're speaking naturally — Amplifier decides when bash is the right tool and formulates the command for you.

## When to Use Bash (and When Not To)

This is the most important thing to understand about the Bash tool. Amplifier has specialized tools that do many things better than raw shell commands. Here's the decision matrix:

| Task | Best tool | Why not bash? |
|------|-----------|---------------|
| Run tests, builds | **Bash** | This is bash's sweet spot |
| Install packages | **Bash** | pip, npm, cargo — all bash territory |
| Git status, diff, commit | **Bash** | Simple git ops work great here |
| Read a file | **read_file** | Structured output with line numbers |
| Search file contents | **grep** | Regex-aware, respects .gitignore, paginated |
| Find files by name | **glob** | Faster, handles exclusions automatically |
| Fetch a web page | **web_fetch** | Handles encoding, size limits, save-to-file |
| Complex git workflows | **git-ops agent** | Multi-step reasoning about branches and merges |

The rule of thumb: if Amplifier has a dedicated tool for the job, use it. Reach for bash when you need to run an actual program — a test runner, a build system, a package manager, a container tool.

## Timeouts and Long-Running Commands

By default, commands time out after 30 seconds. That's fine for `git status` or `pip install`, but a full test suite or a production build needs more room:

> Run the full integration test suite

```
[Tool: bash] pytest tests/integration/ --verbose
  timeout: 300
===== 47 passed, 2 skipped in 4m 12s =====
```

Here's the general guidance:

- **30s (default)** — quick commands: git status, ls, pip install a single package
- **120–300s** — builds and test suites: pytest, cargo build, npm run build
- **run_in_background** — anything that runs indefinitely: dev servers, file watchers

## Background Processes

Some commands aren't meant to finish — dev servers, file watchers, database processes. Use `run_in_background` to start them without blocking your session:

> Start the Next.js dev server

```
[Tool: bash] npm run dev
  run_in_background: true
Started background process with PID 48291
```

The process runs independently and Amplifier returns immediately. You can check on it later or stop it:

> Is the dev server still running?

```
[Tool: bash] ps aux | grep "npm run dev"
user  48291  0.5  1.2  npm run dev
```

> Stop the dev server

```
[Tool: bash] kill 48291
```

Background mode is perfect for:
- Development servers (`npm run dev`, `python manage.py runserver`)
- File watchers (`cargo watch`, `nodemon`)
- Local databases (`redis-server`, `mongod`)

Don't use it for commands where you need the output — test results, build logs, or anything with an exit code you care about.

## Output Truncation

When a command produces a lot of output, Amplifier automatically truncates it to prevent flooding the conversation context. You'll see something like this:

```
[Tool: bash] find . -name "*.py" -type f
./src/main.py
./src/config.py
./src/utils/helpers.py
[...truncated...]
./tests/test_integration.py
./tests/conftest.py

[Truncated: showing 2000 of 5234 lines, 45KB of 180KB total]
```

The truncation shows the first lines, a `[...truncated...]` marker, and the last lines, plus byte counts so you know what you're missing.

**Warning:** If the output contains JSON, XML, or other structured data, truncation can break it mid-structure. The workaround is to redirect to a file:

> Get the full API response

```
[Tool: bash] curl https://api.example.com/data > response.json
[Tool: read_file] response.json (lines 1-50)
```

Redirect first, then read the file in controlled chunks.

## Safety Guardrails

The Bash tool blocks destructive commands that could damage your system — things like `rm -rf /` or `sudo rm -rf`. It also doesn't support interactive commands that wait for user input. Editors like vim, interactive REPLs, and commands without `-y` flags will hang and eventually time out.

The fix is usually straightforward — add non-interactive flags:

> Initialize a new npm project

```
[Tool: bash] npm init -y
Wrote to /home/user/project/package.json
```

The `-y` flag accepts all defaults without prompting.

## Practical Patterns

### Chaining Commands

Use `&&` to chain dependent commands — if any step fails, the rest don't run:

> Set up a new Python virtual environment and install dependencies

```
[Tool: bash] python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
```

Avoid `;` for chaining — it runs every command regardless of failures, which can leave you in a messy state.

### Quoting Paths

Always quote paths that might contain spaces:

```bash
# Works
cd "/home/user/My Projects/webapp"

# Breaks — shell sees three arguments
cd /home/user/My Projects/webapp
```

### Absolute vs. Relative Paths

Prefer absolute paths when you're unsure of the working directory. Each bash invocation maintains its directory context within a session, but absolute paths remove all ambiguity:

```bash
# Unambiguous
pytest /home/user/project/tests/

# Depends on where you are
pytest tests/
```

### Redirecting Large Output

When a command generates more output than you need in context, redirect and then inspect:

```
> Show me just the failing tests from the full suite

[Tool: bash] pytest tests/ --tb=short 2>&1 | grep -E "FAILED|ERROR"
tests/test_api.py::test_timeout FAILED
tests/test_api.py::test_retry ERROR
```

Piping through grep keeps the output focused and avoids truncation.

## Tips

- **Increase timeout for builds.** A 30-second default is too short for `cargo build --release` or `npm run build`. Bump to 120–300 seconds.
- **Redirect structured output.** If a command returns JSON or XML, save to a file first: `command > output.json`. Then use `read_file` to inspect it.
- **Check before you chain.** Run `which pytest` or `python --version` before a long chain — catching a missing tool early saves time.
- **Use background for servers, not builds.** You want to see build output. You don't want to wait for a server to "finish."
- **Pipe to filter.** Combine bash with `grep`, `head`, `tail`, or `wc` to keep output manageable within a single command.

## Next Steps

- Learn about the [Filesystem Tool](./filesystem.md) for reading and editing files — the preferred alternative to `cat` and `sed` via bash
- Explore [Search Tools](./search.md) to see why `grep` and `glob` beat `find | xargs grep`
- See [Task (Sub-Agents)](./task.md) for delegating complex work to specialized agents
