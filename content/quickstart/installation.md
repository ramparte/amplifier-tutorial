---
id: installation
type: quickstart
title: "Installation"
---

# Installation

Get Amplifier running in under 2 minutes.

## Prerequisites

- **Python 3.11+** (3.12 recommended)
- **macOS or Linux** (Windows users: use WSL2)
- **API key** for at least one AI provider (Anthropic, OpenAI, or free local with Ollama)

## Step 1: Install UV

UV is a fast Python package manager. Amplifier uses it for installation.

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify installation
uv --version
```

!!! tip "Why UV?"
    UV is 10-100x faster than pip and handles virtual environments automatically. Amplifier uses it for both installation and dependency management.

## Step 2: Install Amplifier

```bash
# Install from GitHub
uv tool install git+https://github.com/microsoft/amplifier

# Verify installation
amplifier --version
```

This installs the `amplifier` command globally.

## Step 3: Configure Your Provider

Run the initialization wizard:

```bash
amplifier init
```

You'll be prompted to choose a provider and enter your API key:

```
? Select AI provider:
  > anthropic (Claude - recommended)
    openai (GPT models)
    azure-openai (Enterprise Azure)
    ollama (Local models - free)
    
? Enter your Anthropic API key: sk-ant-...
```

### Provider Options

=== "Anthropic (Recommended)"

    Best tested, most capable for coding tasks.
    
    ```bash
    # Get API key from https://console.anthropic.com
    amplifier provider add anthropic --api-key sk-ant-...
    amplifier provider use anthropic
    ```

=== "OpenAI"

    GPT models including GPT-4.
    
    ```bash
    # Get API key from https://platform.openai.com
    amplifier provider add openai --api-key sk-...
    amplifier provider use openai
    ```

=== "Ollama (Free, Local)"

    Run models locally - no API key needed.
    
    ```bash
    # First, install Ollama: https://ollama.ai
    ollama pull llama3.2
    
    # Then configure Amplifier
    amplifier provider add ollama
    amplifier provider use ollama
    ```

=== "Azure OpenAI"

    Enterprise deployment with managed identity.
    
    ```bash
    amplifier provider add azure-openai \
      --endpoint https://your-resource.openai.azure.com \
      --deployment your-deployment-name
    ```

## Step 4: Verify Installation

```bash
# Start an interactive session
amplifier

# Or run a single command
amplifier run "Hello! What tools do you have?"
```

You should see Amplifier respond with a list of available tools.

## Configuration Files

Amplifier stores configuration at `~/.amplifier/`:

```
~/.amplifier/
├── settings.yaml     # Provider config, preferences
├── sessions/         # Session logs and history
└── bundles/          # Downloaded bundles
```

### View Your Configuration

```bash
# See current settings
amplifier config show

# See available providers
amplifier provider list
```

## Troubleshooting

### "Command not found: amplifier"

UV installs tools to `~/.local/bin`. Add it to your PATH:

```bash
# Add to your shell config (.bashrc, .zshrc, etc.)
export PATH="$HOME/.local/bin:$PATH"

# Reload your shell
source ~/.bashrc  # or ~/.zshrc
```

### "API key invalid"

Double-check your key has no extra whitespace:

```bash
# Re-add the provider with the correct key
amplifier provider add anthropic --api-key "sk-ant-your-key-here"
```

### Windows Issues

Amplifier has limited Windows support. Use WSL2:

```bash
# In PowerShell (as admin)
wsl --install

# Then follow Linux instructions inside WSL
```

### Network/Proxy Issues

Set environment variables for proxy:

```bash
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
```

## Next

Installation complete! Let's have your first conversation.

→ [Your First Conversation](first-conversation.md)
