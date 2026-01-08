# Amplifier Tutorial

A narrative tutorial for learning [Amplifier](https://github.com/microsoft/amplifier), the modular AI agent framework.

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally (with hot reload)
mkdocs serve

# Build static site
mkdocs build

# The site/ directory can be:
# - Opened directly (site/index.html)
# - Zipped and shared
# - Deployed to any static host
```

## Structure

```
tutorial/
├── content/          # Markdown source files
├── recipes/          # Amplifier recipes for content generation
├── site/             # Generated static site (after build)
├── .planning/        # Planning documents and manifests
├── mkdocs.yml        # Site configuration
└── requirements.txt  # Python dependencies
```

## Updating Content

The tutorial can be updated using Amplifier recipes:

```bash
# Scan ecosystem for new components
amplifier recipes execute recipes/scan-ecosystem.yaml

# Refresh all content from source repos
amplifier recipes execute recipes/refresh-tutorial.yaml

# Regenerate a single component
amplifier recipes execute recipes/research-component.yaml \
  --context '{"component_id": "tool-filesystem", "component_type": "tools"}'
```

## Content Approach

1. **Writer agent** creates content from source READMEs
2. **Reviewer agent** validates against source for accuracy
3. **Human review** for final approval

All content includes copy-pasteable code snippets for hands-on learning.
