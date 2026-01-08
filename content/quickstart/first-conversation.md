---
id: first-conversation
type: quickstart
title: "Your First Conversation"
---

# Your First Conversation

Now that Amplifier is installed, let's explore what it can do.

## Starting a Session

```bash
# Start interactive mode
amplifier
```

You'll see a prompt where you can type messages:

```
amplifier> _
```

Type your first message:

```
amplifier> Hello! What can you help me with?
```

## Understanding the Response

Amplifier will respond and may use **tools** to help you. Watch for tool usage in the output:

```
I can help you with software development tasks. I have access to several tools:

[Tool: read_file] Reading project structure...
[Tool: bash] Running: ls -la

Based on what I see, I can:
- Read and write files
- Run shell commands
- Search the web
- Navigate code with semantic understanding
...
```

## Try These Examples

### Example 1: Explore a Codebase

```
amplifier> What's in the current directory? Give me a summary.
```

Amplifier will:
1. Use the `bash` tool to run `ls` or `tree`
2. Read key files like `README.md` or `package.json`
3. Summarize what it finds

### Example 2: Write Some Code

```
amplifier> Create a Python function that checks if a number is prime. Save it to prime.py
```

Amplifier will:
1. Write the code using the `write_file` tool
2. Show you what it created
3. You can verify: `cat prime.py`

### Example 3: Debug Something

```
amplifier> This Python code has a bug. Find and fix it:

def factorial(n):
    if n == 0:
        return 0  # Bug here!
    return n * factorial(n - 1)
```

Amplifier will identify the bug (should return 1, not 0) and explain the fix.

### Example 4: Search and Learn

```
amplifier> What's the latest version of React? Search the web.
```

Amplifier uses the `web_search` tool to find current information.

## Session Commands

While in a session, you can use special commands starting with `/`:

```
/help        - Show available commands
/tools       - List available tools  
/agents      - List available agents
/clear       - Clear conversation history
/exit        - End the session
```

Try it:

```
amplifier> /tools
```

You'll see all the tools currently loaded and what they do.

## Single-Shot Mode

Don't need a conversation? Run a single command:

```bash
# Run one task and exit
amplifier run "Count the lines in all Python files in this directory"

# Pipe input
cat error.log | amplifier run "What's causing these errors?"

# Specify output format
amplifier run "List all TODO comments in src/" --format json
```

## Resuming Sessions

Every session is saved. Come back to where you left off:

```bash
# Resume most recent session
amplifier continue

# List all sessions
amplifier session list

# Resume a specific session
amplifier session resume [session-id]
```

## What Just Happened?

Behind the scenes, Amplifier:

1. **Started a session** with your configured provider (e.g., Claude)
2. **Loaded modules** - tools, hooks, and other capabilities
3. **Managed context** - keeping track of the conversation
4. **Logged events** - every action recorded to `~/.amplifier/sessions/`

You can inspect any session:

```bash
# Find your session
ls ~/.amplifier/sessions/

# See the transcript
cat ~/.amplifier/sessions/[id]/transcript.md
```

## Tips for Effective Use

### Be Specific
```
# Vague
amplifier> Fix the bug

# Specific  
amplifier> The login function in src/auth.py returns None when it should return a User object. Fix it.
```

### Provide Context
```
# Give relevant files
amplifier> Here's my config file. Update it to use port 8080:
$(cat config.yaml)
```

### Use Multi-Turn Conversations
```
amplifier> Create a REST API endpoint for user registration
# ... Amplifier creates it ...

amplifier> Add input validation
# ... Amplifier updates the code ...

amplifier> Now add tests for it
# ... Amplifier creates tests ...
```

## Try It Yourself

1. **Create something**: Ask Amplifier to create a simple script in your language of choice
2. **Explore**: Point it at an existing project and ask for a summary
3. **Debug**: Give it some buggy code and ask for fixes

## Next

Learn the key commands that make you productive.

→ [Key Commands](key-commands.md)
