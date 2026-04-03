---
id: todo-tool
type: tool
title: "Todo Tool"
---

# Todo Tool

When Amplifier is working through a complex, multi-step task, you might wonder: where are we? What's done? What's left? The todo tool answers those questions. It lets Amplifier create a visible checklist that tracks progress through multi-step work — and you see it updating in real time in your UI.

This isn't a background mechanism. The todo list appears directly in your interface, so you can watch items move from pending to in-progress to completed as Amplifier works. It's your window into what's happening during long-running tasks, and it's used *constantly*. Any time Amplifier is planning or executing multi-step work, the todo tool should be involved.

## Core Capabilities

The todo tool has three operations:

- **create** — build a new todo list, replacing any existing one
- **update** — replace the entire list with a new version (used to change statuses)
- **list** — read the current todo list without changing it

Each todo item has three fields:

- **content** — an imperative description of the task: "Run tests", "Build project", "Update imports"
- **activeForm** — the present continuous version: "Running tests", "Building project", "Updating imports"
- **status** — one of `pending`, `in_progress`, or `completed`

The `content` field describes what to do. The `activeForm` field is what you see in the UI while that step is active — it reads naturally as a status indicator. And the `status` field tracks where things stand.

## How It Works

When Amplifier starts complex work, it creates a todo list up front with all the steps planned out. Then, as it works through each step, it updates the list — marking the current step as `in_progress` and completed steps as `completed`. The key discipline: exactly one item should be `in_progress` at any time.

Here's what that looks like. Say you ask:

> Add a rate limiter to the API endpoints

Amplifier starts by creating the plan:

```
[Tool: todo] create
  - "Research existing rate limiting patterns"     pending
  - "Create rate limiter middleware"               pending
  - "Add configuration options"                    pending
  - "Apply middleware to API routes"               pending
  - "Write tests for rate limiting"                pending
  - "Run tests and verify"                         pending
```

Your UI immediately shows six items, all pending. Then Amplifier begins work and updates the list as it goes:

```
[Tool: todo] update
  - "Research existing rate limiting patterns"     completed
  - "Create rate limiter middleware"               in_progress
  - "Add configuration options"                    pending
  - "Apply middleware to API routes"               pending
  - "Write tests for rate limiting"                pending
  - "Run tests and verify"                         pending
```

You can see at a glance: research is done, the middleware is being created now, four steps remain. This continues through each step until everything is completed.

## A Full Example

Let's walk through a realistic scenario to see the todo tool in action throughout an entire task.

> Add user avatar support — upload, storage, and display

First, Amplifier plans the work:

```
[Tool: todo] create
  - "Explore current user model and file handling"     pending
  - "Add avatar field to user model"                   pending
  - "Create avatar upload endpoint"                    pending
  - "Add file validation and resizing"                 pending
  - "Update user profile response to include avatar"   pending
  - "Write tests"                                      pending
  - "Run tests and verify"                             pending
```

Amplifier starts exploring the codebase:

```
[Tool: todo] update
  - "Explore current user model and file handling"     in_progress
  - "Add avatar field to user model"                   pending
  ...

[Tool: read_file] src/models/user.py
[Tool: grep] "upload" --type py
[Tool: read_file] src/utils/storage.py
```

After exploring, it moves on to the model change:

```
[Tool: todo] update
  - "Explore current user model and file handling"     completed
  - "Add avatar field to user model"                   in_progress
  - "Create avatar upload endpoint"                    pending
  ...

[Tool: edit_file] src/models/user.py
  + avatar_url: Optional[str] = None
  + avatar_updated_at: Optional[datetime] = None
```

Then the upload endpoint:

```
[Tool: todo] update
  - "Explore current user model and file handling"     completed
  - "Add avatar field to user model"                   completed
  - "Create avatar upload endpoint"                    in_progress
  ...

[Tool: write_file] src/routes/avatar.py
[Tool: edit_file] src/routes/__init__.py
```

This continues step by step. By the time tests run, the list looks like:

```
[Tool: todo] update
  - "Explore current user model and file handling"     completed
  - "Add avatar field to user model"                   completed
  - "Create avatar upload endpoint"                    completed
  - "Add file validation and resizing"                 completed
  - "Update user profile response to include avatar"   completed
  - "Write tests"                                      completed
  - "Run tests and verify"                             in_progress

[Tool: bash] pytest tests/test_avatar.py --verbose
===== 8 passed in 2.1s =====
```

And finally:

```
[Tool: todo] update
  - "Explore current user model and file handling"     completed
  - "Add avatar field to user model"                   completed
  - "Create avatar upload endpoint"                    completed
  - "Add file validation and resizing"                 completed
  - "Update user profile response to include avatar"   completed
  - "Write tests"                                      completed
  - "Run tests and verify"                             completed
```

All done. Throughout the entire process, you could see exactly where things stood without asking.

## When to Use It

The todo tool should appear any time work involves more than a couple of steps. Here's the decision:

| Situation | Use todo? | Why |
|-----------|-----------|-----|
| "Fix this typo" | No | One step, nothing to track |
| "Add a new API endpoint" | **Yes** | Multiple steps: model, route, tests |
| "Refactor the auth module" | **Yes** | Research, plan, implement, verify |
| "What does this function do?" | No | Single investigation, immediate answer |
| "Set up CI/CD for the project" | **Yes** | Many discrete steps across config files |
| "Run the tests" | No | Single command |

The rule: if you're planning, you should be using the todo tool. It keeps both you and Amplifier honest about what's been done and what remains.

## Checking Progress

You can always ask about the current state:

> Where are we on that refactor?

```
[Tool: todo] list
  - "Extract shared utilities"                completed
  - "Update import paths"                     completed
  - "Refactor database layer"                 in_progress
  - "Update API handlers"                     pending
  - "Run full test suite"                     pending
```

The list operation reads without modifying — useful when you want a status check mid-task or when resuming after a break.

## Tips

- **Create the list early.** The best time to create a todo list is right when work starts — before any files are read or commands are run. Planning up front gives you visibility from the very beginning.
- **Keep exactly one item `in_progress`.** This is the most important discipline. Zero in-progress items means you can't tell what's happening. Two means the list is inaccurate. One gives you a clear signal.
- **Mark items completed immediately.** Don't batch status updates. As soon as a step finishes, update the list before moving on. This keeps the UI accurate in real time.
- **Use imperative content, continuous activeForm.** "Run tests" and "Running tests" is the pattern. The content says what to do; the activeForm says what's happening right now.
- **Update replaces the whole list.** There's no "update item 3" operation — you send the full list each time. This keeps things simple and means the list is always a complete, consistent snapshot.
- **Don't over-plan.** Five to eight items is the sweet spot. Two items isn't worth tracking. Fifteen items means your steps are too granular — group related work into meaningful chunks.
- **Adjust the plan as you go.** If you discover a step needs to be split, or a planned step turns out to be unnecessary, update the list. The todo list is a living plan, not a contract.

## Next Steps

- See the [Bash Tool](./bash.md) for running the test commands that often appear as todo items
- Learn about [Delegate Tool](./task.md) for breaking large tasks into parallel sub-agents — each with their own todo tracking
- Read [Recipes](./recipes-tool.md) for declarative multi-step workflows that formalize sequential execution
