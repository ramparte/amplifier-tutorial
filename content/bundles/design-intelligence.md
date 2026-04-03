---
id: design-intelligence
type: bundle
title: "Design Intelligence Bundle"
---

# Design Intelligence Bundle

## What is the Design Intelligence Bundle?

Most AI assistants treat design as a single skill. Ask for a button, get a button. Ask for a color palette, get a palette. But real design work is collaborative — art directors set vision, system architects build foundations, component designers sweat the details, and copywriters shape every word users read.

The Design Intelligence bundle mirrors that reality. Instead of one generalist, you get a team of seven specialized agents, each an expert in their domain, all sharing a common philosophy called **Amplified Design**. This philosophy is built on the **Nine Dimensions** of design quality and the **Five Pillars** of design excellence, establishing a quality baseline of **9.5/10** — meaning no placeholders, no generic solutions, and every decision justified.

When you ask for a dashboard layout, the right specialist handles it. When the work crosses domains, agents hand off to each other naturally. The result is design guidance that feels like it came from a senior design team, not a chatbot.

## What's Included

Seven agents, each owning specific dimensions of the design process:

| Agent | Focus | Dimensions Owned |
|-------|-------|-----------------|
| `design-system-architect` | Tokens, scales, foundations | Color, Type, Space |
| `component-designer` | UI components, variants, states | Touch, State |
| `layout-architect` | Page structure, information flow | Space, Flow |
| `responsive-strategist` | Breakpoints, device adaptation | Touch, Space |
| `animation-choreographer` | Motion, transitions, timing | Motion, State |
| `art-director` | Aesthetic vision, brand expression | Style, Color |
| `voice-strategist` | UX writing, tone, microcopy | Voice |

These agents share two frameworks that keep their work coherent:

**The Nine Dimensions** — Style, Color, Voice, Space, Motion, Type, Touch, State, and Flow. Every design decision is evaluated against the dimensions it touches.

**The Five Pillars** — Clarity, Consistency, Accessibility, Performance, and Delight. These establish non-negotiable quality standards across all agents.

## Getting Started

The bundle is included in your Amplifier configuration:

```yaml
# .amplifier/config.yaml
bundles:
  - foundation
  - design-intelligence
```

Once loaded, you can address design needs naturally. Amplifier routes to the right specialist, or you can invoke agents directly.

> I'm building a project management tool. Help me establish the visual direction.

`[Tool: art-director]` The art director analyzes your domain, audience, and constraints, then produces an aesthetic guide covering color direction, typography approach, and visual principles — not a mood board, but actionable design decisions.

> Now turn that direction into a design token system.

`[Tool: design-system-architect]` The system architect translates aesthetic intent into concrete foundations: color scales with semantic mappings, a type scale with responsive sizes, and a spacing system with named tokens.

## Key Agents

### design-system-architect

The foundation layer. This agent creates the tokens, scales, and systematic patterns that every other agent builds on. Start here when beginning a new design system or when inconsistency creeps into an existing one.

> Create a design token system for a dark-mode-first developer tool.

`[Tool: design-system-architect]` Produces color primitives, semantic color tokens (surface, foreground, accent, danger), type scale, spacing scale, and elevation tokens — all evaluated against the Color, Type, and Space dimensions.

### component-designer

Works at the individual element level. Buttons, cards, inputs, modals — this agent designs them with full variant coverage, state handling, and accessibility baked in.

> Design a notification badge component with unread counts.

`[Tool: component-designer]` Delivers a complete spec: variants (dot, count, overflow at 99+), states (default, attention, muted), size options, color mappings to your token system, and ARIA attributes.

### layout-architect

Handles page-level structure and information hierarchy. This agent determines how content flows and how users navigate through your application.

> Design the layout for an analytics dashboard with sidebar navigation.

`[Tool: layout-architect]` Produces a spatial composition: sidebar width and collapse behavior, header structure, main content grid, widget placement hierarchy, and content flow patterns optimized for scanning.

### responsive-strategist

Ensures designs work across every device and input method. This agent doesn't just shrink things — it rethinks the experience at each breakpoint.

> Our dashboard has too much data for mobile screens.

`[Tool: responsive-strategist]` Analyzes content priority, then designs a progressive disclosure strategy: what collapses, what hides behind tabs, what reorganizes into cards, and what gets a dedicated drill-down view — with specific breakpoint definitions.

### animation-choreographer

Treats motion as communication. Every transition carries meaning, every micro-interaction provides feedback, and timing is deliberate.

> The save button feels static. Add meaningful feedback.

`[Tool: animation-choreographer]` Designs a micro-interaction sequence: button press scale (95%, 80ms ease-out), processing spinner (fade-in at 300ms delay), success checkmark (draw-on, 400ms), and return to rest — with `prefers-reduced-motion` fallbacks.

### art-director

Sets the aesthetic vision that all other agents follow. This agent transforms abstract feelings ("premium but approachable") into systematic design principles.

> I want our meditation app to feel calming but not new-age, minimal but not cold.

`[Tool: art-director]` Creates an aesthetic guide: warm neutrals with a single muted accent, generous whitespace, rounded-but-not-bubbly border radii, serif headings for warmth, and photography direction favoring natural light and texture.

### voice-strategist

Every word in your interface shapes the user experience. This agent crafts copy that's clear, helpful, and consistent with your product's personality.

> Write error messages for a payment form.

`[Tool: voice-strategist]` Produces contextual messages: "This card number doesn't look right — check for typos" (validation), "Your card has expired — try a different one" (expired), "We couldn't reach your bank — please try again in a moment" (network) — each with tone rationale and an action the user can take.

## Practical Example: Designing a Dashboard

Here's how multiple agents collaborate on a real task. You're building an analytics dashboard from scratch.

**Step 1 — Vision**

> Set the aesthetic direction for a B2B analytics dashboard. It should feel authoritative but not intimidating.

`[Tool: art-director]` Establishes: cool blue-gray palette, data-ink-forward design, restrained use of color for meaning, and a professional sans-serif type system.

**Step 2 — Foundations**

> Build a token system from that direction.

`[Tool: design-system-architect]` Creates tokens: `color.surface.primary`, `color.data.series-1` through `series-6`, `type.scale.data-label`, `space.card-padding`, and so on — a complete design language.

**Step 3 — Structure**

> Design the dashboard layout with a collapsible sidebar and a header with search.

`[Tool: layout-architect]` Defines the spatial grid: 240px sidebar (collapsible to 56px icon rail), sticky 56px header, main area with 24px-gap CSS grid for widget cards, and a defined content hierarchy.

**Step 4 — Components**

> Design the metric card component for KPI display.

`[Tool: component-designer]` Specs the card: label, primary value, trend indicator (up/down/flat with color), sparkline slot, comparison period text, and three size variants (compact, standard, expanded).

**Step 5 — Responsiveness**

> Make this dashboard work on tablets and phones.

`[Tool: responsive-strategist]` Defines three tiers: desktop (full layout), tablet (sidebar collapses to icon rail, grid drops to 2 columns), mobile (sidebar becomes bottom nav, single-column stack, KPI cards become a horizontal scroll).

**Step 6 — Motion**

> Add transitions for the dashboard — loading states, widget appearance, sidebar collapse.

`[Tool: animation-choreographer]` Choreographs: skeleton shimmer for loading, staggered fade-up for widget cards (50ms offset each), sidebar collapse with width transition (200ms ease-in-out), and chart draw-on animations.

**Step 7 — Copy**

> Write the empty states and onboarding copy for the dashboard.

`[Tool: voice-strategist]` Crafts: "No data yet — connect a data source to see your metrics here" (empty state), "Welcome to your dashboard. Let's set up your first widget." (onboarding), with a friendly-professional tone throughout.

## Tips

**Start with art-director and design-system-architect.** These two set the foundation that all other agents build on. Skipping them is like building without blueprints.

**Let agents own their domains.** Ask `responsive-strategist` about breakpoints, not `component-designer`. Each agent's Nine Dimensions expertise makes them the right specialist for their area.

**Chain agents for complete solutions.** The dashboard example above shows the natural flow: vision, foundations, structure, components, responsiveness, motion, copy. Each step builds on the previous.

**Trust the 9.5/10 baseline.** These agents don't produce rough sketches. Recommendations are production-ready, with accessibility considered, edge cases addressed, and implementation guidance included.

**Use the Nine Dimensions as a review checklist.** After any design pass, ask: "How does this score across all nine dimensions?" Agents will self-assess and identify gaps.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core development agents
- Explore [Recipes](./recipes.md) to automate multi-agent design workflows
- Read about [Custom Bundles](../advanced/custom-bundle.md) to extend Design Intelligence
- See [Agents](../concepts/agents.md) for how agent delegation works under the hood
