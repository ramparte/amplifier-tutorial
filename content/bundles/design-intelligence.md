---
id: design-intelligence
type: bundles
title: "Design Intelligence Bundle"
---

# Design Intelligence Bundle

## Overview

The Design Intelligence bundle provides a suite of specialized AI agents for UI/UX design work. These agents embody the **Nine Dimensions** philosophy and **Five Pillars** framework, ensuring consistent, high-quality design decisions across your entire system.

Each agent operates at a specific level of the design hierarchy—from strategic aesthetic direction down to individual component refinement—allowing you to engage the right expertise for each design challenge.

## Agents Included

| Agent | Purpose | Scope |
|-------|---------|-------|
| `art-director` | Visual strategy and aesthetic direction | System-wide |
| `design-system-architect` | Design tokens, foundations, architecture | System-wide |
| `layout-architect` | Page structure and information architecture | Page/View level |
| `component-designer` | Individual UI component design | Component level |
| `animation-choreographer` | Motion, transitions, micro-interactions | Element level |
| `responsive-strategist` | Multi-device adaptation and breakpoints | Cross-device |
| `voice-strategist` | UX writing, tone, and microcopy | Content layer |

## Agent Deep Dive

### art-director

The strategic leader for visual expression. Transforms vague aesthetic visions ("make it feel premium", "we want a modern vibe") into systematic design principles.

**Use for:**
- Defining aesthetic direction and visual strategy
- Creating and maintaining `.design/AESTHETIC-GUIDE.md`
- Ensuring visual coherence across the system
- Translating feelings and "vibes" into actionable design principles
- Brand expression decisions

**Output artifacts:**
- Aesthetic guidelines
- Visual direction documents
- Style dimension specifications

### design-system-architect

The foundation builder. Works at the system level to establish scalable design infrastructure that supports all other design work.

**Use for:**
- Design system architecture and token design
- Establishing foundations (color, typography, spacing, motion)
- Evaluating decisions against the Nine Dimensions
- Validating Five Pillars alignment
- Cross-cutting design concerns

**Output artifacts:**
- Design token specifications
- Foundation documentation
- System-level design decisions

### layout-architect

The spatial organizer. Handles page-level structure and how information flows through your application.

**Use for:**
- Page/view layout structure (header, sidebar, main, footer)
- Information architecture and navigation hierarchy
- Grid systems and spatial composition
- Content flow and reading patterns
- Screen-level structural decisions

**Owns:** Space dimension (Nine Dimensions #4) at the page/view level

### component-designer

The craftsman. Focuses on individual UI elements, ensuring each component embodies the design system while maintaining the 9.5/10 quality baseline.

**Use for:**
- Designing new UI components
- Refining existing components
- Component-level design decisions
- Component documentation and examples
- Variant design and props API specification

**Output artifacts:**
- Component specifications
- Variant matrices
- Props API documentation

### animation-choreographer

The motion director. Transforms movement from decoration into communication, using motion to convey system state and provide feedback.

**Use for:**
- Icon animations and micro-interactions
- Page transitions and choreography
- Loading states and progress indicators
- State change animations
- Motion timing and easing decisions
- Accessibility considerations for motion

**Principle:** Animation is communication, not decoration.

### responsive-strategist

The multi-device expert. Ensures your design works across all viewport sizes and input methods.

**Use for:**
- Responsive design strategy and breakpoint definitions
- Mobile-first vs desktop-first approach decisions
- Touch vs mouse interaction patterns
- Device-specific optimizations (phone, tablet, desktop)
- Fluid typography and spacing systems

**Handles:** Web modalities across desktop, tablet, and mobile.

### voice-strategist

The content architect. Ensures language throughout your application is clear, helpful, and consistent with brand personality.

**Use for:**
- Voice & tone strategy and framework
- UX writing and microcopy (buttons, labels, placeholders)
- Error message patterns
- Empty state messaging
- Content guidelines for developers

**Owns:** Voice dimension (Nine Dimensions #3)

## The Nine Dimensions

The Design Intelligence agents evaluate and create designs using the Nine Dimensions framework:

| # | Dimension | Description | Primary Owner |
|---|-----------|-------------|---------------|
| 1 | **Style** | Visual aesthetic and brand expression | art-director |
| 2 | **Color** | Palette, contrast, accessibility | design-system-architect |
| 3 | **Voice** | Tone, language, personality | voice-strategist |
| 4 | **Space** | Layout, hierarchy, breathing room | layout-architect |
| 5 | **Motion** | Animation, transitions, feedback | animation-choreographer |
| 6 | **Touch** | Interaction patterns, affordances | component-designer |
| 7 | **Scale** | Responsive behavior, adaptation | responsive-strategist |
| 8 | **State** | Loading, error, empty, success | component-designer |
| 9 | **Flow** | User journeys, navigation | layout-architect |

Each agent considers all dimensions but takes primary ownership of specific ones.

## The Five Pillars

All design decisions are validated against the Five Pillars:

1. **Clarity** - Is the design immediately understandable?
2. **Consistency** - Does it align with established patterns?
3. **Efficiency** - Does it respect user time and attention?
4. **Delight** - Does it create positive emotional response?
5. **Accessibility** - Is it usable by everyone?

## When to Use

### Starting a New Project

```
1. art-director        → Establish aesthetic direction
2. design-system-architect → Define tokens and foundations
3. layout-architect    → Create page structure patterns
4. component-designer  → Design core UI components
5. voice-strategist    → Define content guidelines
```

### Adding a New Feature

```
1. layout-architect    → Determine where it fits
2. component-designer  → Design the components
3. animation-choreographer → Add appropriate motion
4. responsive-strategist → Ensure multi-device support
```

### Refining Existing Design

```
1. art-director        → Validate aesthetic alignment
2. component-designer  → Refine individual elements
3. animation-choreographer → Polish interactions
```

### Design System Maintenance

```
1. design-system-architect → Token updates, pattern additions
2. component-designer  → Component updates
3. voice-strategist    → Content pattern updates
```

## Agent Collaboration Patterns

### Sequential Handoff

For comprehensive design work, agents hand off to each other:

```
art-director (strategy)
    ↓
design-system-architect (foundations)
    ↓
layout-architect (structure)
    ↓
component-designer (components)
    ↓
animation-choreographer (motion)
```

### Parallel Consultation

For specific design questions, consult multiple agents simultaneously:

```
┌─────────────────────┐
│   Design Question   │
└─────────────────────┘
         │
    ┌────┼────┐
    ↓    ↓    ↓
  [A1] [A2] [A3]  (parallel consultation)
    │    │    │
    └────┼────┘
         ↓
┌─────────────────────┐
│  Synthesized Answer │
└─────────────────────┘
```

## Try It Yourself

### Define an Aesthetic Direction

```
Task: "We're building a developer tools platform. We want it to feel 
professional but approachable, technical but not intimidating."

Agent: art-director

Expected output: Aesthetic guide with:
- Core visual principles
- Color direction
- Typography recommendations
- Imagery style
```

### Design a Component

```
Task: "Design a notification toast component with success, warning, 
error, and info variants."

Agent: component-designer

Expected output:
- Component specification
- Variant matrix
- Props API
- Accessibility notes
- Usage guidelines
```

### Create Motion Guidelines

```
Task: "Define animation standards for our application, including 
page transitions and micro-interactions."

Agent: animation-choreographer

Expected output:
- Motion timing protocol
- Easing function recommendations
- Transition patterns
- Accessibility considerations (reduced motion)
```

### Responsive Strategy

```
Task: "Our app needs to work on desktop, tablet, and mobile. 
Define our responsive approach."

Agent: responsive-strategist

Expected output:
- Breakpoint definitions
- Mobile-first vs desktop-first recommendation
- Touch target specifications
- Fluid scaling approach
```

## Integration with Development

### Design-to-Code Workflow

1. Design agents produce specifications
2. Specifications feed into implementation agents
3. Component specifications become React/Vue/etc. components
4. Token specifications become CSS custom properties or theme files

### Artifact Locations

Design Intelligence agents create and maintain files in:

```
.design/
├── AESTHETIC-GUIDE.md      # art-director
├── tokens/
│   ├── colors.json         # design-system-architect
│   ├── typography.json     # design-system-architect
│   └── spacing.json        # design-system-architect
├── components/
│   └── [component].md      # component-designer
├── motion/
│   └── guidelines.md       # animation-choreographer
└── content/
    └── voice-guide.md      # voice-strategist
```

## Best Practices

1. **Start strategic, then tactical** - Begin with art-director for vision, then work down to components
2. **Let agents own their dimensions** - Trust each agent's expertise in their domain
3. **Cross-reference the Nine Dimensions** - Ensure all dimensions are addressed
4. **Validate against Five Pillars** - Use pillars as a quality checklist
5. **Iterate with specificity** - Give agents concrete context for better outputs

## Quality Baseline

All Design Intelligence agents maintain a **9.5/10 quality baseline**. This means:

- No placeholder content or "lorem ipsum" in specifications
- Accessibility is always considered
- Edge cases are documented
- Rationale is provided for design decisions
- Specifications are implementation-ready
