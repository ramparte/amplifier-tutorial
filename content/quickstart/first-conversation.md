---
id: first-conversation
type: quickstart
title: "Your First Conversation"
---

# Your First Conversation

This guide walks you through your first interaction with Amplifier, helping you understand how to communicate effectively with the AI assistant and what to expect.

## Starting a Session

Launch Amplifier from your terminal:

```bash
amplifier run
```

You'll see the Amplifier prompt appear:

```
User:
```

This means Amplifier is ready to receive your instructions. You're now in an interactive session where you can have a natural conversation with the AI assistant.

## Your First Prompt

Start with something simple to get a feel for how Amplifier works. Here are some great first prompts:

### Example 1: Ask About Your Project

```
What files are in the current directory?
```

Amplifier will use the `glob` or `bash` tool to explore your workspace and provide a structured summary of what it finds.

### Example 2: Request Code Analysis

```
Read the main.py file and explain what it does
```

The assistant will use the `read_file` tool to access the file, then provide a detailed explanation of the code's functionality.

### Example 3: Simple Code Generation

```
Create a Python function that calculates the factorial of a number
```

Amplifier will write the code and can save it to a file if you specify a location.

### Example 4: Multi-Step Task

```
Find all TODO comments in my Python files and create a summary
```

This demonstrates Amplifier's ability to chain multiple operations: searching files, reading content, and synthesizing information.

## Understanding Responses

Amplifier responses typically include several components:

### 1. Thinking Process (Sometimes Visible)

The assistant may show its reasoning before taking action. This helps you understand its approach to solving your request.

### 2. Tool Usage

You'll see when Amplifier uses tools to accomplish tasks:

```
[Using read_file: ./src/main.py]
[Using bash: pytest tests/]
```

These indicators show what actions are being taken on your behalf.

### 3. Results and Explanations

After using tools, Amplifier provides:
- **Summaries**: Condensed information about what was found or done
- **Analysis**: Interpretation of results
- **Code blocks**: Formatted code with syntax highlighting
- **Recommendations**: Suggestions for next steps

### 4. Error Handling

If something goes wrong, Amplifier will:
- Explain what happened
- Suggest fixes or alternatives
- Often retry with a different approach automatically

## Tool Usage

Amplifier has access to various tools that extend its capabilities. Understanding these helps you make better requests.

### File Operations

**Reading files:**
```
Read the config.yaml file
```

**Writing files:**
```
Create a new file called utils.py with a helper function
```

**Editing files:**
```
Update the README.md to include installation instructions
```

### Code Quality

**Checking Python code:**
```
Check this Python file for errors and style issues
```

Amplifier uses `ruff` for formatting/linting and `pyright` for type checking.

### Search Operations

**Finding files:**
```
Find all JavaScript files in the src directory
```

**Searching content:**
```
Search for all functions named "calculate" in Python files
```

### Command Execution

**Running tests:**
```
Run the test suite
```

**Installing packages:**
```
Install the requests library
```

**Git operations:**
```
Show me the git status
```

### Complex Tasks with Sub-Agents

For complex, multi-step tasks, Amplifier can delegate work to specialized agents:

```
Review the code I just wrote and suggest improvements
```

This might spawn a `code-reviewer` agent that works autonomously.

## Best Practices for Prompts

### Be Specific

❌ **Vague**: "Fix the code"
✅ **Specific**: "Fix the syntax error in main.py on line 42"

### Provide Context

❌ **Minimal**: "Add a test"
✅ **Contextual**: "Add a unit test for the calculate_total function that checks edge cases like negative numbers and zero"

### Break Down Complex Requests

For very complex tasks, consider breaking them into steps:

```
1. First, analyze the current authentication system
2. Then suggest improvements for security
3. Finally, implement the changes
```

Or let Amplifier handle it all at once:

```
Analyze the auth system, suggest security improvements, and implement them
```

### Iterate and Refine

Don't worry about getting it perfect the first time. You can refine:

```
Actually, make that function async instead
```

```
Add error handling to the code you just wrote
```

## Common Workflows

### Code Review Workflow

```
User: I just updated the payment processor. Can you review it?
Assistant: [reads file, analyzes code, provides feedback]
User: Make those changes
Assistant: [edits file with improvements]
User: Now add tests for the edge cases
Assistant: [creates test file]
```

### Debugging Workflow

```
User: My tests are failing
Assistant: [runs tests, analyzes output]
User: What's causing the failure?
Assistant: [explains issue with specific line numbers]
User: How do I fix it?
Assistant: [provides solution and can implement it]
```

### Learning Workflow

```
User: Explain how async/await works in Python
Assistant: [provides explanation with examples]
User: Show me a practical example
Assistant: [creates working code example]
User: What are common pitfalls?
Assistant: [explains gotchas and best practices]
```

## Ending a Session

To exit Amplifier, use:

```bash
Ctrl+D (or type 'exit')
```

Your conversation history is saved, and any changes made to files are preserved in your workspace.

## Pro Tips

### Parallel Operations

Request multiple things at once:
```
Read both config.py and utils.py, then tell me how they interact
```

### Asking for Alternatives

```
Show me three different ways to implement this feature
```

### Context Awareness

Amplifier remembers your conversation history:
```
User: Create a User class
Assistant: [creates User class]
User: Now add a method to validate email addresses
Assistant: [knows which class you mean]
```

### Skill Loading

For specialized domains:
```
Load the REST API design skill and help me design an endpoint
```

## What Not to Expect

- **No internet access by default**: Amplifier works with local files (unless web-research agent is used)
- **No memory between sessions**: Each session starts fresh
- **No real-time interactions**: Can't run interactive programs directly
- **No destructive operations without review**: Safety guardrails prevent accidental damage

## Next Steps

Now that you understand the basics, explore:

1. **[Core Concepts](../concepts/overview.md)** - Deeper understanding of how Amplifier works
2. **[Common Workflows](../workflows/code-review.md)** - Step-by-step guides for typical tasks
3. **[Tool Reference](../reference/tools.md)** - Complete documentation of available tools
4. **[Best Practices](../guides/best-practices.md)** - Tips for getting the most out of Amplifier

Try experimenting with different types of requests to discover what Amplifier can do for you!

## Quick Reference Card

| Task | Example Prompt |
|------|----------------|
| Read a file | `Show me the contents of app.py` |
| Edit code | `Add error handling to the save function` |
| Run tests | `Run pytest and show me the results` |
| Search code | `Find all TODO comments` |
| Get help | `How do I use the grep tool?` |
| Check code quality | `Check this Python file for issues` |
| Create files | `Create a new module for database operations` |
| Explain code | `Explain what the authenticate function does` |

---

**Ready to dive deeper?** Continue to [Understanding Tools](./understanding-tools.md) to learn about Amplifier's capabilities in detail.
