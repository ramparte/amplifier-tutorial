---
id: foundation
type: bundles
title: "Foundation Bundle"
---

# Foundation Bundle

## Overview

The Foundation Bundle is the default bundle that ships with Amplifier, providing essential tools and agents for general-purpose software development workflows. It forms the backbone of most development tasks, offering file operations, code search, shell access, web research capabilities, and a suite of specialized agents for common development patterns.

**Key Characteristics:**
- **Universal Coverage**: Handles 80% of common development tasks
- **Production-Ready**: Battle-tested tools with robust error handling
- **Zero Configuration**: Works out of the box with sensible defaults
- **Extensible**: Serves as a foundation for specialized bundles

The Foundation Bundle is automatically loaded when Amplifier starts, making its tools and agents immediately available without additional configuration.

## Tools Included

| Tool | Purpose | Key Use Cases |
|------|---------|---------------|
| **bash** | Shell command execution | Running builds, tests, package managers, git operations |
| **read_file** | Read files and directories | Inspecting code, configuration files, documentation |
| **write_file** | Create/overwrite files | Writing new code, generating configs, creating documentation |
| **edit_file** | Precise string replacements | Modifying existing code, refactoring, bug fixes |
| **grep** | Content search with regex | Finding function definitions, locating imports, tracking usage |
| **glob** | File pattern matching | Discovering project structure, finding files by type |
| **task** | Launch specialized agents | Delegating complex multi-step workflows to sub-agents |
| **load_skill** | Access domain knowledge | Loading best practices, coding standards, design patterns |
| **python_check** | Python code quality | Linting, formatting, type checking Python code |
| **recipes** | Execute workflows | Running multi-stage automated processes |
| **todo** | Task tracking | Managing complex multi-step work |

### Tool Details

#### Filesystem Tools
The `read_file`, `write_file`, and `edit_file` tools provide comprehensive file manipulation:
- Support for absolute and relative paths
- Bundle resource access via `@bundle:path` syntax
- Automatic backup before destructive operations
- Line-based reading for large files with offset/limit parameters

#### Search Tools
`grep` and `glob` enable fast codebase navigation:
- `grep`: Regex-based content search with ripgrep performance
- `glob`: Fast file pattern matching with ignore rules
- Both exclude common directories (node_modules, .venv, .git) by default
- Pagination support for large result sets

#### Shell Access
The `bash` tool provides controlled shell access:
- Safe execution with destructive command blocking
- Background process support for dev servers
- Output truncation to prevent context overflow
- Proper handling of exit codes and errors

## Agents Included

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| **explorer** | Codebase reconnaissance | Understanding unfamiliar projects, mapping dependencies |
| **zen-architect** | Design and architecture | Planning major features, refactoring, system design |
| **modular-builder** | Implementation | Building features, writing tests, creating modules |
| **bug-hunter** | Debugging and diagnosis | Investigating failures, tracing errors, fixing bugs |
| **git-ops** | Git operations | Branch management, commits, PR workflows |
| **file-ops** | Bulk file operations | Renaming, moving, restructuring multiple files |
| **ecosystem-expert** | Technology guidance | Learning APIs, frameworks, best practices |
| **integration-specialist** | System integration | Connecting services, APIs, third-party tools |
| **security-guardian** | Security analysis | Reviewing code for vulnerabilities, security best practices |
| **test-coverage** | Test strategy | Analyzing coverage, writing tests, test planning |
| **web-research** | Internet research | Finding documentation, examples, solutions |
| **post-task-cleanup** | Code cleanup | Organizing imports, removing dead code, formatting |
| **session-analyst** | Workflow analysis | Understanding conversation history, extracting insights |

### Agent Details

#### explorer
**Capabilities:**
- Analyzes project structure and technology stack
- Maps dependencies and relationships
- Identifies entry points and core modules
- Generates architecture summaries

**Best For:**
- First-time project exploration
- Onboarding to unfamiliar codebases
- Pre-implementation reconnaissance

#### zen-architect
**Capabilities:**
- Designs system architecture and component boundaries
- Plans implementation strategies
- Creates technical specifications
- Evaluates design tradeoffs

**Best For:**
- Major feature planning
- Refactoring large systems
- API design
- Database schema design

#### modular-builder
**Capabilities:**
- Implements features following best practices
- Creates modular, testable code
- Writes comprehensive tests
- Documents public APIs

**Best For:**
- Feature implementation
- Module creation
- Test development
- Code generation

#### bug-hunter
**Capabilities:**
- Investigates test failures and runtime errors
- Traces execution paths
- Identifies root causes
- Proposes and implements fixes

**Best For:**
- Debugging failing tests
- Investigating production issues
- Performance problems
- Unexpected behavior

#### git-ops
**Capabilities:**
- Manages branches and commits
- Handles merge conflicts
- Creates pull requests
- Reviews git history

**Best For:**
- Branch operations
- Commit management
- Git workflow automation
- History analysis

## Getting Started

### Basic Usage

The Foundation Bundle is active by default. Simply start using Amplifier and all Foundation tools and agents are immediately available.

**Example 1: Reading and Editing Code**
```
You: "Find all TODO comments in the src/ directory"
Amplifier: [Uses grep tool to search for TODO patterns]

You: "Update the authentication logic in auth.py"
Amplifier: [Uses read_file to inspect, then edit_file to modify]
```

**Example 2: Delegating to Agents**
```
You: "I need to understand this legacy codebase"
Amplifier: [Launches explorer agent to analyze structure]

You: "Design a new caching layer for this system"
Amplifier: [Launches zen-architect to create design]

You: "Now implement that design"
Amplifier: [Launches modular-builder to write code]
```

### Agent Workflows

Agents can be chained for complex workflows:

1. **Full Feature Development:**
   - `explorer` → Understand existing code
   - `zen-architect` → Design the feature
   - `modular-builder` → Implement with tests
   - `git-ops` → Commit and create PR

2. **Bug Investigation and Fix:**
   - `bug-hunter` → Diagnose the issue
   - `modular-builder` → Implement fix
   - `test-coverage` → Add regression tests
   - `git-ops` → Create fix PR

3. **Code Quality Improvement:**
   - `session-analyst` → Review recent changes
   - `security-guardian` → Check for vulnerabilities
   - `post-task-cleanup` → Clean up code
   - `test-coverage` → Ensure adequate testing

### Configuration

The Foundation Bundle works with zero configuration, but can be customized via `.amplifier/config.yaml`:

```yaml
bundles:
  foundation:
    enabled: true
    agents:
      # Customize agent behavior
      explorer:
        max_depth: 5
      bug-hunter:
        auto_fix: false
```

## When to Use

### Perfect For:

✅ **General software development** - Most common programming tasks  
✅ **Project exploration** - Understanding new codebases  
✅ **Feature implementation** - Building new functionality  
✅ **Bug fixing** - Debugging and resolving issues  
✅ **Code refactoring** - Improving code structure  
✅ **Git workflows** - Managing version control  
✅ **Test development** - Writing and maintaining tests  
✅ **Documentation** - Creating and updating docs  

### Consider Specialized Bundles For:

🔧 **Python development** - Use `python-dev` bundle for enhanced Python tools  
🎨 **Design work** - Use `design-intelligence` bundle for UI/UX  
⚙️ **DevOps tasks** - Use infrastructure-specific bundles  
📊 **Data science** - Use data-analysis bundles  

### Complementary Bundles

The Foundation Bundle works well alongside:
- **python-dev**: Adds Python-specific linting, type checking, testing tools
- **design-intelligence**: Adds UI/UX design agents and tools
- **recipes**: Adds workflow automation and multi-stage processes

Multiple bundles can be loaded simultaneously, with their tools and agents available together.

## Best Practices

### Tool Selection

**Use specialized tools first:**
- Prefer `read_file` over `bash cat` for reading files
- Use `grep` instead of `bash grep` for searching
- Use `glob` for file discovery before manual searching

**Delegate complex tasks:**
- Launch agents for multi-step workflows
- Use `task` tool when work requires multiple operations
- Let agents operate autonomously instead of micromanaging

### Agent Usage

**Choose the right agent:**
- `explorer` for understanding, not building
- `zen-architect` for design, not implementation
- `modular-builder` for implementation, not debugging
- `bug-hunter` for diagnosis and fixes

**Provide clear instructions:**
```
Good: "Use bug-hunter to investigate why test_auth.py::test_login fails"
Bad: "Fix the tests"
```

### Workflow Patterns

**1. Exploration First**
Before major changes, understand the codebase:
```
explorer → zen-architect → modular-builder
```

**2. Test-Driven Development**
Write tests before implementation:
```
zen-architect → test-coverage → modular-builder
```

**3. Iterative Refinement**
Build, test, improve:
```
modular-builder → test-coverage → post-task-cleanup
```

## Common Patterns

### Pattern: Safe Refactoring

1. **Search** - Find all usages with `grep`
2. **Analyze** - Review with `explorer` or `session-analyst`
3. **Plan** - Design changes with `zen-architect`
4. **Execute** - Implement with `modular-builder`
5. **Verify** - Test with `test-coverage`
6. **Commit** - Save with `git-ops`

### Pattern: New Feature Development

1. **Explore** - Use `explorer` to understand integration points
2. **Design** - Use `zen-architect` to plan architecture
3. **Build** - Use `modular-builder` to implement
4. **Test** - Use `test-coverage` to ensure quality
5. **Document** - Update docs with `write_file`
6. **Review** - Use `security-guardian` for security check
7. **Ship** - Create PR with `git-ops`

### Pattern: Bug Fix Workflow

1. **Reproduce** - Verify the issue with `bash` to run tests
2. **Investigate** - Use `bug-hunter` to diagnose
3. **Fix** - Implement solution (often done by `bug-hunter`)
4. **Test** - Add regression test
5. **Commit** - Save fix with descriptive message

## Try It Yourself

### Exercise 1: Project Exploration

Try exploring a new project:
```
1. Launch explorer agent to analyze project structure
2. Use grep to find all exported functions
3. Use glob to list all test files
4. Read package.json or requirements.txt to understand dependencies
```

### Exercise 2: Feature Implementation

Build a simple feature:
```
1. Use zen-architect to design a new utility function
2. Launch modular-builder to implement it
3. Use test-coverage to ensure it's tested
4. Use post-task-cleanup to format code
5. Use git-ops to commit changes
```

### Exercise 3: Bug Hunt

Practice debugging:
```
1. Run tests with bash to identify failures
2. Launch bug-hunter to investigate
3. Review the proposed fix
4. Run tests again to verify
5. Commit the fix
```

### Exercise 4: Search and Replace

Try refactoring:
```
1. Use grep to find all occurrences of a function name
2. Use read_file to inspect each usage
3. Use edit_file to update the function signature
4. Use bash to run tests
5. Use post-task-cleanup to organize imports
```

## Advanced Usage

### Multi-Agent Workflows

Launch multiple agents in parallel for complex tasks:
```
You: "Audit this codebase for quality and security"
Amplifier: [Launches security-guardian and test-coverage simultaneously]
```

### Custom Agent Instructions

Provide detailed context for better results:
```
You: "Use explorer to map the authentication flow, focusing on JWT token
validation and refresh logic. Document the security boundaries."
```

### Combining Tools and Agents

Mix direct tool usage with agent delegation:
```
1. Use glob to find all API route files
2. Launch explorer to analyze each route
3. Use zen-architect to design improvements
4. Implement changes yourself with edit_file
```

## Integration

### With Other Bundles

Foundation provides the base layer for specialized bundles:
- **python-dev** extends Foundation with Python-specific tools
- **design-intelligence** adds design agents while using Foundation's filesystem tools
- Custom bundles can leverage Foundation's agents in their workflows

### With Recipes

Foundation agents are frequently used in recipes:
```yaml
stages:
  - name: analyze
    agent: foundation:explorer
    instruction: "Analyze project structure"
  
  - name: design
    agent: foundation:zen-architect
    instruction: "Design new feature based on analysis"
  
  - name: implement
    agent: foundation:modular-builder
    instruction: "Implement the designed feature"
```

## Troubleshooting

### Common Issues

**Agent not responding or slow:**
- Check if task is too broad - provide more specific instructions
- Review agent's tool access - ensure needed tools are available

**Search returning too many results:**
- Use more specific regex patterns with `grep`
- Add file type filters with `glob`
- Use `head_limit` parameter to paginate

**File operations failing:**
- Verify file paths are correct
- Check file permissions
- Ensure files exist before editing

### Getting Help

- Use `load_skill` to access domain-specific knowledge
- Launch `ecosystem-expert` for technology questions
- Use `web-research` agent for finding documentation

## What's Next?

- Explore [Python Development Bundle](./python-dev.md) for enhanced Python tools
- Learn about [Design Intelligence Bundle](./design-intelligence.md) for UI/UX work
- Check out [Recipes](../guides/recipes.md) for workflow automation
- Read [Agent Patterns](../guides/agent-patterns.md) for advanced usage

---

The Foundation Bundle provides everything you need for general-purpose development. Master these tools and agents, and you'll be productive in any programming environment.
