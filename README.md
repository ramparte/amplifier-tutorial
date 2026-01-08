# Amplifier Tutorial

A narrative tutorial for learning [Amplifier](https://github.com/microsoft/amplifier), the modular AI agent framework.

## View the Tutorial

**The built tutorial is included in this repo** - no build step required!

### Option 1: Browse Locally
```bash
# Clone and open
git clone https://github.com/ramparte/amplifier-tutorial.git
cd amplifier-tutorial

# Open in browser (Windows)
explorer.exe site/index.html

# Open in browser (Mac)
open site/index.html

# Open in browser (Linux)
xdg-open site/index.html
```

### Option 2: Simple HTTP Server
```bash
cd site
python -m http.server 8000
# Open http://localhost:8000
```

### Option 3: Download as ZIP
Download the repo as a ZIP, extract, and open `site/index.html`.

---

## What's Covered

| Section | Topics |
|---------|--------|
| **Getting Started** | What is Amplifier, installation, first conversation, key commands |
| **Core Concepts** | Architecture deep dive, modules, bundles, agents, recipes, skills, hooks |
| **Tools Reference** | Filesystem, bash, search, web, task (sub-agents), LSP, recipes |
| **Bundles Reference** | Foundation, recipes, LSP-Python, design-intelligence |
| **Skills Reference** | Playwright, curl, image-vision |
| **Developer Setup** | CLI tools, shadow workspace, remote dev, debugging |
| **Advanced** | Custom bundles, custom tools, custom recipes, MCP integration |

---

## Regenerating the Tutorial

The tutorial can be rebuilt from the Amplifier source repos.

### Quick Rebuild (content unchanged)
```bash
pip install mkdocs
mkdocs build --clean
```

### Full Regeneration (pull latest from repos)
```bash
# Using Amplifier recipes
amp recipes execute recipes/full-rebuild.yaml

# Or update specific sections
amp recipes execute recipes/update-tutorial.yaml --context '{"focus": "tools"}'
```

### Manual Content Edits
1. Edit markdown files in `content/`
2. Run `mkdocs build --clean`
3. Commit both `content/` and `site/` changes

---

## Project Structure

```
amplifier-tutorial/
├── site/             # ✅ Built tutorial (ready to view!)
├── content/          # Markdown source files
├── .notes/           # Editorial additions (not auto-generated)
├── .planning/        # Planning documents
├── recipes/          # Amplifier recipes for rebuilding
├── mkdocs.yml        # Site configuration
├── BUILD.md          # Detailed build instructions
└── requirements.txt  # Python dependencies (just mkdocs)
```

---

## Contributing

### Add Editorial Content
Drop notes in `.notes/` with `[PENDING]` status. They'll be incorporated on the next rebuild.

### Fix Errors
Edit markdown in `content/`, rebuild, and submit PR.

### Update from Source
Run the update recipes to pull latest from Amplifier repos, review changes, and submit PR.

---

## Content Approach

1. **Generated from source** - Content derives from official Amplifier documentation
2. **Writer/reviewer process** - Separate agents create and validate content
3. **Editorial additions** - Team insights added via `.notes/` system
4. **Human review** - Final approval before merge

All content includes copy-pasteable code snippets for hands-on learning.
