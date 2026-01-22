---
id: tool-bash
type: tools
title: "Bash Tool"
---

# Bash Tool

The Bash Tool provides low-level shell command execution for build operations, testing, package management, version control, and system utilities. While specialized tools exist for many operations, bash serves as a powerful fallback for commands that don't have dedicated interfaces.

## Operations

| Operation | Purpose | Example |
|-----------|---------|---------|
| Run command | Execute shell command synchronously | `pytest tests/` |
| Background | Long-running process (returns immediately) | `npm run dev` |

## Basic Usage

### Conversation Format

When you ask Amplifier to run commands, it uses the bash tool behind the scenes:

```
> Run pytest tests/test_auth.py

[Tool: bash]
Command: pytest tests/test_auth.py
Output: ===== test session starts =====
        test_auth.py::test_login PASSED
        ===== 1 passed in 0.23s =====
```

### Direct Syntax

The tool accepts these parameters:

```json
{
  "command": "pytest tests/",
  "run_in_background": false
}
```

## Common Commands

### Build and Test

Execute test runners and build tools:

```
> Run the test suite

pytest tests/ --verbose --cov
cargo test --all
npm test
make test
go test ./...
mvn test
```

### Package Management

Install and manage dependencies:

```
> Install the requests package

pip install requests
npm install lodash
cargo add serde
gem install rails
brew install ripgrep
apt-get install curl
```

### Version Control

Git operations for tracking changes:

```
> Show git status

git status
git diff src/main.py
git log --oneline -10
git branch -a
git add .
git commit -m "feat: add authentication"
git push origin main
```

### Container Operations

Work with Docker and Kubernetes:

```
> Build the docker image

docker build -t myapp:latest .
docker ps
docker logs container-name
kubectl get pods
kubectl logs pod-name
podman run -d nginx
```

### GitHub CLI

Interact with GitHub repositories:

```
> Create a pull request

gh pr create --title "Fix bug" --body "Details"
gh issue list --label bug
gh repo view
gh workflow run tests.yml
```

### System Utilities

File operations and system commands:

```
> Find all Python files

find . -name "*.py"
ls -la src/
cat config.json
wc -l src/*.py
df -h
ps aux | grep python
```

## Background Processes

Use `run_in_background: true` for long-running processes that shouldn't block:

```
> Start the development server

[Tool: bash]
{
  "command": "npm run dev",
  "run_in_background": true
}

Returns immediately with PID: 12345
```

### When to Use Background Mode

**Use background mode for:**
- Development servers (`npm run dev`, `python manage.py runserver`)
- File watchers (`npm run watch`, `cargo watch`)
- Database servers (`mongod`, `redis-server`)
- Any process that runs continuously

**Don't use background mode for:**
- Quick commands that finish in seconds
- Commands where you need to see the full output
- Build or test commands that should block

### Managing Background Processes

Once started, background processes run independently:

```
> Start the dev server
npm run dev  # Started in background as PID 12345

> Kill the process later
kill 12345
# or
pkill -f "npm run dev"
```

## Safety Features

### Blocked Commands

Amplifier blocks destructive operations to protect your system:

```
❌ BLOCKED:
rm -rf /
sudo rm -rf /
dd if=/dev/zero of=/dev/sda
mkfs.ext4 /dev/sda1
:(){ :|:& };:  # Fork bomb
```

### Interactive Commands Not Supported

Commands requiring user input will fail:

```
❌ NOT SUPPORTED:
vim file.txt
nano config.ini
python  # Interactive REPL
npm init  # Without -y flag
sudo apt install package  # Without -y flag
```

**Workaround:** Use non-interactive flags:

```
✅ WORKS:
npm init -y
sudo apt install -y package
python script.py  # Non-interactive script
```

### Output Truncation

Long outputs are automatically truncated to prevent context overflow:

```
Command: find / -name "*.log"
Output: /var/log/syslog
        /var/log/auth.log
        ... [truncated 1847 lines] ...
        /home/user/app.log
        
[Truncated: 2000/5234 lines, 45KB shown, 180KB total]
```

**Workaround for large outputs:**

```bash
# Redirect to file instead
find / -name "*.log" > results.txt

# Then read the file in chunks
head -n 100 results.txt
tail -n 100 results.txt
```

### Structured Data Warning

⚠️ **JSON/XML truncation breaks parsing:**

```bash
# ❌ Bad: Large JSON may get truncated mid-structure
curl https://api.example.com/large-dataset

# ✅ Good: Save to file first
curl https://api.example.com/large-dataset > data.json
# Then read specific parts
```

## Best Practices

### Path Handling

Quote paths with spaces and prefer absolute paths:

```bash
# ✅ Good
cd "/path/with spaces/project"
pytest /home/user/project/tests/

# ❌ Problematic
cd /path/with spaces/project  # Breaks on spaces
pytest ../tests/  # Relative paths lose context
```

### Command Chaining

Chain dependent commands with `&&` to stop on failure:

```bash
# ✅ Stops if mkdir fails
mkdir build && cd build && cmake ..

# ❌ Continues even if mkdir fails
mkdir build; cd build; cmake ..
```

### Error Handling

Check command exit codes for critical operations:

```bash
# ✅ Explicit error handling
npm test || echo "Tests failed with exit code $?"

# ✅ Conditional execution
if pytest tests/; then
    echo "Tests passed, deploying..."
    ./deploy.sh
fi
```

### Working Directory Context

Remember that each bash invocation maintains context:

```
> Create directory and enter it
mkdir myproject && cd myproject

> Next command runs in myproject/
pwd  # Shows: /home/user/myproject
```

### Prefer Specialized Tools

Before using bash, check if a specialized tool exists:

```
# ❌ Avoid: Manual file operations
bash: cat file.py

# ✅ Better: Use read_file
[Tool: read_file]

# ❌ Avoid: Manual search
bash: find . -name "*.py" | xargs grep "class User"

# ✅ Better: Use grep tool
[Tool: grep]
pattern: "class User"
type: "py"
```

## Try It Yourself

Practice these common scenarios:

### 1. Run Tests
```
> Run my Python tests with coverage

Result: Executes pytest with coverage report
```

### 2. Check Git Status
```
> Show me what files have changed

Result: Displays git status and diff
```

### 3. Install Dependencies
```
> Install packages from requirements.txt

Result: Runs pip install -r requirements.txt
```

### 4. Start Development Server
```
> Start the Next.js dev server

Result: Launches npm run dev in background
```

### 5. Build Project
```
> Build the Rust project in release mode

Result: Runs cargo build --release
```

## Errors and Troubleshooting

### Command Not Found

```
Error: bash: pytest: command not found

Solution:
1. Verify installation: which pytest
2. Install if missing: pip install pytest
3. Check PATH: echo $PATH
4. Use full path: /usr/local/bin/pytest
```

### Permission Denied

```
Error: Permission denied: ./script.sh

Solution:
1. Make executable: chmod +x script.sh
2. Or run with interpreter: bash script.sh
```

### Timeout Issues

```
Error: Command timed out after 300s

Solution:
1. Use background mode for long processes
2. Break into smaller commands
3. Add progress indicators to scripts
```

### Truncated Output

```
Warning: Output truncated [showing 2000/8000 lines]

Solution:
1. Redirect to file: command > output.txt
2. Filter output: command | grep "ERROR"
3. Use specialized tools for structured data
```

### Working Directory Confusion

```
Error: tests/ directory not found

Solution:
1. Check current directory: pwd
2. Use absolute paths: /home/user/project/tests/
3. Chain cd command: cd project && pytest tests/
```

### Interactive Command Hangs

```
Error: Command appears to hang (waiting for input)

Solution:
1. Add non-interactive flags: npm init -y
2. Use expect/heredoc for scripted input
3. Pre-create config files before running
```

### Environment Variables

```
Error: Missing required environment variable

Solution:
1. Set inline: API_KEY=abc123 python script.py
2. Export first: export API_KEY=abc123 && python script.py
3. Use .env file: source .env && python script.py
```

## Advanced Patterns

### Multi-line Commands

```bash
# Use line continuation for readability
docker run -d \
  --name myapp \
  -p 8080:80 \
  -v $(pwd):/app \
  myimage:latest
```

### Conditional Logic

```bash
# Run tests, deploy only if successful
pytest tests/ && npm run build && ./deploy.sh
```

### Output Processing

```bash
# Filter and transform output
git log --oneline | head -n 10 | cut -d' ' -f2-
```

### Variable Substitution

```bash
# Use shell variables
VERSION=$(cat VERSION)
docker build -t myapp:$VERSION .
```

---

**Next Steps:**
- Learn about [Specialized Tools](/tools/specialized) for better alternatives
- Explore [Git Operations](/tools/git) for version control
- See [Python Check](/tools/python-check) for code quality
