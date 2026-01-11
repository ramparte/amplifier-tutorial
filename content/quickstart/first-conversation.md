---
id: first-conversation
type: quickstart
title: "Your First Conversation"
---

# Your First Conversation

This guide walks you through your first interactive session with Amplifier. You'll learn
how to start a conversation, write effective prompts, interpret responses, and understand
how Amplifier uses tools to accomplish tasks.

## Starting a Session

Launch Amplifier from your terminal in any project directory:

```bash
amplifier run
```

You'll see a welcome message and a prompt indicator waiting for your input:

```
Amplifier v1.0.0
Session: abc123-def456

>
```

The session ID helps you reference this conversation later if needed.

### Starting in a Specific Directory

To work on a particular project, navigate there first:

```bash
cd ~/projects/my-app
amplifier run
```

Amplifier automatically gains context about your project structure, making it more
effective at helping with your specific codebase.

## Your First Prompt

Start with something simple. Try asking Amplifier to explore your project:

```
> What files are in this project?
```

Or ask for help with a specific task:

```
> Create a Python function that validates email addresses
```

### Writing Effective Prompts

Good prompts are clear and specific. Compare these examples:

| Less Effective | More Effective |
|----------------|----------------|
| "Fix the bug" | "Fix the TypeError in auth.py line 42" |
| "Make it faster" | "Optimize the database query in get_users()" |
| "Add tests" | "Add unit tests for the UserService class" |

### Providing Context

You can reference files directly in your prompts:

```
> Review the code in src/utils/helpers.py for potential improvements
```

Or provide inline context:

```
> I'm getting this error when running tests:
> 
> AssertionError: expected 3 but got 5
> 
> The test is in tests/test_calculator.py
```

## Understanding Responses

Amplifier responds conversationally while working on your request. A typical response
includes several elements:

### Explanatory Text

Amplifier explains what it's doing and why:

```
I'll create an email validation function using regex. This will check for:
- A valid local part (before the @)
- A valid domain with at least one dot
- No invalid characters
```

### Code Blocks

Code appears in formatted blocks with syntax highlighting:

```python
import re

def validate_email(email: str) -> bool:
    """Validate an email address format."""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

### File References

When Amplifier references code locations, it uses the format `file_path:line_number`:

```
The validation logic is in src/validators.py:23
```

This makes it easy to navigate to the exact location in your editor.

## Tool Usage

Amplifier uses tools to interact with your system. You'll see tool activity as it works.

### Common Tools

| Tool | Purpose |
|------|---------|
| `read_file` | View file contents |
| `write_file` | Create new files |
| `edit_file` | Modify existing files |
| `bash` | Run shell commands |
| `grep` | Search file contents |
| `glob` | Find files by pattern |

### Observing Tool Usage

When Amplifier uses tools, you'll see indicators of what's happening:

```
> Add a new endpoint to the API

Reading src/api/routes.py...
Editing src/api/routes.py...
Running tests to verify changes...

I've added the new endpoint at /api/users/{id}/profile. The route is defined
in src/api/routes.py:45. Tests pass successfully.
```

### Tool Permissions

Some operations may require confirmation. Amplifier respects your system and won't
make destructive changes without appropriate safeguards.

## Multi-Turn Conversations

Sessions maintain context across multiple exchanges. You can build on previous work:

```
> Create a User class with name and email fields

[Amplifier creates the class]

> Add a method to validate the email

[Amplifier adds to the existing class]

> Now add unit tests for the validation

[Amplifier creates tests referencing the class it built]
```

### Referencing Previous Context

Use natural language to refer back:

```
> Actually, make that method raise an exception instead of returning False
```

Amplifier understands "that method" refers to the validation method from the
previous exchange.

## Handling Errors

If something goes wrong, Amplifier will explain and often suggest fixes:

```
> Run the deployment script

The deployment script failed with exit code 1:
  Error: Missing environment variable DATABASE_URL

To fix this, you can:
1. Set the variable: export DATABASE_URL="your-connection-string"
2. Create a .env file with the variable
3. Pass it inline: DATABASE_URL="..." ./deploy.sh
```

### Asking for Clarification

If your request is ambiguous, Amplifier asks for clarification rather than guessing:

```
> Delete the old files

I found several potential matches for "old files":
- backup/ directory (23 files, last modified 30 days ago)
- *.bak files (5 files across the project)
- deprecated/ folder (12 files)

Which would you like me to delete?
```

## Ending a Session

To end your conversation, simply type:

```
> exit
```

Or use the keyboard shortcut `Ctrl+C`.

Your session is automatically saved. You can resume it later or start fresh.

### Session Persistence

Sessions are stored locally and can be referenced later. To see recent sessions:

```bash
amplifier sessions list
```

To resume a specific session:

```bash
amplifier resume <session-id>
```

## Next Steps

Now that you've had your first conversation, explore these topics:

- **[Working with Files](./working-with-files.md)** - Deep dive into file operations
- **[Running Commands](./running-commands.md)** - Execute shell commands effectively
- **[Project Context](./project-context.md)** - Help Amplifier understand your codebase
- **[Advanced Prompting](../guides/advanced-prompting.md)** - Write more effective prompts

### Tips for Success

1. **Be specific** - Clear requests get better results
2. **Provide context** - Share error messages, file paths, and background
3. **Iterate** - Build on responses in multi-turn conversations
4. **Ask questions** - Amplifier can explain code and concepts
5. **Trust but verify** - Review generated code before committing

Welcome to Amplifier. Happy building.
