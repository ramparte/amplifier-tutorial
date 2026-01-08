# Building the Amplifier Tutorial

Instructions for building, updating, and maintaining this tutorial site.

## Quick Build

```bash
cd /path/to/tutorial
source .venv/bin/activate  # Or create: python -m venv .venv && pip install mkdocs
mkdocs build --clean
```

The site will be in `site/` - open `site/index.html` in a browser.

## Prerequisites

- Python 3.10+
- mkdocs (`pip install mkdocs`)

## Project Structure

```
tutorial/
├── content/           # Markdown source files
│   ├── index.md      # Home page
│   ├── quickstart/   # Getting Started
│   ├── concepts/     # Core Concepts
│   ├── tools/        # Tools Reference
│   ├── bundles/      # Bundles Reference
│   ├── skills/       # Skills Reference
│   ├── dev-setup/    # Developer Setup
│   └── advanced/     # Advanced Topics
├── .notes/           # Editorial notes (not in site)
├── .planning/        # Planning docs (not in site)
├── recipes/          # Build/update recipes
├── mkdocs.yml        # Site configuration
├── site/             # Built output (gitignored)
└── BUILD.md          # This file
```

## Updating Content

### Manual Edits

1. Edit markdown files in `content/`
2. Run `mkdocs build --clean`
3. Test locally

### Adding New Pages

1. Create markdown file in appropriate `content/` subdirectory
2. Add to `nav:` section in `mkdocs.yml`
3. Rebuild

### Using the Update Recipe

```bash
amplifier recipes execute recipes/update-tutorial.yaml \
  --context '{"focus": "tools"}'
```

## Editorial Notes

The `.notes/` directory contains:
- Curated explanations from team members
- User feedback and suggestions
- Content ideas not yet incorporated

When rebuilding, check `.notes/` for pending items marked `[PENDING]`.

## Full Regeneration

To regenerate all content from the Amplifier repos:

```bash
amplifier recipes execute recipes/full-rebuild.yaml
```

This will:
1. Scan microsoft/amplifier and related repos
2. Scan bkrabach and robotdad repos for tools/patterns
3. Generate all content sections
4. Incorporate pending editorial notes
5. Build the site

## Content Sources

The tutorial pulls from:

| Source | What |
|--------|------|
| microsoft/amplifier | Main docs, USER_ONBOARDING.md |
| microsoft/amplifier-core | Kernel philosophy, contracts |
| microsoft/amplifier-foundation | Bundles, examples, patterns |
| bkrabach repos | Setup tools, patterns |
| robotdad repos | Skills (playwright, curl, etc.) |
| .notes/ | Editorial additions |

## Serving Locally

```bash
# Simple Python server
cd site
python -m http.server 8000
# Open http://localhost:8000

# Or with mkdocs (hot reload)
mkdocs serve
```

## Distribution

The site is fully static and self-contained:

```bash
# Zip for sharing
zip -r amplifier-tutorial.zip site/

# The zip can be extracted and browsed locally
```

## Troubleshooting

### Navigation not working
Ensure `use_directory_urls: false` is in mkdocs.yml (allows file:// browsing).

### Missing pages
Check that new pages are listed in `nav:` in mkdocs.yml.

### Build errors
Usually YAML syntax issues - check mkdocs.yml indentation.
