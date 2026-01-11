---
id: installation
type: quickstart
title: "Installation"
---

# Installation

This guide walks you through installing Amplifier on your system. The process takes about 5 minutes and requires minimal setup.

## Prerequisites

Before installing Amplifier, ensure your system meets these requirements:

### Python Version

Amplifier requires **Python 3.10 or higher**. Check your Python version:

```bash
python --version
```

If you need to install or upgrade Python:

- **macOS**: `brew install python@3.12`
- **Ubuntu/Debian**: `sudo apt install python3.12`
- **Windows**: Download from [python.org](https://www.python.org/downloads/)

### Operating System Support

Amplifier runs on:

- macOS 12 (Monterey) or later
- Linux (Ubuntu 20.04+, Debian 11+, Fedora 36+)
- Windows 10/11 with WSL2

### Additional Requirements

- Terminal access (bash, zsh, or PowerShell with WSL)
- Internet connection for package downloads
- At least 500MB of free disk space

---

## Step 1: Install UV

UV is the recommended package manager for Amplifier. It's fast, reliable, and handles dependencies automatically.

### macOS and Linux

Run the installation script:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

After installation, restart your terminal or run:

```bash
source ~/.bashrc  # or ~/.zshrc for zsh
```

### Windows (WSL)

Open your WSL terminal and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Alternative: Install via Homebrew

If you prefer Homebrew on macOS:

```bash
brew install uv
```

### Verify UV Installation

Confirm UV is installed correctly:

```bash
uv --version
```

You should see output like:

```
uv 0.5.x
```

---

## Step 2: Install Amplifier

With UV installed, getting Amplifier is a single command:

```bash
uvx amplifier
```

This command:

1. Downloads the latest Amplifier release
2. Creates an isolated environment
3. Installs all dependencies
4. Makes `amplifier` available globally

### What Happens During Installation

UV creates a tool environment at `~/.local/share/uv/tools/amplifier/` containing:

- The Amplifier CLI application
- All required Python packages
- Configuration templates

### Installation Location

By default, the `amplifier` command is installed to:

- **macOS/Linux**: `~/.local/bin/amplifier`
- **Windows WSL**: `~/.local/bin/amplifier`

Ensure this directory is in your PATH. UV typically handles this automatically.

---

## Step 3: Setup Wizard

Run the interactive setup wizard to configure Amplifier:

```bash
amplifier init
```

The wizard guides you through:

### Provider Configuration

Select your AI provider(s):

```
? Select AI providers to configure:
  [x] Anthropic (Claude)
  [ ] OpenAI (GPT-4)
  [ ] Azure OpenAI
  [ ] Google (Gemini)
```

### API Key Setup

Enter your API key when prompted:

```
? Enter your Anthropic API key: sk-ant-...
```

Your keys are stored securely in `~/.amplifier/settings.yaml`.

### Default Model Selection

Choose your preferred model:

```
? Select default model:
  > claude-sonnet-4-20250514
    claude-3-5-haiku-20241022
    claude-3-opus-20240229
```

### Configuration File

The wizard creates `~/.amplifier/settings.yaml`:

```yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
    default_model: claude-sonnet-4-20250514

defaults:
  provider: anthropic
  temperature: 0.7
```

### Skip Interactive Mode

For automated setups, use environment variables:

```bash
export ANTHROPIC_API_KEY="your-key-here"
amplifier init --non-interactive
```

---

## Step 4: Verify Installation

Confirm everything is working:

```bash
amplifier version
```

Expected output:

```
Amplifier v0.x.x
Python: 3.12.x
UV: 0.5.x
Config: ~/.amplifier/settings.yaml
```

### Quick Test

Start an interactive session:

```bash
amplifier
```

You should see the Amplifier prompt:

```
Amplifier v0.x.x | claude-sonnet-4-20250514
Type /help for commands, /quit to exit

>
```

Type a simple query to confirm everything works:

```
> Hello, can you hear me?
```

Press `/quit` to exit.

---

## Common Issues

### UV Not Found

**Symptom**: `command not found: uv`

**Solution**: Add UV to your PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Add this line to your `~/.bashrc` or `~/.zshrc` for persistence.

### Python Version Too Old

**Symptom**: `Python 3.10+ required, found 3.8.x`

**Solution**: Install a newer Python version or use pyenv:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Install Python 3.12
pyenv install 3.12
pyenv global 3.12
```

### API Key Invalid

**Symptom**: `Authentication failed: Invalid API key`

**Solution**: Verify your API key is correct:

```bash
# Check the stored key
cat ~/.amplifier/settings.yaml | grep api_key

# Re-run setup
amplifier init
```

### Permission Denied

**Symptom**: `Permission denied: ~/.local/bin/amplifier`

**Solution**: Fix permissions:

```bash
chmod +x ~/.local/bin/amplifier
```

### SSL Certificate Errors

**Symptom**: `SSL: CERTIFICATE_VERIFY_FAILED`

**Solution**: Update certificates:

```bash
# macOS
/Applications/Python\ 3.12/Install\ Certificates.command

# Linux
sudo apt install ca-certificates
sudo update-ca-certificates
```

### Proxy Configuration

If you're behind a corporate proxy:

```bash
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
uvx amplifier
```

### WSL-Specific Issues

**Symptom**: Slow performance or networking issues in WSL

**Solution**: Ensure WSL2 is being used:

```powershell
# In PowerShell
wsl --set-default-version 2
```

---

## Updating Amplifier

Keep Amplifier current with:

```bash
uvx amplifier@latest
```

Or specify a version:

```bash
uvx amplifier@0.5.0
```

### Check for Updates

See if a newer version is available:

```bash
amplifier version --check-updates
```

---

## Uninstalling

To remove Amplifier:

```bash
uv tool uninstall amplifier
```

To also remove configuration:

```bash
rm -rf ~/.amplifier
```

---

## Next Steps

Now that Amplifier is installed, continue with:

- **[First Session](first-session.md)** - Start your first AI-assisted coding session
- **[Configuration Guide](configuration.md)** - Customize Amplifier settings
- **[Provider Setup](providers.md)** - Configure additional AI providers
- **[Bundles Overview](bundles.md)** - Extend functionality with bundles

### Quick Commands Reference

| Command | Description |
|---------|-------------|
| `amplifier` | Start interactive session |
| `amplifier init` | Run setup wizard |
| `amplifier version` | Show version info |
| `amplifier --help` | Show all commands |

---

## Getting Help

If you encounter issues not covered here:

- Check the [FAQ](../reference/faq.md)
- Search [GitHub Issues](https://github.com/microsoft/amplifier/issues)
- Join the community discussions

Welcome to Amplifier!
