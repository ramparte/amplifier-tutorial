# Bash Tool - Background Processes Supplement

This content is injected into the bash tool tutorial page.

## Background Processes

For long-running commands like dev servers or file watchers, use background mode.

### Starting Background Processes

```
> Start the dev server
```

```
[Tool: bash]
command: npm run dev
run_in_background: true
```

Returns immediately with a PID:

```
Process started in background with PID 12345
```

The server continues running while you do other work.

### Common Background Use Cases

**Development Servers:**
```
> Start the Flask server in the background

[Tool: bash]
command: python app.py
run_in_background: true
```

**File Watchers:**
```
> Watch for TypeScript changes

[Tool: bash]
command: tsc --watch
run_in_background: true
```

**Build Processes:**
```
> Start a long build

[Tool: bash]
command: make all
run_in_background: true
```

### Managing Background Processes

Check if a process is running:

```
[bash] ps aux | grep npm
```

Stop a background process:

```
[bash] kill 12345  # Use the PID from earlier
```

### When NOT to Use Background Mode

- Commands you need output from immediately
- Quick operations (< 30 seconds)
- Commands that need to complete before the next step
