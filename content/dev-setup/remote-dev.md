---
id: remote-dev
type: dev-setup
title: "Remote Development"
---

# Remote Development

Run Amplifier on remote machines, containers, and cloud environments.

## Overview

Remote development options:

| Method | Best For |
|--------|----------|
| SSH | Remote servers, VMs |
| VS Code Remote | Seamless IDE experience |
| Docker | Isolated environments |
| GitHub Codespaces | Cloud development |
| WSL | Windows + Linux |

## SSH

Run Amplifier on a remote server.

### Setup

```bash
# SSH to your server
ssh user@remote-server

# Install Amplifier
curl -LsSf https://astral.sh/uv/install.sh | sh
uv tool install amplifier

# Configure provider (on remote)
export ANTHROPIC_API_KEY="your-key"

# Run
amp
```

### Port Forwarding

For web UIs or local services:

```bash
ssh -L 8080:localhost:8080 user@remote-server
```

### Persistent Sessions

Use tmux to keep sessions alive:

```bash
# Start tmux
ssh user@remote-server
tmux new -s amplifier

# Run Amplifier
amp

# Detach: Ctrl+b, then d
# Reconnect later
ssh user@remote-server
tmux attach -t amplifier
```

## VS Code Remote

Use VS Code's Remote extensions for seamless experience.

### Remote - SSH

```bash
# Install extension
code --install-extension ms-vscode-remote.remote-ssh

# Connect
# Ctrl+Shift+P → "Remote-SSH: Connect to Host"

# Open terminal in VS Code
# Run Amplifier normally
amp
```

### Remote - Containers

```bash
# Install extension
code --install-extension ms-vscode-remote.remote-containers

# Create devcontainer config
mkdir .devcontainer
```

`.devcontainer/devcontainer.json`:
```json
{
  "name": "Amplifier Dev",
  "image": "python:3.12",
  "postCreateCommand": "curl -LsSf https://astral.sh/uv/install.sh | sh && uv tool install amplifier",
  "remoteEnv": {
    "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}"
  }
}
```

## Docker

Run Amplifier in a container.

### Basic Usage

```bash
# Run interactive container
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  python:3.12 bash

# Inside container
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.cargo/env
uv tool install amplifier
amp
```

### Dockerfile

```dockerfile
FROM python:3.12-slim

# Install uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# Add uv to PATH
ENV PATH="/root/.cargo/bin:$PATH"

# Install Amplifier
RUN uv tool install amplifier

WORKDIR /workspace

CMD ["amp"]
```

Build and run:
```bash
docker build -t amplifier-dev .
docker run -it --rm \
  -v $(pwd):/workspace \
  -e ANTHROPIC_API_KEY \
  amplifier-dev
```

### Docker Compose

```yaml
version: '3.8'
services:
  amplifier:
    build: .
    volumes:
      - .:/workspace
    environment:
      - ANTHROPIC_API_KEY
    stdin_open: true
    tty: true
```

```bash
docker compose run amplifier
```

## GitHub Codespaces

Cloud-hosted development environment.

### Setup

1. Create `.devcontainer/devcontainer.json` in your repo:

```json
{
  "name": "Amplifier Workspace",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "postCreateCommand": "curl -LsSf https://astral.sh/uv/install.sh | sh && source ~/.cargo/env && uv tool install amplifier",
  "secrets": {
    "ANTHROPIC_API_KEY": {
      "description": "API key for Anthropic Claude"
    }
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python"
      ]
    }
  }
}
```

2. Set secret in GitHub repo settings:
   - Settings → Secrets → Codespaces
   - Add `ANTHROPIC_API_KEY`

3. Create Codespace:
   - Code → Codespaces → Create on main

4. In terminal:
```bash
amp
```

## WSL (Windows Subsystem for Linux)

Run Amplifier in Linux on Windows.

### Setup

```powershell
# Install WSL (PowerShell Admin)
wsl --install -d Ubuntu

# Start WSL
wsl
```

In WSL:
```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.cargo/env

# Install Amplifier
uv tool install amplifier

# Configure
export ANTHROPIC_API_KEY="your-key"
echo 'export ANTHROPIC_API_KEY="your-key"' >> ~/.bashrc

# Run
amp
```

### Access Windows Files

```bash
# Windows drives are mounted at /mnt
cd /mnt/c/Users/YourName/Projects

# Run Amplifier on Windows project
amp
```

### VS Code + WSL

```bash
# From WSL
cd /mnt/c/YourProject
code .  # Opens VS Code connected to WSL
```

## Environment Variables

Securely handle API keys:

### Local .env File

```bash
# .env (gitignored!)
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
```

```bash
# Load before running
source .env
amp
```

### direnv

Auto-load environment per directory:

```bash
# Install
brew install direnv

# Setup
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc

# Create .envrc
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' > .envrc
direnv allow

# Now auto-loads when you cd into directory
```

## Try It Yourself

### Exercise 1: Docker

```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  -e ANTHROPIC_API_KEY \
  python:3.12 bash -c "
    curl -LsSf https://astral.sh/uv/install.sh | sh &&
    source ~/.cargo/env &&
    uv tool install amplifier &&
    amp
  "
```

### Exercise 2: tmux

```bash
# Start tmux
tmux new -s dev

# Run Amplifier
amp

# Detach: Ctrl+b, d
# Reattach: tmux attach -t dev
```

## Next

Learn debugging techniques:

→ [Debugging](debugging.md)
