# Filesystem Tool - Directory Operations Supplement

This content is injected into the filesystem tool tutorial page.

## Directory Listing

Point `read_file` at a directory to see its contents:

```
> What files are in the src folder?
```

```
[Tool: read_file]
file_path: src/
```

Returns a formatted listing:

```
DIR  components/
DIR  utils/
FILE app.py
FILE config.yaml
FILE README.md
```

### Nested Directory Exploration

```
> Show me what's in src/components/

[Tool: read_file]
file_path: src/components/
```

```
DIR  auth/
DIR  ui/
FILE Button.tsx
FILE Modal.tsx
FILE index.ts
```

### Combining with Bash for Trees

For deeper exploration:

```
> Show me the full project structure

[Tool: bash]
command: tree -L 2
```

Or with file counts:

```
> How many files in each directory?

[Tool: bash]
command: find . -type f | cut -d/ -f2 | sort | uniq -c | sort -rn
```
