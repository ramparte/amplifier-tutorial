---
id: tool-filesystem
type: tools
title: "Filesystem Tool"
---

# Filesystem Tool

Read, write, and edit files in your project.

## Overview

The filesystem tool provides three operations:

| Operation | Purpose | When to Use |
|-----------|---------|-------------|
| `read_file` | Read file contents | View code, configs, docs |
| `write_file` | Create or overwrite | New files, full rewrites |
| `edit_file` | Targeted replacements | Small changes, refactoring |

## read_file

Read the contents of a file.

### Basic Usage

```
> Show me the contents of config.yaml
```

Behind the scenes:
```
[Tool: read_file]
file_path: config.yaml
```

### Reading Specific Lines

For large files, specify offset and limit:

```
> Show me lines 100-150 of app.py
```

```
[Tool: read_file]
file_path: app.py
offset: 100
limit: 50
```

### Directory Listing

Point to a directory to see its contents:

```
> What files are in src/
```

```
[Tool: read_file]
file_path: src/
```

Returns:
```
DIR  components/
DIR  utils/
FILE app.py
FILE config.yaml
```

## write_file

Create a new file or completely overwrite an existing one.

### Creating Files

```
> Create a Python script that calculates fibonacci numbers
```

```
[Tool: write_file]
file_path: fibonacci.py
content: |
  def fibonacci(n):
      if n <= 1:
          return n
      return fibonacci(n-1) + fibonacci(n-2)
  
  if __name__ == "__main__":
      for i in range(10):
          print(f"fib({i}) = {fibonacci(i)}")
```

### Overwriting Files

```
> Replace the entire config with a new version
```

**Warning**: This overwrites the entire file. For small changes, use `edit_file`.

## edit_file

Make targeted string replacements without rewriting the whole file.

### Basic Edit

```
> Change the port from 3000 to 8080 in config.yaml
```

```
[Tool: edit_file]
file_path: config.yaml
old_string: "port: 3000"
new_string: "port: 8080"
```

### Multi-line Edits

```
> Add error handling to the fetch function
```

```
[Tool: edit_file]
file_path: api.py
old_string: |
  def fetch_data(url):
      response = requests.get(url)
      return response.json()
new_string: |
  def fetch_data(url):
      try:
          response = requests.get(url)
          response.raise_for_status()
          return response.json()
      except requests.RequestException as e:
          logger.error(f"Fetch failed: {e}")
          raise
```

### Replace All Occurrences

```
> Rename the function from 'getData' to 'fetchData' everywhere in this file
```

```
[Tool: edit_file]
file_path: utils.js
old_string: "getData"
new_string: "fetchData"
replace_all: true
```

## Best Practices

### Read Before Edit

Always read a file before editing to understand context:

```
> Read src/auth.py then fix the login function
```

The AI will:
1. Read the file
2. Understand the structure
3. Make targeted edits

### Use edit_file for Small Changes

```
# Good - targeted edit
[edit_file] old: "timeout: 30" new: "timeout: 60"

# Avoid - rewriting entire file for one change
[write_file] content: [entire file with one line changed]
```

### Check Your Changes

After edits:
```
> Show me the file again to verify the changes
```

## Common Patterns

### Add Import

```
[Tool: edit_file]
file_path: app.py
old_string: "import os"
new_string: "import os\nimport json"
```

### Add Function

```
[Tool: edit_file]
file_path: utils.py
old_string: "# End of file"
new_string: |
  def new_helper():
      pass
  
  # End of file
```

### Modify Configuration

```
[Tool: edit_file]
file_path: package.json
old_string: '"version": "1.0.0"'
new_string: '"version": "1.1.0"'
```

## Try It Yourself

### Exercise 1: Create and Read

```
> Create a file called hello.py that prints "Hello, World!"
> Now show me what's in hello.py
```

### Exercise 2: Edit in Place

```
> Change hello.py to print "Hello, Amplifier!" instead
```

### Exercise 3: Explore a Directory

```
> List all files in the current directory recursively
```

## Errors and Troubleshooting

### "File not found"

The file doesn't exist at that path. Check:
- Spelling of filename
- Working directory
- Relative vs absolute path

### "Edit failed: old_string not found"

The exact string wasn't found. This can happen if:
- Whitespace differs (spaces vs tabs)
- Line endings differ
- The file changed since you read it

**Solution**: Read the file again and use the exact string.

### "Permission denied"

You don't have write access. Check file permissions:
```bash
ls -la filename
```
