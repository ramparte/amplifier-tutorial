---
id: tool-lsp
type: tool
title: "LSP Tool"
---

# LSP Tool (Code Intelligence)

## What Is LSP?

The [Search Tools](./search.md) find text patterns. The LSP tool understands what the text *means*. LSP — Language Server Protocol — connects Amplifier to a language server that has parsed your code into a full semantic model: types, definitions, call chains, references. When you ask "who calls this function?" or "what type does this variable have?", LSP gives you the precise, compiler-grade answer that grep can only approximate.

Amplifier currently configures LSP for **Python** via the Pyright language server. All operations use 1-based line and character positions — line 1 is the first line, character 1 is the first column.

## Core Capabilities

LSP operations fall into three groups: navigation, verification, and refactoring.

### Navigation

**goToDefinition** — jump to where a symbol is defined:

```
> Where is the process_data function defined?

[Tool: LSP] goToDefinition  file: src/api/handlers.py  line: 12  character: 15
  src/core/pipeline.py  line 45, character 1
```

**findReferences** — find every place a symbol is used:

```
> Show me everywhere that validate_input is referenced

[Tool: LSP] findReferences  file: src/utils.py  line: 8  character: 5
  src/api/handlers.py    line 22, character 12
  src/api/middleware.py   line 45, character 8
  tests/test_utils.py    line 15, character 20
  (3 references)
```

**hover** — get type information, signatures, and docstrings:

```
> What type is the result variable on line 30?

[Tool: LSP] hover  file: src/main.py  line: 30  character: 5
  (variable) result: Dict[str, List[Item]]
```

**documentSymbol** — list all symbols defined in a file:

```
> What's defined in src/models.py?

[Tool: LSP] documentSymbol  file: src/models.py
  class User (line 8)
  class Order (line 34)
  function create_user (line 62)
  function validate_order (line 78)
```

**prepareCallHierarchy, incomingCalls, outgoingCalls** — trace the call graph in either direction. `prepareCallHierarchy` sets up the target, then `incomingCalls` shows who calls it and `outgoingCalls` shows what it calls:

```
> Who calls the authenticate function?

[Tool: LSP] incomingCalls  file: src/auth.py  line: 15  character: 5
  login_handler        src/api/routes.py     line 42
  refresh_token        src/api/routes.py     line 67
  middleware_check      src/middleware.py      line 23
```

```
> What does authenticate call internally?

[Tool: LSP] outgoingCalls  file: src/auth.py  line: 15  character: 5
  verify_password      src/auth.py           line 44
  load_user            src/models.py         line 18
  create_session       src/sessions.py       line 12
```

### Verification

**diagnostics** — get compiler errors and warnings for a file. This is especially valuable *after editing code* to confirm you haven't introduced problems:

```
> Check src/main.py for errors after that refactor

[Tool: LSP] diagnostics  file: src/main.py
  line 42, character 15: error — Argument of type "str" cannot be assigned to parameter "count" of type "int"
  line 67, character 8: warning — Variable "result" is not accessed
```

Run diagnostics after every significant edit. It catches type mismatches, missing imports, and undefined names before you ever run the code.

### Refactoring

**rename** — semantic cross-file rename. Returns the edits needed but doesn't apply them, so you can review first:

```
> Rename process_data to transform_records everywhere

[Tool: LSP] rename  file: src/core/pipeline.py  line: 45  character: 5  newName: transform_records
  src/core/pipeline.py      line 45: process_data → transform_records
  src/api/handlers.py       line 22: process_data → transform_records
  tests/test_pipeline.py    line 8:  process_data → transform_records
  (3 files, 7 changes)
```

This is semantically aware — it only renames actual references to that symbol, not string matches in comments or unrelated variables that happen to share the name.

## The Investigation Workflow

The most common LSP pattern is a three-step investigation: **hover → findReferences → incomingCalls**. Here's how it works in practice:

```
> I need to understand the calculate_total function before I change it

[Tool: LSP] hover  file: src/billing.py  line: 23  character: 5
  (function) calculate_total(items: List[LineItem], tax_rate: float = 0.0) -> Decimal
  """Calculate the total cost including optional tax."""
```

Now you know the signature. Next, see how widely it's used:

```
[Tool: LSP] findReferences  file: src/billing.py  line: 23  character: 5
  src/api/checkout.py    line 45
  src/api/invoices.py    line 78
  src/reports/monthly.py line 112
  tests/test_billing.py  line 15, 28, 42
  (6 references)
```

Six references — not trivial. Check the actual callers to understand the dependency chain:

```
[Tool: LSP] incomingCalls  file: src/billing.py  line: 23  character: 5
  process_checkout     src/api/checkout.py     line 40
  generate_invoice     src/api/invoices.py     line 72
  build_monthly_report src/reports/monthly.py  line 108
```

Three callers across three modules. Now you know exactly what will be affected before you touch anything.

## LSP vs grep

This is the key decision. Use LSP when you need *meaning*, grep when you need *text*.

| Task | Best Tool | Why |
|------|-----------|-----|
| Find callers of a function | **LSP** `incomingCalls` | Semantic — skips comments and strings |
| Jump to a definition | **LSP** `goToDefinition` | Goes to the exact spot |
| Get type info or signature | **LSP** `hover` | grep can't infer types |
| Rename a symbol safely | **LSP** `rename` | Cross-file, semantically aware |
| Check for errors after edits | **LSP** `diagnostics` | Compiler-grade validation |
| Find a text pattern anywhere | **grep** | Faster, works on any file |
| Search comments or strings | **grep** | LSP ignores non-code content |
| Search non-code files | **grep** | LSP only covers source code |
| Bulk search across many files | **grep** | Text search is grep's strength |

They also combine well. Use grep to find candidate files fast, then LSP to understand the semantic relationships within them.

## Delegating Complex Navigation

For multi-step investigations that require chaining many LSP operations — tracing a call chain across five files, mapping out a class hierarchy, or building a dependency graph — delegate to the **python-dev:code-intel** agent rather than issuing operations one at a time. The agent handles the chaining, backtracking, and synthesis for you.

```
> Trace the full call chain from the /checkout API endpoint down to the database layer

[delegates to python-dev:code-intel agent]
```

Use direct LSP calls for quick, focused queries. Delegate when the investigation needs reasoning across many hops.

## Tips

**Run diagnostics after every edit.** It's the fastest way to catch type errors, missing imports, and broken references before running tests. Make it a habit.

**Start with hover.** Before diving into definitions or references, hover tells you the type, signature, and docstring in one call. Often that's all you need.

**Positions are 1-based.** Line 1, character 1 is the top-left corner of the file. This matches what you see in `read_file` output with line numbers.

**Fall back to grep when LSP can't help.** Dynamic imports, metaprogramming, and string-based lookups are invisible to the language server. If LSP returns nothing, grep will still find the text.

**Use rename instead of find-and-replace.** `rename` understands scope — it won't accidentally rename a local variable that happens to share a name with the function you're targeting.

## Next Steps

- Learn about [Search Tools](./search.md) for text-based search — the complement to LSP's semantic analysis
- See [Filesystem Tools](./filesystem.md) for reading and editing the files LSP points you to
- Explore [Bash Tool](./bash.md) for running tests after LSP-guided refactoring
