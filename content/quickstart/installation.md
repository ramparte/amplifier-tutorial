---
id: installation
type: quickstart
title: "Installation"
---

# Installation

Let's get Amplifier running on your machine. By the end of this page you'll have the CLI installed, an API key configured, and a working session you can talk to.

## Prerequisites

You need three things before you start:

**Python 3.11 or newer.** Check what you have:

```bash
python3 --version
```

If you see `Python 3.11.x` or higher, you're good. If not, grab the latest from [python.org](https://www.python.org/downloads/) or use your system package manager.

**uv — the fast Python package manager.** This is the recommended way to install Amplifier. If you already have it, skip ahead. If not, it takes one command (we'll cover this in Step 1).

**An LLM API key.** Amplifier supports multiple providers, but Anthropic Claude is the most tested and recommended. Get a key at [console.anthropic.com](https://console.anthropic.com/settings/keys) if you don't have one.

## Step 1: Install uv

uv handles Amplifier's installation and dependency isolation so you don't pollute your global Python.

**macOS / Linux / WSL:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows PowerShell:**

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verify it worked:

```bash
uv --version
```

> uv 0.6.x (or similar)

If that fails, you can also install uv through pip (`pip install uv`), but the curl method above is preferred.

## Step 2: Install Amplifier

One command:

```bash
uv tool install git+https://github.com/microsoft/amplifier
```

This pulls Amplifier from the main branch, installs it into an isolated environment, and puts the `amplifier` command on your PATH. It takes about 30 seconds on a typical connection.

**Alternative — pip install:**

If you prefer pip over uv:

```bash
pip install git+https://github.com/microsoft/amplifier
```

This works fine, though uv's isolation is cleaner for CLI tools.

## Step 3: Run the Setup Wizard

The first time you launch Amplifier, run the init wizard to configure your provider and API key:

```bash
amplifier init
```

The wizard walks you through three choices:

```
Welcome to Amplifier!

Step 1: Provider
Which provider? [1] Anthropic [2] OpenAI [3] Azure OpenAI [4] Ollama: 1

API key: ••••••••
  Get one: https://console.anthropic.com/settings/keys
✓ Saved to ~/.amplifier/keys.env

Model? [1] claude-sonnet-4-5 [2] claude-opus-4-6 [3] custom: 1
✓ Using claude-sonnet-4-5

Step 2: Bundle
Which bundle? [1] foundation [2] dev [3] full: 1
✓ Using 'foundation' bundle

Ready! Try: amplifier run "Hello world"
```

That's it. Your API key is saved to `~/.amplifier/keys.env` and your provider/model/bundle choices go into `~/.amplifier/settings.yaml`. You can change any of these later.

**Tip — pre-set your key with an environment variable:** If you export `ANTHROPIC_API_KEY` in your shell before running `amplifier init`, the wizard detects it automatically and you just press Enter to confirm.

```bash
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
amplifier init
```

## Step 4: Say Hello

Try a single-shot command:

```bash
amplifier run "Hello, Amplifier!"
```

> Hello! I'm ready to help. What would you like to work on?

Now try interactive chat mode — just run `amplifier` with no arguments:

```bash
amplifier
```

You'll land at an interactive prompt. Type a message, get a response, and keep going. Type `exit` or press `Ctrl+C` when you're done.

> /help

This shows the slash commands available in chat: `/tools`, `/agents`, `/status`, `/config`, and more. We'll cover these in [Your First Conversation](./first-conversation.md).

## Verify It Works

Run through this quick checklist to confirm everything is connected:

```bash
# Is the CLI installed?
amplifier --help

# Can it reach your provider?
amplifier run "What model are you?"

# What bundle is active?
amplifier bundle current

# What's loaded?
amplifier module current
```

If `amplifier run` gives you a coherent response, you're fully operational. The model will identify itself (e.g., "I'm Claude, made by Anthropic") which confirms your API key and provider are working.

## Keeping Amplifier Updated

Amplifier moves fast. Update to the latest version with:

```bash
amplifier update
```

This pulls the newest code and updates your installation in place. Run it periodically — especially if you hit a bug that might already be fixed.

If an update leaves things broken, see [Clean Reinstall](#clean-reinstall) in Common Issues below.

## Choosing a Bundle (Optional)

Bundles define what tools and agents are available in your session. The default `foundation` bundle covers most needs, but you can switch:

```bash
# See what's available
amplifier bundle list

# Switch to a different bundle
amplifier bundle use dev

# Use a specific bundle for one command without changing your default
amplifier run --bundle recipes "Plan a migration strategy"
```

You can also install additional bundles from Git:

```bash
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
amplifier bundle use recipes
```

Don't worry about bundles right now — the default is fine for getting started. We cover bundles properly in [Your First Bundle](./first-bundle.md).

## Shell Completion (Optional)

Tab completion makes the CLI much faster to use:

```bash
amplifier --install-completion
```

This auto-detects your shell (bash, zsh, or fish) and adds the right completion config. Open a new terminal or source your shell config to activate it.

## Platform Notes

**macOS** — Amplifier's primary development platform. Everything works out of the box. Make sure `~/.local/bin` is on your PATH (uv handles this during install).

**Linux** — Works great. Same PATH note as macOS. If you're on a minimal server image, make sure `curl` and `git` are installed (`sudo apt install curl git` on Debian/Ubuntu).

**Windows** — Use WSL2. Native Windows shells have unresolved issues. Install WSL if you haven't:

```bash
wsl --install
```

Then open the Ubuntu terminal and follow the macOS/Linux steps above. Everything works normally inside WSL.

## Common Issues

### Python version too old

```
Error: Python 3.11+ required
```

Amplifier needs Python 3.11 or newer. Check with `python3 --version` and upgrade if needed:

```bash
# macOS
brew install python@3.12

# Ubuntu/Debian
sudo apt install python3.12

# Or download from python.org
```

### uv not found

```
command not found: uv
```

Either uv isn't installed or it's not on your PATH. Try the install command from Step 1 again. If it installs but still isn't found, open a new terminal — the PATH update happens in your shell config.

### API key not set

```
Error: No API key configured
```

You haven't run `amplifier init`, or the key wasn't saved. Three ways to fix this:

1. Run `amplifier init` and enter your key when prompted
2. Export it directly: `export ANTHROPIC_API_KEY="sk-ant-your-key"`
3. Add it to `~/.amplifier/keys.env` manually

The init wizard is easiest. It validates the key before saving.

### Firewall or proxy blocking the install

If `uv tool install` hangs or fails with connection errors, your network might block GitHub or PyPI. Try:

```bash
# If behind a corporate proxy, set these first
export HTTP_PROXY="http://your-proxy:port"
export HTTPS_PROXY="http://your-proxy:port"

# Then retry
uv tool install git+https://github.com/microsoft/amplifier
```

If Git access is blocked entirely, ask your IT team about GitHub access or try from a different network.

### Clean Reinstall

If things get into a bad state — stale cache, broken update, weird module conflicts — a clean reinstall fixes it reliably:

```bash
# 1. Remove Amplifier data (preserves session history if you skip the projects/ dir)
rm -rf ~/.amplifier

# 2. Clear uv cache and uninstall
uv cache clean
uv tool uninstall amplifier

# 3. Reinstall fresh
uv tool install git+https://github.com/microsoft/amplifier

# 4. Reconfigure
amplifier init
```

This is the nuclear option but it works every time. Your API key will need to be re-entered during `amplifier init`.

## Next Steps

You've got Amplifier installed and talking. Here's where to go from here:

1. **[Your First Conversation](./first-conversation.md)** — Learn how to interact with Amplifier effectively
2. **[Key Commands](./key-commands.md)** — Essential commands for daily use
3. **[Your First Bundle](./first-bundle.md)** — Customize what tools and agents are available
4. **[Core Concepts](../concepts/index.md)** — Understand the architecture underneath
