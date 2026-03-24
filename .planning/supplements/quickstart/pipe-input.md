# Quickstart - Pipe Input Supplement

This content is injected into quickstart pages about CLI usage.

## Piping Input

Send file contents or command output directly to Amplifier.

### Basic Pipe

```bash
# Analyze a file
cat error.log | amplifier run "What's causing these errors?"

# Process command output
git diff | amplifier run "Summarize these changes"

# Multiple files
cat src/*.py | amplifier run "Find potential bugs in this code"
```

### Practical Examples

**Log Analysis:**
```bash
tail -100 /var/log/app.log | amplifier run "Are there any errors? What do they mean?"
```

**Code Review:**
```bash
git diff HEAD~1 | amplifier run "Review this commit for issues"
```

**Config Validation:**
```bash
cat docker-compose.yml | amplifier run "Is this configuration valid? Any improvements?"
```

**Data Processing:**
```bash
curl -s api.example.com/data | amplifier run "Parse this JSON and summarize the key metrics"
```

### Output Formats

Control how Amplifier responds:

```bash
# Plain text (default)
cat data.csv | amplifier run "Summarize this data"

# JSON output
cat data.csv | amplifier run "Extract key metrics" --format json

# Markdown
cat README.md | amplifier run "Improve this documentation" --format markdown
```

### Combining with Other Tools

```bash
# Find and fix
grep -r "TODO" src/ | amplifier run "Prioritize these TODOs"

# Generate and save
cat spec.md | amplifier run "Generate Python code for this spec" > implementation.py

# Chain processing
cat raw_data.json | amplifier run "Clean this data" | amplifier run "Generate a report"
```
