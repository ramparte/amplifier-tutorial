---
id: design-intelligence
type: bundles
title: "Design Intelligence Bundle"
---

# Design Intelligence Bundle

## Overview

The Design Intelligence bundle provides a team of specialized AI agents for UI/UX design work. Rather than a single generalist design assistant, this bundle offers role-specific experts that mirror how professional design teams operate.

Each agent brings deep expertise in their domain while sharing a common design philosophy rooted in the Nine Dimensions and Five Pillars framework. They work together through orchestration, allowing you to tackle design challenges from strategic vision down to pixel-level implementation.

## Agents Included

| Agent | Purpose | Primary Domain |
|-------|---------|----------------|
| `art-director` | Visual strategy and aesthetic direction | Style, brand expression |
| `component-designer` | Individual UI component design | Component-level refinement |
| `animation-choreographer` | Motion and transitions | Animation timing, micro-interactions |
| `design-system-architect` | Design tokens and foundations | System-level architecture |
| `layout-architect` | Page structure and spatial composition | Information architecture |
| `responsive-strategist` | Multi-device adaptation | Breakpoints, device patterns |
| `voice-strategist` | UX writing and tone | Microcopy, messaging |

### art-director

The Art Director owns aesthetic direction and visual strategy. This agent transforms abstract concepts like "feels premium" or "approachable but professional" into systematic design principles.

**Responsibilities:**
- Defining and maintaining the aesthetic guide
- Ensuring visual coherence across the system
- Translating brand personality into design decisions
- Resolving conflicts between competing visual approaches

**Best for:** Starting new projects, establishing visual identity, reviewing design consistency.

### component-designer

The Component Designer works at the individual element level, crafting buttons, cards, inputs, and other UI building blocks that embody your design system.

**Responsibilities:**
- Designing new UI components from specifications
- Refining existing components for quality
- Creating component variants and props APIs
- Documenting component usage and examples

**Best for:** Building design system libraries, refining individual elements, component documentation.

### animation-choreographer

The Animation Choreographer treats motion as communication. Every transition, micro-interaction, and state change carries meaning and provides feedback.

**Responsibilities:**
- Designing icon animations and micro-interactions
- Planning page transitions and choreography
- Creating loading states and progress indicators
- Establishing motion timing and easing standards
- Ensuring accessibility for motion-sensitive users

**Best for:** Adding polish to interactions, designing loading experiences, establishing motion language.

### design-system-architect

The Design System Architect works at the foundation level, establishing tokens, scales, and systematic patterns that ensure consistency across all design work.

**Responsibilities:**
- Designing color, typography, and spacing scales
- Creating and managing design tokens
- Establishing grid systems and layout foundations
- Evaluating decisions against the Nine Dimensions
- Defining cross-cutting design concerns

**Best for:** Starting design systems, establishing foundations, ensuring systematic consistency.

### layout-architect

The Layout Architect handles page-level structure and information hierarchy. They determine how content flows and how users navigate through information.

**Responsibilities:**
- Designing page/view layouts (header, sidebar, main, footer)
- Planning information architecture and navigation
- Creating grid systems and spatial composition
- Establishing content flow and reading patterns

**Best for:** Planning application structure, designing navigation systems, establishing page templates.

### responsive-strategist

The Responsive Strategist ensures designs work across all devices and input methods, from mobile touch to desktop mouse interactions.

**Responsibilities:**
- Defining breakpoint strategy and behavior
- Planning mobile-first vs desktop-first approaches
- Adapting touch vs mouse interaction patterns
- Optimizing for specific device categories
- Implementing fluid typography and spacing

**Best for:** Multi-device design, mobile optimization, touch interface considerations.

### voice-strategist

The Voice Strategist ensures every word in your interface serves users. From button labels to error messages, language shapes user experience.

**Responsibilities:**
- Establishing voice and tone frameworks
- Writing UX copy and microcopy
- Creating error message patterns
- Designing empty state messaging
- Developing content guidelines for developers

**Best for:** Interface copy, error handling UX, establishing tone guidelines.

## The Nine Dimensions

The Design Intelligence agents share a common evaluation framework called the Nine Dimensions. When assessing design quality, each dimension receives attention:

| Dimension | Focus | Key Questions |
|-----------|-------|---------------|
| **Style** | Visual identity and aesthetic | Does it feel cohesive? Does it express the brand? |
| **Color** | Palette, contrast, meaning | Is contrast accessible? Do colors communicate state? |
| **Voice** | Language, tone, personality | Is copy clear and helpful? Is tone consistent? |
| **Space** | Layout, whitespace, breathing room | Does content have room to breathe? Is hierarchy clear? |
| **Motion** | Animation, transitions, feedback | Does motion communicate? Is timing appropriate? |
| **Type** | Typography, readability, scale | Is text readable? Does hierarchy guide the eye? |
| **Touch** | Interaction, affordance, feedback | Are targets large enough? Is interaction clear? |
| **State** | Loading, error, empty, success | Are all states designed? Is feedback immediate? |
| **Flow** | Navigation, progression, journey | Is the path clear? Can users recover from mistakes? |

Agents reference these dimensions when making decisions and can explain their recommendations in terms of specific dimensional impacts.

## The Five Pillars

Beyond the Nine Dimensions, agents are guided by Five Pillars that establish baseline quality:

1. **Clarity** - Every element communicates its purpose
2. **Consistency** - Patterns repeat predictably across the system
3. **Accessibility** - Design works for all users and abilities
4. **Performance** - Visual design supports fast, responsive experiences
5. **Delight** - Thoughtful details that elevate the mundane

## When to Use

### Starting a New Project

Begin with `art-director` to establish aesthetic direction, then `design-system-architect` to create foundations:

```
You: "I'm building a developer tools SaaS. It should feel professional 
     but not corporate, technical but approachable."

art-director: Analyzes requirements, creates AESTHETIC-GUIDE.md with 
              color direction, typography approach, and visual principles.

design-system-architect: Translates aesthetic into concrete tokens - 
                         color scales, type scales, spacing system.
```

### Building Components

Use `component-designer` when implementing specific UI elements:

```
You: "Design a notification badge component that shows unread counts"

component-designer: Creates component spec with variants (dot, count, 
                    overflow), states (default, attention, muted), 
                    and accessibility considerations.
```

### Adding Motion

Bring in `animation-choreographer` to design purposeful animations:

```
You: "The save button feels static. It should provide feedback."

animation-choreographer: Designs micro-interaction sequence - 
                         button press response, processing state, 
                         success confirmation with appropriate timing.
```

### Multi-Device Work

Consult `responsive-strategist` for device-specific decisions:

```
You: "Our dashboard has too much information for mobile screens"

responsive-strategist: Analyzes content priority, suggests progressive 
                       disclosure pattern, defines what collapses vs 
                       hides vs reorganizes at each breakpoint.
```

### Writing Interface Copy

Use `voice-strategist` for any text that users see:

```
You: "What should the error message say when file upload fails?"

voice-strategist: Crafts message considering context, user emotion, 
                  actionability: "Upload failed. Check your connection 
                  and try again, or try a smaller file (max 10MB)."
```

## Try It Yourself

### Establish Visual Direction

```
I'm building a meditation app for busy professionals. The vibe should 
be calming but not new-age, minimal but not cold. Help me establish 
the visual direction.
```

### Design a Component

```
Design a progress indicator component for a multi-step form. It needs 
to show: current step, completed steps, remaining steps, and allow 
clicking back to completed steps.
```

### Plan Motion

```
Design the animation for a mobile menu that slides in from the right. 
Consider: the trigger button, the overlay, the menu content, and how 
items stagger in.
```

### Create Responsive Strategy

```
I have a data table that works on desktop but breaks on mobile. The 
table has 8 columns: name, status, date, amount, category, assignee, 
priority, and actions. Help me plan the responsive approach.
```

### Write Error Messages

```
Create a set of error messages for a payment form. Cover: invalid card 
number, expired card, insufficient funds, network error, and general 
processing failure.
```

## Agent Collaboration

Design Intelligence agents work together through orchestration. Common collaboration patterns:

**Art Director + Design System Architect**: Art director sets direction, architect implements as tokens.

**Component Designer + Animation Choreographer**: Designer creates static component, choreographer adds motion.

**Layout Architect + Responsive Strategist**: Layout architect designs desktop, strategist adapts for all devices.

**Voice Strategist + Component Designer**: Voice strategist writes copy, designer integrates into component specs.

## Configuration

The Design Intelligence bundle is included in Amplifier Foundation. Enable it in your bundle configuration:

```yaml
# .amplifier/config.yaml
bundles:
  - foundation
  - design-intelligence
```

Individual agents can be invoked directly or through the main assistant which will delegate to the appropriate specialist based on your request.

## Quality Baseline

All Design Intelligence agents work toward a 9.5/10 quality baseline. This means:

- No placeholder or generic solutions
- Every decision justified against the Nine Dimensions
- Accessibility considered by default
- Edge cases and states addressed
- Implementation guidance included

When you receive design recommendations, you're getting production-ready guidance, not rough sketches.

## Related Resources

- [Amplifier Foundation Bundle](./foundation.md) - Core development agents
- [Advanced Topics](../advanced/index.md) - Custom bundles and tools
- [Bundles Guide](../bundles/index.md) - How bundles work together
