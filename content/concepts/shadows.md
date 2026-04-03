---
id: shadows
type: concept
title: "Shadow Environments"
---

# Shadow Environments

You've made a change to a core library. The unit tests pass. But will the six
downstream packages that depend on it still work? You could push and wait for
CI, but that's a 15-minute feedback loop -- and if it breaks, you've broken
everyone's builds. Shadow environments give you a faster answer: an OS-isolated sandbox where your
local changes become the "real" repository, and you can test the full dependency
chain without pushing anything.

## What Is a Shadow Environment?

A shadow environment is an **isolated container** that sees your local working
tree as if it were the upstream repository. When code inside the shadow runs
`pip install git+https://github.com/microsoft/amplifier-core`, it doesn't
fetch from GitHub -- it fetches from a snapshot of your local checkout. Everything
else resolves normally.

```
Your machine:                  Shadow container:
┌──────────────────┐           ┌──────────────────────────┐
│ ~/repos/core     │──snapshot─▶│ Embedded Gitea server     │
│  (your changes)  │           │   ↕ git URL rewriting     │
└──────────────────┘           │                          │
                               │ pip install git+https://  │
                               │   github.com/ms/core     │
                               │   → resolves to Gitea    │
                               │   → YOUR local code      │
                               │                          │
                               │ Everything else           │
                               │   → real GitHub           │
                               └──────────────────────────┘
```

The key insight: **selective git URL rewriting**. Only the URLs you specify
get redirected. The container has full network access for everything else --
real dependencies, real package registries, real GitHub for repos you aren't
overriding.

## How It Works

Shadow environments use three components working together:

### 1. OS-Level Isolation

On **Linux**, shadows use [bubblewrap](https://github.com/containers/bubblewrap)
(`bwrap`) -- the same sandboxing technology used by Flatpak. On **macOS**, they
use `sandbox-exec` with a custom profile. Both provide:

- Filesystem isolation (the shadow can't modify your host files)
- Network namespace separation
- Dropped capabilities and no-new-privileges flags
- Security-hardened containers by default

### 2. Local Source Snapshots

When you create a shadow, it takes an **exact working tree snapshot** of your
local repo -- including untracked files, uncommitted changes, and deleted files.
This isn't just the latest commit; it's the exact state of your working
directory. The snapshot is pushed to an embedded Gitea server running inside the
container, preserving full git history so pinned commits still resolve.

### 3. Git URL Rewriting

Git is configured inside the container to rewrite *only* the specific GitHub
URLs you specify to point at the embedded Gitea server. When any tool -- pip,
uv, cargo, npm -- resolves a git dependency, the rewriting happens at the git
transport layer. The tools don't know they're hitting a local server.

## Using Shadows

### Create

Create a shadow with one or more local source overrides:

```bash
# Single repo override
amplifier-shadow create --local ~/repos/amplifier-core:microsoft/amplifier-core

# Multiple repos in one shadow
amplifier-shadow create \
  --local ~/repos/amplifier-core:microsoft/amplifier-core \
  --local ~/repos/amplifier-foundation:microsoft/amplifier-foundation \
  --local ~/repos/amplifier-app-cli:microsoft/amplifier-app-cli
```

The `--local` flag maps a local path to a GitHub `org/repo` identifier. Inside
the shadow, any git fetch to `github.com/microsoft/amplifier-core` resolves to
your local snapshot instead.

### Execute

Run commands inside the shadow:

```bash
# Install a package (resolves to your local code)
amplifier-shadow exec <id> "uv pip install git+https://github.com/microsoft/amplifier-core"

# Run Amplifier itself against your changes
amplifier-shadow exec <id> "amplifier run 'Hello from shadow'"

# Run a test suite
amplifier-shadow exec <id> "cd /workspace && pytest tests/"
```

### Inspect and Iterate

```bash
# See what changed inside the shadow
amplifier-shadow diff <id>

# Open an interactive shell
amplifier-shadow shell <id>

# Check shadow status and snapshot commits
amplifier-shadow status <id>

# Inject a file into the running shadow
amplifier-shadow inject <id> local-fix.py /workspace/src/fix.py

# Extract results from the shadow
amplifier-shadow extract <id> /workspace/test-results.xml ./results.xml
```

### Clean Up

```bash
# Destroy a single shadow
amplifier-shadow destroy <id>

# List all active shadows
amplifier-shadow list
```

### Verifying Your Code Is Used

The `create` and `status` commands return the exact commit hash captured from
your local repo. When pip or uv installs a package inside the shadow, the
resolved commit appears in the install output. Match those hashes to confirm
your local code is actually being used -- not a stale cached version.

> I want to verify my local core changes are being picked up in the shadow.

[Tool: bash] amplifier-shadow status shadow-7f3a

## Use Cases

### Testing Core Changes

You've changed the Amplifier kernel's tool dispatch logic. Before pushing, you
want to know if downstream bundles still load correctly:

```bash
amplifier-shadow create --local ~/repos/amplifier-core:microsoft/amplifier-core
amplifier-shadow exec <id> "uv tool install git+https://github.com/microsoft/amplifier"
amplifier-shadow exec <id> "amplifier run 'load foundation bundle and list tools'"
```

If the bundle loads and tools fire, your change is compatible. If not, you
caught the regression before CI did.

### Multi-Repo Integration

You're changing a data model in `amplifier-core` and updating the CLI in
`amplifier-app-cli` to match. These changes must ship together:

```bash
amplifier-shadow create \
  --local ~/repos/amplifier-core:microsoft/amplifier-core \
  --local ~/repos/amplifier-app-cli:microsoft/amplifier-app-cli

amplifier-shadow exec <id> "uv tool install git+https://github.com/microsoft/amplifier"
amplifier-shadow exec <id> "amplifier --version && amplifier run 'smoke test'"
```

Both repos resolve to your local snapshots. You're testing the *combination*
of changes, not each in isolation.

### Destructive Tests

Need to test an uninstall script? A database migration rollback? A filesystem
cleanup routine? Run it in a shadow. The container is disposable -- destroy it
when done and your host is untouched.

## The Testing Ladder

Shadow environments sit in the middle of a testing progression. Each level
catches different kinds of bugs at different costs:

```
Level 1: Unit Tests
  ├─ Scope: Single function or class
  ├─ Speed: Seconds
  └─ Catches: Logic errors, regressions in isolated code

Level 2: Local Override
  ├─ Scope: Your package, editable install
  ├─ Speed: Seconds
  └─ Catches: Integration within your own repo

Level 3: Shadow Environment          ← you are here
  ├─ Scope: Full dependency chain, multi-repo
  ├─ Speed: Minutes
  └─ Catches: Cross-repo breakage, dependency conflicts, install failures

Level 4: Push & CI
  ├─ Scope: Full matrix (OS × Python version × config)
  ├─ Speed: 10-30 minutes
  └─ Catches: Platform-specific issues, environment matrix failures

Level 5: Docker E2E
  ├─ Scope: Production-like environment
  ├─ Speed: 15-45 minutes
  └─ Catches: Deployment issues, config drift, system integration
```

### When to Use Each Level

| Situation | Best Level | Why |
|-----------|-----------|-----|
| Fixed a typo in a docstring | Unit tests (1) | No behavior change to verify |
| Changed a function signature | Unit tests + local override (1-2) | Need to check callers within the repo |
| Changed a public API used by other repos | Shadow (3) | Need to check cross-repo consumers |
| Adding a new dependency | Shadow (3) | Need to verify it installs cleanly in context |
| Multi-repo coordinated change | Shadow (3) | Need to test the combination |
| Ready to merge | Push & CI (4) | Need the full platform matrix |
| Preparing a release | Docker E2E (5) | Need production-like validation |

The key question for shadows: **"Does my change break things outside my repo?"**
If the blast radius is contained to your repository, unit tests and local
overrides are sufficient. If it crosses repo boundaries, reach for a shadow.

## What You've Learned

- Shadow environments are OS-isolated containers that redirect specific git URLs
  to your local working tree snapshots
- They use bubblewrap (Linux) or sandbox-exec (macOS) for security isolation
  and an embedded Gitea server for git URL rewriting
- The `amplifier-shadow` CLI manages the full lifecycle: create, exec, shell,
  diff, inject, extract, destroy
- Shadows excel at cross-repo integration testing -- verifying that your local
  changes work with the full dependency chain before pushing
- They sit at Level 3 of the testing ladder, between local overrides and CI --
  catching cross-repo breakage at the cost of minutes, not the 15+ minutes of
  a full CI pipeline
