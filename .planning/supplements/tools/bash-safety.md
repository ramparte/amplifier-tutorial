# Bash Tool - Safety Features Supplement

This content is injected into the bash tool tutorial page.

## Safety Features

The bash tool has built-in safety protections to prevent dangerous operations.

### Blocked Commands

These patterns are automatically blocked:

```
rm -rf /           # Root deletion
sudo rm -rf        # Privileged deletion  
:(){ :|:& };:      # Fork bomb
mkfs               # Filesystem formatting
dd if=/dev/zero    # Disk overwriting
> /dev/sda         # Direct device writes
```

If you try a blocked command:

```
> Delete everything in root

Error: Command denied for safety: Matches denied command pattern
```

### Timeout Protection

Commands timeout after 30 seconds by default. For long operations:

```
> This build might take a while
```

```
[Tool: bash]
command: npm run build
timeout: 300
```

Or use the timeout command:

```
[Tool: bash]
command: timeout 300 npm run build
```

### Interactive Commands

Interactive commands that require user input don't work:

```
# Won't work - requires interaction
[bash] vim file.txt
[bash] python  # interactive shell
[bash] ssh user@host  # without key auth

# Use non-interactive alternatives
[bash] cat file.txt              # instead of vim
[bash] python -c "print('hi')"   # instead of interactive python
[bash] ssh -o BatchMode=yes ...  # for scripted SSH
```

### Output Truncation

Long outputs are automatically truncated to prevent context overflow:

```
[bash] find / -name "*.py"

# Output shows:
# - First N lines
# - "[...truncated...]"
# - Last N lines
# - Byte counts
```

For large outputs, redirect to a file:

```
[bash] find / -name "*.py" > results.txt
```

Then read portions with `read_file`.
