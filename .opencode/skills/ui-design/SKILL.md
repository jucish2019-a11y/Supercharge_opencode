---
name: ui-design
description: Design polished, Stitch-quality interfaces from wireframe to implementation with layout, spacing, color, and typography mastery
---

## What I do

I design and implement polished, production-quality UIs that feel like Stitch by Google output:

- Translate requirements and wireframes into beautiful, functional interfaces
- Apply professional layout, spacing, typography, and color decisions
- Ensure visual consistency, proper alignment, and breathability
- Deliver pixel-precise implementations that match the design intent

## When to use me

Use this skill when:
- Building a new page, screen, or feature from scratch
- Converting a rough wireframe or description into a polished UI
- Redesigning an existing interface that looks unpolished or inconsistent
- Creating a dashboard, settings page, form, or any user-facing view
- The user wants "clean", "modern", "professional", or "beautiful" UI

## Design principles (Stitch quality)

### Layout & Spacing
- Use an 8px grid system. All spacing values are multiples of 4 (4, 8, 12, 16, 24, 32, 48, 64).
- Give elements room to breathe. Generous padding beats cramped UI.
- Use consistent spacing scales. Define spacing as: `xs:4, sm:8, md:16, lg:24, xl:32, 2xl:48, 3xl:64`.
- Align everything. Left-align labels, right-align values. Consistent gutters.

### Visual Style
- Clean surfaces with subtle elevation. Use `box-shadow: 0 1px 3px rgba(0,0,0,0.08)` for cards.
- Rounded corners: `8px` for cards, `6px` for buttons, `4px` for inputs. Consistent.
- Hover states: subtle background shifts or shadow lifts, never jarring.
- Borders should be soft: `1px solid rgba(0,0,0,0.06)` or `1px solid var(--border)`.
- Use whitespace as a design element, not something to fill.

### Color
- Limit the palette. 1 primary, 1-2 accents, surface/background tones, semantic colors (success, warning, error).
- Primary color for CTAs and key actions. Everything else is neutral.
- Text: `--text-primary` (high contrast), `--text-secondary` (medium), `--text-tertiary` (low).
- Background hierarchy: `--bg-primary` (main), `--bg-secondary` (cards/sidebar), `--bg-tertiary` (nested).

### Typography
- 2 font weights maximum per design: regular (400) and semibold/bold (600/700).
- Clear type scale: 12/14/16/20/24/32/40 with consistent line heights (1.4-1.6 for body, 1.1-1.3 for headings).
- Use font size and weight to create hierarchy, not color or decoration alone.
- Left-align body text. Center only for short labels and hero text.

## How I work

1. **Understand the content** — What information needs to be displayed? What actions can the user take? What's the primary goal?
2. **Structure the layout** — Wireframe the information architecture first. Identify sections, cards, and groupings before styling.
3. **Apply the design system** — Use the project's design tokens, components, and patterns. If none exist, establish them.
4. **Implement with precision** — Write clean markup and styles. Use semantic HTML. Apply proper spacing, alignment, and visual hierarchy.
5. **Polish** — Add hover states, focus states, transitions, loading states, empty states, and error states.
6. **Review against principles** — Check spacing consistency, alignment, color usage, and typography hierarchy.

## Quality checklist

- [ ] Consistent spacing (4/8px grid)
- [ ] Proper visual hierarchy (size, weight, color — not decoration)
- [ ] Breathable layout (not cramped)
- [ ] Hover, focus, active, disabled states on all interactive elements
- [ ] Loading and empty states
- [ ] Responsive at all breakpoints
- [ ] Accessible: contrast ratios, keyboard nav, ARIA labels
- [ ] No orphaned or isolated elements
- [ ] Consistent border radius, shadows, and colors throughout