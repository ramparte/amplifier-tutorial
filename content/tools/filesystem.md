---
id: tool-filesystem
type: tools
title: "Filesystem Tool"
---

# Filesystem Tool

The Filesystem Tool provides direct access to read, write, and edit files on your local system. Unlike shell commands that require parsing text output, these tools offer structured, reliable file operations with built-in safety features.

## Overview

The filesystem capabilities are split across three specialized tools, each optimized for a specific operation type:

| Operation | Tool | Purpose |
|-----------|------|---------|
| Read | `read_file` | Read file contents or list directories |
| Write | `write_file` | Create new files or overwrite existing ones |
| Edit | `edit_file` | Make surgical string replacements in existing files |

This separation ensures clarity of intent and enables appropriate safety checks for each operation type.

---

## Reading Files

The `read_file` tool retrieves file contents with line numbers, making it easy to reference specific locations in code.

### Basic Usage

```
read_file("src/config.py")
```

Output format uses `cat -n` style with line numbers:

```
     1	import os
     2	from pathlib import Path
     3	
     4	CONFIG_DIR = Path("~/.config/myapp").expanduser()
     5	DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

### Reading Large Files

For files too large to read at once, use `offset` and `limit` parameters:

```
# Read lines 100-200 of a large log file
read_file("logs/application.log", offset=100, limit=100)
```

| Parameter | Description | Default |
|-----------|-------------|---------|
| `file_path` | Path to the file (required) | - |
| `offset` | Starting line number (1-indexed) | 1 |
| `limit` | Maximum lines to read | 2000 |

### Line Length Truncation

Lines longer than 2000 characters are automatically truncated to prevent context overflow. This is common with:

- Minified JavaScript/CSS files
- Generated code with long lines
- Log files with embedded JSON

### Reading Directories

When given a directory path, `read_file` returns a formatted listing:

```
read_file("src/components")
```

Output:

```
DIR  Button/
DIR  Form/
FILE index.ts
FILE types.ts
```

---

## Writing Files

The `write_file` tool creates new files or completely overwrites existing ones.

### Basic Usage

```
write_file("src/utils/helpers.py", """
def format_date(dt):
    return dt.strftime("%Y-%m-%d")

def slugify(text):
    return text.lower().replace(" ", "-")
""")
```

### When to Use write_file

Use `write_file` when:

- Creating a new file that doesn't exist
- Completely replacing a file's contents
- The file is small enough to rewrite entirely
- You've already read the file and need to restructure it significantly

Avoid `write_file` when:

- Making small changes to existing files (use `edit_file` instead)
- You haven't read the current contents first
- The file is large and you only need to change a few lines

### Safety Requirements

The tool enforces a critical safety rule:

> You must read a file before you can write to it.

This prevents accidental overwrites of files you haven't examined. The tool will fail if you attempt to write to an existing file without reading it first in the conversation.

---

## Editing Files

The `edit_file` tool performs precise string replacements, making it ideal for surgical changes to existing code.

### Basic Usage

```
edit_file(
    file_path="src/config.py",
    old_string="DEBUG = False",
    new_string="DEBUG = True"
)
```

### How It Works

1. The tool searches for `old_string` in the file
2. If found exactly once, it replaces with `new_string`
3. If not found or found multiple times, the operation fails

### Handling Non-Unique Strings

If your `old_string` appears multiple times, you have two options:

**Option 1: Add Context**

Include surrounding lines to make the match unique:

```
edit_file(
    file_path="src/handlers.py",
    old_string="""def process_user(user):
    return user.name""",
    new_string="""def process_user(user):
    return user.display_name"""
)
```

**Option 2: Replace All Occurrences**

Use `replace_all=True` to replace every instance:

```
edit_file(
    file_path="src/handlers.py",
    old_string="user.name",
    new_string="user.display_name",
    replace_all=True
)
```

### Preserving Indentation

When copying text from `read_file` output, preserve the exact indentation. The line number prefix format is:

```
     5	    def method(self):
```

Everything after the tab following the line number is actual file content. Match this exactly in your `old_string`.

### Multi-Line Edits

For changes spanning multiple lines, include all relevant lines:

```
edit_file(
    file_path="src/api.py",
    old_string="""@app.route("/users")
def get_users():
    return jsonify(users)""",
    new_string="""@app.route("/users")
@require_auth
def get_users():
    return jsonify(users)"""
)
```

---

## Directory Listing

The `read_file` tool doubles as a directory lister when given a directory path.

### Basic Listing

```
read_file("src/")
```

### Understanding Output

```
DIR  components/
DIR  utils/
FILE app.py
FILE config.py
FILE __init__.py
```

- `DIR` entries are subdirectories (shown with trailing `/`)
- `FILE` entries are regular files
- Results are sorted with directories first

### Navigating Deep Structures

To explore nested directories, make multiple calls:

```
read_file("src/")           # See top-level structure
read_file("src/components") # Dive into components
read_file("src/components/Button")  # Explore specific component
```

---

## Best Practices

### 1. Always Read Before Editing

Never attempt to edit a file you haven't read in the current conversation:

```
# Correct workflow
read_file("config.yaml")    # First, understand current state
edit_file("config.yaml", old_string="...", new_string="...")

# Incorrect - will fail
edit_file("config.yaml", old_string="...", new_string="...")  # Haven't read it!
```

### 2. Prefer edit_file Over write_file

For existing files, `edit_file` is safer and more precise:

| Scenario | Recommended Tool |
|----------|------------------|
| Change one function | `edit_file` |
| Update a config value | `edit_file` |
| Rename a variable throughout | `edit_file` with `replace_all` |
| Create new file | `write_file` |
| Complete rewrite | `write_file` (after reading) |

### 3. Use Exact Matches

The `old_string` must match exactly, including:

- Whitespace (spaces, tabs, newlines)
- Indentation
- Line endings

### 4. Handle Large Files Strategically

For files exceeding 2000 lines:

1. Use `offset` and `limit` to read in chunks
2. Identify the specific section needing changes
3. Use `edit_file` for targeted modifications

### 5. Verify After Writing

After significant changes, read the file to confirm:

```
write_file("new_module.py", content)
read_file("new_module.py")  # Verify it looks correct
```

---

## Try It Yourself

### Exercise 1: Read and Explore

Explore a project structure:

1. "List the contents of the src directory"
2. "Show me what's in src/utils"
3. "Read the first 50 lines of src/main.py"

### Exercise 2: Create a New File

Create a configuration file:

1. "Create a file called config.json with database settings"
2. Verify: "Read config.json to confirm"

### Exercise 3: Make Surgical Edits

Practice precise editing:

1. "Read src/constants.py"
2. "Change MAX_RETRIES from 3 to 5"
3. "Replace all occurrences of 'localhost' with '127.0.0.1'"

### Exercise 4: Multi-Line Edit

Try editing across lines:

1. "Read the authenticate function in src/auth.py"
2. "Add a logging statement at the start of the function"

---

## Errors and Troubleshooting

### "File not found"

The specified path doesn't exist. Check:

- Spelling and case sensitivity (Linux is case-sensitive)
- Relative vs absolute paths
- Whether you're in the expected working directory

### "old_string not found in file"

Your search string doesn't match the file contents exactly:

- Re-read the file to see current contents
- Check for whitespace differences
- Verify indentation matches exactly
- Look for invisible characters or different line endings

### "old_string appears multiple times"

The string isn't unique. Solutions:

- Add surrounding context to make it unique
- Use `replace_all=True` if you want to replace all occurrences

### "Must read file before writing"

You attempted to overwrite a file without reading it first:

- Read the file first: `read_file("path/to/file")`
- Then proceed with your write or edit operation

### "Line truncated at 2000 characters"

The file contains very long lines:

- This is informational, not an error
- Consider if the file is minified or generated
- You can still edit the file, but may need to work with truncated content

### "Permission denied"

The file or directory has restrictive permissions:

- Check file ownership and permissions
- Some system files require elevated privileges
- Consider whether you should be modifying this file

---

## Summary

| Tool | Use For | Key Parameters |
|------|---------|----------------|
| `read_file` | Reading contents, listing directories | `offset`, `limit` |
| `write_file` | Creating new files, complete rewrites | `content` |
| `edit_file` | Precise string replacements | `old_string`, `new_string`, `replace_all` |

The filesystem tools provide a safe, structured approach to file operations. By separating read, write, and edit operations, each action has appropriate safeguards while remaining straightforward to use.

**Next:** [Search Tool](search.md) - Finding files and content across your codebase
