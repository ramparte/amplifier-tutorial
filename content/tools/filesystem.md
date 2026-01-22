---
id: tool-filesystem
type: tools
title: "Filesystem Tool"
---

# Filesystem Tool

The Filesystem Tool provides three essential operations for file manipulation: reading, writing, and editing files. These tools work with absolute paths, relative paths, and @mention paths for bundle resources.

## Operations

| Operation | Tool | Purpose |
|-----------|------|---------|
| Read | read_file | Read file contents or list directories |
| Write | write_file | Create or overwrite entire files |
| Edit | edit_file | Perform surgical string replacements |

## Reading Files

The `read_file` tool reads file contents or lists directory contents with line numbers.

### Basic File Reading

```json
{
  "file_path": "./src/main.py"
}
```

Returns the file with line numbers in `cat -n` format:
```
1    import sys
2    
3    def main():
4        print("Hello, world!")
```

### Reading Large Files

For files with many lines, use `offset` and `limit` parameters:

```json
{
  "file_path": "./logs/app.log",
  "offset": 100,
  "limit": 50
}
```

This reads lines 100-150, useful for:
- Inspecting specific sections of large logs
- Paginating through long files
- Reducing context consumption

**Default limits:**
- Maximum 2000 lines per read
- Lines longer than 2000 characters are truncated
- Line numbers start at 1 (not 0)

### Directory Listing

When `file_path` points to a directory, `read_file` returns a formatted listing:

```json
{
  "file_path": "./src"
}
```

Returns:
```
DIR: components/
DIR: utils/
FILE: main.py
FILE: config.py
```

### Path Formats Supported

**Absolute paths:**
```json
{"file_path": "/home/user/project/file.md"}
```

**Relative paths:**
```json
{"file_path": "./docs/README.md"}
```

**Bundle resources:**
```json
{"file_path": "@mybundle:docs/guide.md"}
```

**Bundle directories:**
```json
{"file_path": "@mybundle:docs"}
```

## Writing Files

The `write_file` tool creates new files or overwrites existing ones completely.

### Basic Writing

```json
{
  "file_path": "./src/new_module.py",
  "content": "def calculate(x, y):\n    return x + y\n"
}
```

### Important Rules

⚠️ **MUST read existing files first:**
```json
// Step 1: Read first
{"file_path": "./src/existing.py"}

// Step 2: Then write
{
  "file_path": "./src/existing.py",
  "content": "new content"
}
```

The tool will fail if you attempt to overwrite an existing file without reading it first. This safety mechanism prevents accidental overwrites.

### When to Use write_file

**✅ Good use cases:**
- Creating entirely new files
- Generating configuration files
- Writing output from code generation
- Creating test fixtures

**❌ Avoid for:**
- Modifying existing files (use `edit_file` instead)
- Small changes to files (use `edit_file` instead)
- Adding single functions to modules (use `edit_file` instead)

### Content Guidelines

- Avoid emoji unless explicitly requested by user
- Never proactively create documentation (*.md) files
- Always prefer editing existing files over creating new ones

## Editing Files

The `edit_file` tool performs precise string replacements in existing files.

### Basic Editing

```json
{
  "file_path": "./src/main.py",
  "old_string": "def calculate(x):\n    return x * 2",
  "new_string": "def calculate(x, multiplier=2):\n    return x * multiplier"
}
```

### Preserving Indentation

⚠️ **Critical:** Preserve exact indentation as shown in `read_file` output.

The line number format is: `spaces + number + tab + content`

**Example from read_file:**
```
    42    def helper():
    43        return True
```

**Correct old_string** (after the tab):
```
"def helper():\n    return True"
```

**Incorrect** (includes line numbers):
```
"    42    def helper():\n    43        return True"
```

### Ensuring Uniqueness

The tool requires `old_string` to be **unique** in the file:

**❌ Will fail** (matches multiple locations):
```json
{
  "old_string": "return x"
}
```

**✅ Include context** to make it unique:
```json
{
  "old_string": "def calculate(x):\n    return x"
}
```

### Replace All Occurrences

Use `replace_all: true` for renaming variables or repeated patterns:

```json
{
  "file_path": "./src/user.py",
  "old_string": "userName",
  "new_string": "user_name",
  "replace_all": true
}
```

**Perfect for:**
- Variable renaming across a file
- Updating repeated string literals
- Changing function names
- Updating import paths

### When to Use edit_file

**✅ Preferred for:**
- Modifying functions or classes
- Adding imports
- Changing variable names
- Updating configuration values
- Bug fixes in existing code

**✅ Benefits:**
- Preserves surrounding code
- Safer than full file rewrites
- More efficient for small changes
- Less prone to losing unrelated changes

## Directory Listing

List directory contents with `read_file`:

```json
{
  "file_path": "./src/components"
}
```

**Output shows:**
- `DIR:` prefix for subdirectories
- `FILE:` prefix for files
- Alphabetically sorted
- Relative names only (not full paths)

**Use cases:**
- Exploring project structure
- Finding configuration files
- Discovering test files
- Checking bundle contents

## Best Practices

### 1. Always Read Before Editing

```json
// ✅ Correct workflow
read_file("./src/app.py")
// ... review contents ...
edit_file("./src/app.py", old_string, new_string)
```

```json
// ❌ Will fail
edit_file("./src/app.py", old_string, new_string)
```

### 2. Prefer edit_file Over write_file

For existing files, surgical edits are safer:

```json
// ✅ Preferred - surgical edit
edit_file({
  "old_string": "version = \"1.0\"",
  "new_string": "version = \"2.0\""
})
```

```json
// ❌ Risky - full rewrite
write_file({
  "content": "... entire file contents ..."
})
```

### 3. Use Parallel Reads

Read multiple files in one message:

```json
[
  {"file_path": "./src/main.py"},
  {"file_path": "./src/utils.py"},
  {"file_path": "./tests/test_main.py"}
]
```

### 4. Handle Large Files Efficiently

```json
// Check file size first
read_file("./data/large.json", 0, 10)

// Read in chunks if needed
read_file("./data/large.json", 0, 100)
read_file("./data/large.json", 100, 100)
```

### 5. Verify Bundle Paths

For bundle resources, check directory listing first:

```json
// List available files
read_file("@recipes:examples")

// Then read specific file
read_file("@recipes:examples/workflow.yaml")
```

## Try It Yourself

### Exercise 1: Read and Edit a File

1. Create a test file:
```json
write_file({
  "file_path": "./test.py",
  "content": "def greet(name):\n    return f'Hello {name}'"
})
```

2. Read it back:
```json
read_file({"file_path": "./test.py"})
```

3. Edit the function:
```json
edit_file({
  "file_path": "./test.py",
  "old_string": "def greet(name):\n    return f'Hello {name}'",
  "new_string": "def greet(name, title='Friend'):\n    return f'Hello {title} {name}'"
})
```

### Exercise 2: Explore a Directory Structure

```json
read_file({"file_path": "./content"})
read_file({"file_path": "./content/tools"})
```

### Exercise 3: Variable Renaming

```json
edit_file({
  "file_path": "./src/config.py",
  "old_string": "max_retries",
  "new_string": "maximum_retry_count",
  "replace_all": true
})
```

## Errors and Troubleshooting

### Error: "Must read file before editing"

**Cause:** Attempted to edit without reading first.

**Solution:**
```json
read_file({"file_path": "./target.py"})
// Then edit
```

### Error: "old_string not unique"

**Cause:** The string appears multiple times in the file.

**Solutions:**
1. Add more context to make it unique:
```json
{
  "old_string": "return value  // More context needed",
  "new_string": "function calculate() {\n    return value\n}"
}
```

2. Use `replace_all: true` if appropriate:
```json
{"replace_all": true}
```

### Error: "File not found"

**Cause:** Path doesn't exist or is incorrect.

**Solutions:**
- Verify path with directory listing
- Check for typos in filename
- Use absolute path if relative path is ambiguous
- Confirm bundle name for @mention paths

### Warning: Empty File Contents

If you read a file that exists but is empty, you'll receive a warning. This helps distinguish between:
- File doesn't exist (error)
- File exists but is empty (warning)
- File has content (normal output)

### Indentation Mismatch

**Cause:** Including line numbers or wrong whitespace in `old_string`.

**Solution:**
- Copy exact content after the line number tab
- Preserve spaces/tabs exactly as shown
- Don't include the line number prefix

### Line Truncation

Lines longer than 2000 characters are truncated. For such files:
- Use specialized tools for specific content types (JSON parsers, etc.)
- Process in smaller chunks
- Consider using `bash` tool for stream processing

## Advanced Patterns

### Conditional Edits

Read first, decide, then edit based on content:

```python
# 1. Read file
content = read_file("./config.yaml")

# 2. Analyze content
if "debug: false" in content:
    # 3. Edit accordingly
    edit_file(
        old_string="debug: false",
        new_string="debug: true"
    )
```

### Multi-Step Refactoring

```python
# 1. Read original
read_file("./src/api.py")

# 2. First edit - rename function
edit_file(old_string="def old_name():", new_string="def new_name():")

# 3. Second edit - update callers
edit_file(old_string="old_name()", new_string="new_name()", replace_all=True)
```

### Safe Templating

```python
# 1. Read template
template = read_file("@mybundle:templates/component.tsx")

# 2. Write new file with modifications
write_file(
    file_path="./src/components/MyComponent.tsx",
    content=template.replace("{{NAME}}", "MyComponent")
)
```

## Summary

The Filesystem Tool provides three complementary operations:

- **read_file**: View contents, paginate large files, list directories
- **write_file**: Create new files (read existing files first!)
- **edit_file**: Surgical replacements (prefer over write_file)

**Key principles:**
1. Always read before editing
2. Prefer edit over write for existing files
3. Ensure old_string is unique or use replace_all
4. Preserve exact indentation from read_file output
5. Use parallel reads for efficiency

These tools form the foundation of file manipulation in Amplifier, enabling precise control over your project's files.
