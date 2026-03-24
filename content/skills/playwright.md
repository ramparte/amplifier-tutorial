---
id: skill-playwright
type: skills
title: "Playwright Skill"
---

# Playwright Skill

Browser automation for testing, validation, and web interaction.

## Overview

The Playwright skill teaches Amplifier how to:

- Navigate websites
- Fill forms and click elements
- Extract data from pages
- Capture screenshots
- Run headless (no visible browser)

## Loading the Skill

```
> Load the playwright skill
```

Once loaded, Amplifier knows browser automation best practices.

## When to Use

| Task | Use Playwright |
|------|----------------|
| Test a login flow | ✅ |
| Capture page screenshot | ✅ |
| Fill and submit forms | ✅ |
| Extract data from pages | ✅ |
| Test REST API | ❌ Use curl |
| Fetch static content | ❌ Use web_fetch |

## Core Workflow

The skill teaches this decision flow:

```
START
├── Check if Playwright installed
│   └── No → Run setup
├── Determine task type
│   ├── Screenshot → Capture flow
│   ├── Form fill → Interaction flow
│   ├── Scraping → Extraction flow
│   └── Testing → Validation flow
└── Execute with best practices
    ├── Headless mode (default)
    ├── Role-based selectors
    ├── Explicit waits
    └── Diagnostic capture
```

## Key Patterns

### Headless First

Always run without visible browser:

```python
browser = playwright.chromium.launch(headless=True)
```

This ensures:
- No focus stealing
- Works in CI/CD
- Faster execution

### Role-Based Selectors

Prefer accessibility selectors:

```python
# Good - role-based
page.get_by_role("button", name="Submit")
page.get_by_label("Email")
page.get_by_placeholder("Enter password")

# Avoid - fragile
page.locator(".btn-primary")
page.locator("#email-input")
```

### Explicit Waits

Wait for specific conditions:

```python
# Wait for element
page.wait_for_selector("[data-loaded='true']")

# Wait for navigation
page.wait_for_url("**/dashboard")

# Wait for network idle
page.wait_for_load_state("networkidle")
```

### Diagnostic Capture

On failure, capture context:

```python
try:
    # Test steps
except Exception as e:
    page.screenshot(path="failure.png")
    print(page.content())
    raise
```

## Examples

### Take Screenshot

```
> Take a screenshot of https://example.com
```

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")
    page.screenshot(path="screenshot.png")
    browser.close()
```

### Login Flow

```
> Test logging into the admin panel
```

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    
    # Navigate
    page.goto("https://example.com/login")
    
    # Fill form
    page.get_by_label("Email").fill("admin@example.com")
    page.get_by_label("Password").fill("password123")
    
    # Submit
    page.get_by_role("button", name="Sign in").click()
    
    # Verify success
    page.wait_for_url("**/dashboard")
    assert "Dashboard" in page.title()
    
    browser.close()
```

### Extract Data

```
> Get all product prices from the catalog page
```

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com/products")
    
    # Extract prices
    prices = page.locator(".product-price").all_text_contents()
    
    for price in prices:
        print(price)
    
    browser.close()
```

## Setup

If Playwright isn't installed:

```bash
# Install package
pip install playwright

# Install browsers
playwright install chromium
```

## Troubleshooting

### "Browser not found"

```bash
playwright install chromium
```

### "Element not found"

- Check selector syntax
- Add explicit wait
- Verify element exists in page

### "Timeout"

```python
# Increase timeout
page.wait_for_selector(".element", timeout=10000)  # 10 seconds
```

### Debugging

Run with visible browser:

```python
browser = p.chromium.launch(headless=False, slow_mo=500)
```

## Anti-patterns

The skill warns against:

- **Visible browser mode** - Use headless
- **Arbitrary sleeps** - Use explicit waits
- **CSS selectors first** - Use role-based
- **No error capture** - Always capture diagnostics

## Try It Yourself

### Exercise 1: Screenshot

```
> Load playwright skill
> Take a screenshot of https://github.com
```

### Exercise 2: Form Test

```
> Test the search functionality on https://duckduckgo.com
```

### Exercise 3: Data Extraction

```
> Extract all headlines from https://news.ycombinator.com
```

## Source

```
robotdad/skills/playwright/
├── SKILL.md           # Core workflow
├── patterns.md        # Advanced patterns
├── setup.md           # Installation
└── troubleshooting.md # Common issues
```
