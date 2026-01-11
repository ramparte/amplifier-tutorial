---
id: tool-bash
type: tools
title: "Bash Tool"
---

# Bash Tool

The Bash tool provides direct shell command execution, giving you access to your system's full command-line capabilities. This is your gateway to build systems, package managers, version control, and any other terminal operation. While specialized tools exist for many tasks, bash serves as the universal fallback when you need raw shell power.

## Operations

| Operation | Purpose | Example |
|-----------|---------|---------|
| Run command | Execute shell command synchronously | `pytest tests/` |
| Background | Start long-running process | `npm run dev` |
| Chain commands | Run dependent operations | `mkdir build && cd build && cmake ..` |
| Pipeline | Process output through filters | `cat log.txt \| grep ERROR` |
| Redirect | Save output to file | `npm test > results.txt 2>&1` |

## Basic Usage

### Interactive Prompt Format

Using the `>` prompt in conversation:

```
> pytest tests/unit/
```

```
> npm install express
```

```
> git status
```

### Tool Call Format

How Amplifier executes bash internally:

```
[Tool: bash]
command: pytest tests/unit/
```

```
[Tool: bash]
command: npm install express
```

### Background Process Format

For long-running processes that shouldn't block:

```
[Tool: bash]
command: npm run dev
run_in_background: true
```

## Common Commands

### Build Commands

```bash
# Python
> pip install -e .
> python setup.py build
> poetry build

# Node.js
> npm install
> npm run build
> yarn build

# Rust
> cargo build
> cargo build --release

# Go
> go build ./...
> go build -o myapp main.go

# Make
> make
> make clean && make all
```

### Test Commands

```bash
# Python
> pytest
> pytest tests/unit/ -v
> pytest --cov=src tests/
> python -m unittest discover

# Node.js
> npm test
> npm run test:unit
> jest --coverage

# Rust
> cargo test
> cargo test -- --nocapture

# Go
> go test ./...
> go test -v -cover ./...
```

### Git Commands

```bash
# Status and info
> git status
> git log --oneline -10
> git diff
> git branch -a

# Staging and commits
> git add .
> git commit -m "Fix authentication bug"
> git push origin main

# Branching
> git checkout -b feature/new-login
> git merge develop
> git rebase main
```

### Package Management

```bash
# Python (pip)
> pip install requests
> pip install -r requirements.txt
> pip freeze > requirements.txt

# Python (poetry)
> poetry add pandas
> poetry install
> poetry update

# Node.js (npm)
> npm install lodash
> npm install --save-dev jest
> npm update

# Node.js (yarn)
> yarn add axios
> yarn add -D typescript

# System (apt)
> sudo apt update
> sudo apt install jq
```

### Container Operations

```bash
# Docker
> docker build -t myapp .
> docker run -p 8080:8080 myapp
> docker-compose up -d
> docker ps
> docker logs container_name

# Kubernetes
> kubectl get pods
> kubectl apply -f deployment.yaml
> kubectl logs pod-name
```

### GitHub CLI

```bash
# Pull requests
> gh pr create --title "Add feature" --body "Description"
> gh pr list
> gh pr checkout 123
> gh pr merge 123

# Issues
> gh issue create --title "Bug report"
> gh issue list --state open
> gh issue close 42
```

## Background Processes

Use `run_in_background: true` for processes that run continuously:

### Development Servers

```
[Tool: bash]
command: npm run dev
run_in_background: true
```

```
[Tool: bash]
command: python -m http.server 8000
run_in_background: true
```

```
[Tool: bash]
command: cargo watch -x run
run_in_background: true
```

### File Watchers

```
[Tool: bash]
command: npm run watch
run_in_background: true
```

### What Happens with Background Processes

1. The command starts in a separate process
2. Amplifier returns immediately with the process ID (PID)
3. The process continues running independently
4. Output is not captured in the response

### When to Use Background Mode

| Scenario | Use Background? |
|----------|-----------------|
| Dev server (`npm run dev`) | Yes |
| File watcher | Yes |
| Running tests | No |
| Building project | No |
| Git operations | No |
| Installing packages | No |

## Safety Features

### Blocked Commands

Dangerous commands are automatically blocked to prevent accidents:

```bash
# These will be rejected:
> rm -rf /
> sudo rm -rf /*
> dd if=/dev/zero of=/dev/sda
> mkfs.ext4 /dev/sda1
```

### Interactive Commands Not Supported

Commands requiring user input will fail:

```bash
# These won't work:
> vim file.txt          # Requires interactive editor
> ssh user@host         # May require password input
> mysql -u root -p      # Prompts for password
> python                # Interactive REPL
```

### Output Truncation

Long outputs are automatically truncated to prevent context overflow:

- First portion of output is shown
- `[...truncated...]` marker appears
- Final portion of output is shown
- Byte counts indicate total size

**Warning:** Truncation may break JSON, XML, or other structured output. For large structured data:

```bash
# Redirect to file instead
> npm test --json > test-results.json

# Then read portions as needed using read_file
```

### Timeouts

Commands have execution time limits. Long-running commands should use background mode or be broken into smaller operations.

## Best Practices

### 1. Use Absolute Paths

Maintain clear working directory context:

```bash
# Preferred
> pytest /home/user/project/tests/

# Instead of
> cd /home/user/project && pytest tests/
```

### 2. Quote Paths with Spaces

```bash
# Correct
> cd "/path/with spaces/project"

# Will fail
> cd /path/with spaces/project
```

### 3. Chain Dependent Commands

Use `&&` to run commands only if previous succeeds:

```bash
> mkdir -p build && cd build && cmake .. && make
```

### 4. Prefer Specialized Tools

Use bash as a fallback, not first choice:

| Task | Use Instead |
|------|-------------|
| Read file | `read_file` tool |
| Edit file | `edit_file` tool |
| Find files | `glob` tool |
| Search content | `grep` tool |

### 5. Capture Important Output

For output you need to process later:

```bash
> command > output.txt 2>&1
```

### 6. Check Exit Status

Chain with `&&` to stop on failure:

```bash
> npm install && npm test && npm run build
```

## Try It Yourself

### Exercise 1: Basic Commands

Try running these commands:

```
> echo "Hello from bash"
> pwd
> ls -la
> date
```

### Exercise 2: Project Setup

Initialize a new project:

```
> mkdir -p /tmp/demo-project
> cd /tmp/demo-project && git init
> echo "# Demo Project" > README.md
> git add . && git commit -m "Initial commit"
```

### Exercise 3: Background Server

Start a simple server in background:

```
[Tool: bash]
command: cd /tmp/demo-project && python -m http.server 8080
run_in_background: true
```

Then verify it's running:

```
> curl http://localhost:8080
```

### Exercise 4: Build and Test

Run a typical development workflow:

```
> pip install pytest
> pytest --version
```

## Errors and Troubleshooting

### Command Not Found

```
bash: npm: command not found
```

**Solution:** The command isn't in PATH. Install the required tool or use full path.

### Permission Denied

```
bash: ./script.sh: Permission denied
```

**Solution:** Make the script executable:
```bash
> chmod +x ./script.sh
```

### No Such File or Directory

```
bash: cd: /nonexistent/path: No such file or directory
```

**Solution:** Verify the path exists. Use `ls` or `glob` to find correct path.

### Interactive Command Fails

```
Error: Command requires interactive input
```

**Solution:** Use non-interactive flags:
```bash
# Instead of: git commit
> git commit -m "message"

# Instead of: npm init
> npm init -y
```

### Output Too Large

```
[Output truncated - 1.2MB total]
```

**Solution:** Redirect to file and read portions:
```bash
> long-command > output.txt
# Then use read_file with offset/limit
```

### Timeout Exceeded

```
Error: Command timed out after 60 seconds
```

**Solution:** Use background mode for long-running processes, or break into smaller operations.

### Blocked Command

```
Error: Command blocked for safety reasons
```

**Solution:** This command is restricted. Find an alternative approach or use more targeted commands.

## Summary

The Bash tool provides:

- Direct shell command execution
- Background process support for servers and watchers
- Safety features to prevent dangerous operations
- Access to build tools, package managers, and system utilities

Remember: prefer specialized tools when available, but bash is always there when you need raw command-line power.
