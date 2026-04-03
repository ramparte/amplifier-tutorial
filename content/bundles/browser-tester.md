---
id: browser-tester
type: bundle
title: "Browser Tester Bundle"
---

# Browser Tester Bundle

## What is the Browser Tester Bundle?

Some things can't be tested by reading HTML. A single-page app that renders behind a JavaScript framework, a login flow that redirects through OAuth, a responsive layout that shifts at a specific breakpoint — these require a real browser. The `web_fetch` tool gets you static HTML, which is plenty for documentation lookups and API responses. But the moment you need JavaScript rendering, form interaction, or a screenshot, you need something that actually runs Chromium.

The Browser Tester bundle provides browser automation through **agent-browser**, a token-efficient CLI that interacts with pages using accessibility-tree references instead of DOM selectors. Three specialized agents handle distinct use cases: general automation, multi-page research, and visual documentation. You describe what you want in natural language, and the right agent drives a real browser to get it done.

## What's Included

The bundle ships three agents, all powered by the `agent-browser` CLI:

| Component | What It Does |
|-----------|-------------|
| **`browser-operator`** | General automation — navigation, forms, data extraction, screenshots, UX testing |
| **`browser-researcher`** | Multi-page research — exploration, synthesis, documentation lookup |
| **`visual-documenter`** | Screenshot capture — QA evidence, responsive testing, change tracking |

### The Key Concept: Accessibility-Tree Refs

Traditional browser automation uses CSS selectors or XPath — fragile, verbose, and expensive in tokens. agent-browser takes a different approach. It reads the browser's **accessibility tree** and assigns short references like `@e1`, `@e2`, `@e3` to interactive elements. These refs are stable across minor DOM changes, compact enough to fit in agent context, and semantically meaningful because they map to what a screen reader sees, not how the HTML is structured.

When an agent takes a snapshot of a page, it gets back something like:

```
@e1  link       "Home"
@e2  link       "Products"
@e3  textbox    "Search..."
@e4  button     "Search"
@e5  link       "Sign In"
```

To click "Sign In," the agent runs `click @e5`. To type in the search box: `fill @e3 "wireless headphones"`. No CSS selectors, no XPath, no DOM inspection. Just short, stable references.

### Core Commands

| Command | What It Does |
|---------|-------------|
| `open <url>` | Navigate to a URL |
| `snapshot -ic` | Capture the accessibility tree (interactive elements, compact) |
| `click @e1` | Click an element by ref |
| `fill @e2 "text"` | Type text into an input by ref |
| `screenshot` | Capture a PNG of the current viewport |
| `close` | Close the browser session |

## Getting Started

### Prerequisites

agent-browser requires Node.js and a Chromium installation:

```bash
# Install the CLI globally
npm install -g agent-browser

# Install Chromium (managed by agent-browser)
agent-browser install

# On Linux, include system dependencies
agent-browser install --with-deps
```

The `--with-deps` flag installs the shared libraries Chromium needs on Linux (libnss3, libatk, etc.). On macOS and Windows, these come with the OS.

### Your First Browser Interaction

Ask the operator to visit a page and report what it finds:

> Go to https://example.com and tell me what's on the page.

```
[Agent: browser-operator]
  open https://example.com
  snapshot -ic

  Page: "Example Domain"
  @e1  link  "More information..."

  The page contains a heading "Example Domain," a paragraph
  explaining this domain is for use in examples, and a single
  link to IANA's documentation.

  close
```

The agent opened Chromium, loaded the page, read the accessibility tree, summarized the content, and closed the session. You never touched a selector.

## Key Agents

### browser-operator

The general-purpose automation agent. It handles navigation, form filling, data extraction, screenshot capture, and UX testing. Use it when you need to *do something* in a browser — submit a form, walk through a checkout flow, verify that a button leads where it should.

> Test the login flow on our staging site. Use test@example.com / password123.

```
[Agent: browser-operator]
  open https://staging.myapp.com/login
  snapshot -ic

  @e1  textbox   "Email"
  @e2  textbox   "Password"
  @e3  button    "Sign In"

  fill @e1 "test@example.com"
  fill @e2 "password123"
  click @e3
  snapshot -ic

  Page redirected to /dashboard
  @e4  heading   "Welcome back, Test User"
  @e5  link      "Settings"
  @e6  link      "Logout"

  Login flow works. Submitted credentials on /login,
  redirected to /dashboard. Welcome message confirms
  the correct user. Session is active (Logout link present).

  screenshot  → login-success.png
  close
```

The operator filled two fields, clicked a button, verified the redirect, confirmed the welcome message, and captured evidence — all from a single natural-language request.

### browser-researcher

The research specialist. Where the operator executes a specific task, the researcher *explores*. It visits multiple pages, follows links, compares content across sites, and synthesizes what it finds. Use it for competitive research, documentation lookup, or any task where you don't know exactly which page has the answer.

> Research how Stripe, Square, and PayPal price their payment processing for small businesses.

```
[Agent: browser-researcher]
  Visiting pricing pages for each provider...

  open https://stripe.com/pricing → snapshot
  open https://squareup.com/us/en/pricing → snapshot
  open https://www.paypal.com/us/business/accept-payments → snapshot

  Summary:
  - Stripe: 2.9% + $0.30 per transaction (online), no monthly fee
  - Square: 2.6% + $0.10 (in-person), 2.9% + $0.30 (online),
    no monthly fee
  - PayPal: 2.99% + $0.49 (standard), volume discounts available,
    no monthly fee

  All three use a per-transaction model with no monthly minimums.
  Stripe is cheapest for online-only. Square is cheapest for
  in-person. PayPal is most expensive at standard rates but
  offers volume tiers.

  close
```

The researcher visited three sites, extracted structured pricing data from JavaScript-rendered pages, and produced a comparative summary. This is the kind of task where `web_fetch` would fail — pricing pages almost always render client-side.

### visual-documenter

The screenshot specialist. It captures visual evidence systematically — at specific viewport widths for responsive testing, before and after changes for regression tracking, or across multiple pages for QA sign-off. Use it when the visual state of the page is the deliverable.

> Capture our homepage at mobile (375px), tablet (768px), and desktop (1440px) widths.

```
[Agent: visual-documenter]
  open https://myapp.com

  Viewport: 375px
  screenshot → homepage-375.png

  Viewport: 768px
  screenshot → homepage-768.png

  Viewport: 1440px
  screenshot → homepage-1440.png

  Three screenshots captured:
  - 375px: Mobile layout, hamburger menu, stacked cards
  - 768px: Tablet layout, 2-column grid, condensed nav
  - 1440px: Desktop layout, full nav bar, 3-column grid

  close
```

## When to Use Browser Agents vs. web_fetch

The two approaches serve different purposes. Choosing wrong wastes time — browser agents are slower and heavier than `web_fetch`, but `web_fetch` can't do what they can.

| Scenario | Use |
|----------|-----|
| Read a static HTML page or API docs | `web_fetch` |
| Fetch a JSON API response | `web_fetch` |
| Download a file from a known URL | `web_fetch` |
| Page renders with JavaScript (React, Vue, etc.) | Browser agent |
| Fill and submit a form | Browser agent |
| Test a multi-step flow (login, checkout) | Browser agent |
| Capture a screenshot | Browser agent |
| Compare visual layouts across breakpoints | Browser agent |
| Research across multiple JS-rendered sites | Browser agent |

The rule of thumb: if the content you need is in the raw HTML response, use `web_fetch`. If you need to *run* the page — execute JavaScript, interact with elements, or see what it looks like — use a browser agent.

## Tips

- **Start with `snapshot -ic`.** The `-ic` flags mean *interactive* and *compact* — you get only the elements you can act on, in the fewest tokens. Omit `-ic` only when you need the full page structure for analysis.

- **Use the right agent.** Don't ask `browser-operator` to research across five sites — that's `browser-researcher`. Don't ask `browser-researcher` to fill out a form — that's `browser-operator`. Don't ask either to capture responsive screenshots — that's `visual-documenter`.

- **Close sessions.** Browser sessions consume resources. Agents close sessions automatically when done, but if you're debugging interactively, remember to close.

- **Prefer `web_fetch` when possible.** Browser agents launch Chromium, which is slower and heavier. If you just need the text content of a static page, `web_fetch` gets it in a fraction of the time.

- **Refs change between snapshots.** After navigation or page changes, take a new `snapshot -ic` to get fresh refs. The `@e1` from before a form submission won't be the same `@e1` after the page reloads.

- **Screenshots are your proof.** When testing UI flows, always capture a screenshot at the critical moment. Screenshots provide evidence that a test passed — a text summary alone doesn't prove the layout was correct.

## Next Steps

- Learn about the [Web Tools](../tools/web.md) for `web_fetch` and `web_search`
- Explore the [Foundation Bundle](./foundation.md) for the core tools all bundles build on
- See [Agents](../concepts/agents.md) for how agent delegation works
- Read about [Recipes](./recipes.md) for automating multi-step browser workflows
