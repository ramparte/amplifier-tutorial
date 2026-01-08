---
id: bundle-design-intelligence
type: bundles
title: "Design Intelligence Bundle"
---

# Design Intelligence Bundle

Seven specialized agents for UI/UX design work.

## Overview

Design Intelligence provides domain experts for:

- Visual strategy and aesthetics
- Component design
- Layout and information architecture
- Motion and animation
- Responsive design
- Voice and tone

## Installation

```bash
# Add the bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-design-intelligence@main

# Use it
amplifier bundle use design-intelligence
```

## Agents

### art-director

Visual strategy and aesthetic direction:

```
> Define the visual direction for our admin dashboard
```

- Defines aesthetic principles
- Ensures visual coherence
- Translates "vibes" to design
- Owns style dimension

### component-designer

Individual UI component design:

```
> Design a data table component with sorting and filtering
```

- Component specifications
- Variant design
- Props API
- Accessibility

### design-system-architect

Design system infrastructure:

```
> Set up our design token system
```

- Token architecture
- Color systems
- Typography scales
- Spacing systems

### layout-architect

Page structure and composition:

```
> Design the layout for our settings page
```

- Page structure
- Information hierarchy
- Grid systems
- Content flow

### animation-choreographer

Motion and transitions:

```
> Design the page transition animations
```

- Micro-interactions
- Page transitions
- Loading states
- Motion timing

### responsive-strategist

Multi-device adaptation:

```
> How should the dashboard adapt from desktop to mobile?
```

- Breakpoint strategy
- Touch vs mouse
- Device-specific patterns
- Fluid typography

### voice-strategist

Tone and UX writing:

```
> Write the error messages for our form validation
```

- Voice and tone
- Microcopy
- Error messages
- Empty states

## Design Framework

### Nine Dimensions

The agents follow a 9-dimension design evaluation:

1. **Style** - Visual aesthetic
2. **Layout** - Spatial composition
3. **Voice** - Language and tone
4. **Space** - Whitespace and density
5. **Motion** - Animation and transitions
6. **Color** - Palette and usage
7. **Typography** - Font choices and hierarchy
8. **Components** - UI building blocks
9. **Interaction** - User input patterns

### Five Pillars

Quality baseline principles:

1. **Clarity** - Instantly understandable
2. **Efficiency** - Minimal friction
3. **Consistency** - Predictable patterns
4. **Delight** - Thoughtful details
5. **Accessibility** - Universal usability

## Workflow Example

### Design a New Feature

```
# 1. Art direction
> Use art-director to define the visual approach for notifications

# 2. Layout
> Use layout-architect to structure the notification panel

# 3. Components
> Use component-designer to design the notification card

# 4. Motion
> Use animation-choreographer to design entry/exit animations

# 5. Copy
> Use voice-strategist to write notification messages
```

### Design System Setup

```
# 1. Architecture
> Use design-system-architect to create our token structure

# 2. Components
> Use component-designer to design core components

# 3. Responsive
> Use responsive-strategist to define breakpoints
```

## Agent Collaboration

Agents reference each other's work:

```yaml
# Recipe for comprehensive design
name: design-feature
steps:
  - id: direction
    agent: design-intelligence:art-director
    instruction: "Define visual direction for {{feature}}"

  - id: layout
    agent: design-intelligence:layout-architect
    instruction: |
      Design layout for {{feature}}
      Following direction: {{direction.result}}

  - id: components
    agent: design-intelligence:component-designer
    instruction: |
      Design components for {{feature}}
      Layout: {{layout.result}}
      Direction: {{direction.result}}
```

## Try It Yourself

### Exercise 1: Component Design

```
> Use component-designer to design a user profile card
```

### Exercise 2: Layout Design

```
> Use layout-architect to design a dashboard home page
```

### Exercise 3: Multi-Agent Flow

```
> First use art-director to define the style for a settings page,
> then use layout-architect to structure it
```

## Output Formats

Agents produce:

- **Specifications** - Detailed design docs
- **Token definitions** - Design system values
- **Component APIs** - Props and variants
- **Guidelines** - Usage documentation

## Source

```
github.com/microsoft/amplifier-bundle-design-intelligence
├── bundle.yaml
├── agents/
│   ├── art-director.yaml
│   ├── component-designer.yaml
│   ├── design-system-architect.yaml
│   ├── layout-architect.yaml
│   ├── animation-choreographer.yaml
│   ├── responsive-strategist.yaml
│   └── voice-strategist.yaml
└── context/
    ├── nine-dimensions.md
    └── five-pillars.md
```
