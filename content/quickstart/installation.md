---
id: installation
type: quickstart
title: "Installation"
---

# Installation

This guide walks you through installing Amplifier, a powerful AI development platform that extends Claude's capabilities with specialized tools, agents, and workflows.

## Prerequisites

Before installing Amplifier, ensure your system meets these requirements:

### Required

- **Python 3.10 or higher** - Amplifier requires modern Python features
  - Check your version: `python --version` or `python3 --version`
  - Download from [python.org](https://www.python.org/downloads/) if needed

### Recommended

- **Git** - For version control and cloning repositories
- **Terminal** - Command-line access (Terminal on macOS/Linux, PowerShell or WSL on Windows)
- **Anthropic API Key** - Required for Claude access
  - Get yours at [console.anthropic.com](https://console.anthropic.com/)

## Quick Install

For most users, the fastest way to get started:

```bash
# Install UV package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install Amplifier
uvx amplifier

# Run setup wizard
amplifier init

# Verify installation
amplifier version
```

## Detailed Installation Steps

### Step 1: Install UV Package Manager

UV is a fast, reliable Python package installer and resolver. Amplifier uses UV for efficient dependency management.

**On macOS and Linux:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**On Windows (PowerShell):**

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Verify UV installation:**

```bash
uv --version
```

You should see output like `uv 0.x.x`.

**Alternative: Install UV with pip:**

```bash
pip install uv
```

### Step 2: Install Amplifier

Once UV is installed, you can install Amplifier using `uvx`, which runs packages in isolated environments:

```bash
uvx amplifier
```

This command will:
- Download the latest version of Amplifier
- Install all required dependencies
- Make the `amplifier` command available in your terminal

**Alternative: Install globally with pip:**

```bash
pip install amplifier
```

**Alternative: Install from source:**

```bash
git clone https://github.com/yourusername/amplifier.git
cd amplifier
pip install -e .
```

### Step 3: Run the Setup Wizard

The setup wizard helps you configure Amplifier for first use:

```bash
amplifier init
```

The wizard will guide you through:

1. **API Key Configuration** - Enter your Anthropic API key
2. **Default Model Selection** - Choose your preferred Claude model
3. **Workspace Setup** - Configure your default workspace directory
4. **Bundle Installation** - Install recommended bundles (optional)

**Example wizard session:**

```
Welcome to Amplifier Setup!

? Enter your Anthropic API key: sk-ant-***************
✓ API key validated successfully

? Select default model:
  > claude-3-5-sonnet-20241022 (recommended)
    claude-3-opus-20240229
    claude-3-haiku-20240307

? Workspace directory: ~/amplifier-workspace
✓ Workspace created at /home/user/amplifier-workspace

? Install recommended bundles? (Y/n): Y
✓ Installing foundation bundle...
✓ Installing python-dev bundle...
✓ Installing recipes bundle...

Setup complete! Run 'amplifier chat' to start.
```

### Step 4: Verify Installation

Confirm everything is working correctly:

```bash
# Check version
amplifier version

# View available commands
amplifier --help

# Test connection
amplifier test-connection

# List installed bundles
amplifier bundle list
```

**Expected output for `amplifier version`:**

```
Amplifier v1.0.0
Python 3.11.5
UV 0.4.0
Claude API: Connected
```

## Platform-Specific Notes

### macOS

**Homebrew installation (coming soon):**

```bash
brew install amplifier
```

**Shell configuration:**

Add to your `~/.zshrc` or `~/.bash_profile`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Linux

**System dependencies:**

On Debian/Ubuntu:

```bash
sudo apt update
sudo apt install python3-pip git curl
```

On Fedora/RHEL:

```bash
sudo dnf install python3-pip git curl
```

**Shell configuration:**

Add to your `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Windows

**Recommended: Use WSL2 (Windows Subsystem for Linux)**

1. Install WSL2: `wsl --install`
2. Open Ubuntu terminal
3. Follow Linux installation steps

**Native Windows installation:**

- Use PowerShell (not CMD)
- Install Python from Microsoft Store or python.org
- Ensure Python is in your PATH

**Common Windows issue:**

If you see "command not found" errors, add Python to PATH:

```powershell
[Environment]::SetEnvironmentVariable("Path", "$env:Path;$env:LOCALAPPDATA\Programs\Python\Python311\Scripts", "User")
```

## Configuration

### API Key Setup

Amplifier needs your Anthropic API key to communicate with Claude. You can configure it in several ways:

**Option 1: Setup wizard (recommended)**

```bash
amplifier init
```

**Option 2: Environment variable**

```bash
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
```

Add to your `~/.bashrc` or `~/.zshrc` to persist.

**Option 3: Configuration file**

Create `~/.config/amplifier/config.yaml`:

```yaml
api_key: sk-ant-your-key-here
default_model: claude-3-5-sonnet-20241022
workspace: ~/amplifier-workspace
```

**Option 4: Command-line flag**

```bash
amplifier chat --api-key sk-ant-your-key-here
```

### Workspace Configuration

Your workspace is where Amplifier stores projects, skills, and bundles:

```bash
# Set default workspace
amplifier config set workspace ~/my-workspace

# View current configuration
amplifier config show
```

## Common Issues

### Issue: "command not found: amplifier"

**Solution:**

Ensure the installation directory is in your PATH:

```bash
echo $PATH | grep -q "$HOME/.local/bin" || export PATH="$HOME/.local/bin:$PATH"
```

Make it permanent by adding to your shell config file.

### Issue: "API key not configured"

**Solution:**

Run the setup wizard or set the environment variable:

```bash
amplifier init
# or
export ANTHROPIC_API_KEY="your-key-here"
```

### Issue: "Python version too old"

**Solution:**

Upgrade Python to 3.10 or higher:

```bash
# macOS with Homebrew
brew install python@3.11

# Ubuntu/Debian
sudo apt install python3.11

# Windows
# Download from python.org
```

### Issue: "Permission denied" during installation

**Solution:**

Don't use `sudo` with pip. Install for your user:

```bash
pip install --user amplifier
```

### Issue: UV installation fails

**Solution:**

Install via pip instead:

```bash
pip install uv
```

Then retry Amplifier installation.

### Issue: SSL certificate verification failed

**Solution:**

Update your system's CA certificates:

```bash
# macOS
brew install ca-certificates

# Ubuntu/Debian
sudo apt install ca-certificates
```

## Verifying Your Installation

Run this comprehensive check:

```bash
# Check all components
amplifier doctor

# Test API connection
amplifier test-connection

# List available tools
amplifier tools list

# View system info
amplifier info
```

## Next Steps

Now that Amplifier is installed, you're ready to start building:

1. **[First Conversation](./first-conversation.md)** - Have your first chat with Amplifier
2. **[Key Commands](./key-commands.md)** - Learn essential commands
3. **[First Bundle](./first-bundle.md)** - Create your first custom bundle
4. **[Core Concepts](../concepts/index.md)** - Understand Amplifier's architecture

## Getting Help

If you encounter issues not covered here:

- **Documentation**: [amplifier.dev/docs](https://amplifier.dev/docs)
- **GitHub Issues**: [github.com/yourusername/amplifier/issues](https://github.com/yourusername/amplifier/issues)
- **Community Discord**: [discord.gg/amplifier](https://discord.gg/amplifier)
- **Email Support**: support@amplifier.dev

## Updating Amplifier

To update to the latest version:

```bash
# With uvx
uvx --refresh amplifier

# With pip
pip install --upgrade amplifier

# View changelog
amplifier changelog
```

## Uninstalling

If you need to remove Amplifier:

```bash
# With pip
pip uninstall amplifier

# Remove configuration
rm -rf ~/.config/amplifier

# Remove workspace (optional)
rm -rf ~/amplifier-workspace
```

---

**Ready to start?** Run `amplifier chat` and say hello to your AI development assistant!
