---
name: spatial-design
description: Design spatial relationships, density, containment, and visual grouping using Gestalt principles and systematic spacing for cohesive UI layouts
---

## What I do

I design spatial systems that make UIs feel organized, intentional, and breathable — the invisible architecture that separates amateur layouts from professional ones:

- **Spatial rhythm** — Consistent spacing that creates visual harmony across the entire product
- **Containment** — Using surfaces, borders, and backgrounds to group related content
- **Density control** — Adjusting information density per context (data-heavy vs editorial)
- **Visual grouping** — Applying Gestalt principles (proximity, similarity, enclosure) for clear information architecture
- **Negative space** — Strategic use of emptiness as a design element

## When to use me

Use this skill when:
- A layout feels cramped, cluttered, or visually chaotic
- Spacing feels inconsistent or arbitrary across a product
- Creating templates or layout systems for pages
- Deciding when to use cards, borders, or whitespace for grouping
- Making data-dense interfaces still feel scanable and breathable
- Translating a design into code with proper spacing tokens

## How I work

1. **Audit current spacing** — List all spacing values currently in use. Identify inconsistencies (13px here, 14px there, 15px elsewhere — all trying to be "a small gap").
2. **Define the spatial scale** — Establish a spacing token system based on 4px increments.
3. **Apply grouping rules** — Use Gestalt principles to determine what belongs together.
4. **Choose containment level** — Decide how to group: whitespace, divider, surface, or card.
5. **Set density levels** — Define compact, default, and comfortable density presets.
6. **Apply systematically** — Map every spatial decision to a token, never guess.

## The spatial scale

Based on 4px increments, every spacing value in the product maps to a token:

```
Token          Value   Usage
--space-0      0px     Reset, no spacing
--space-1      4px     Tight: icon-to-text gap, inline padding in badges
--space-2      8px     Compact: within-form-element spacing, tight list gaps
--space-3     12px     Snug: between label and input, small card inner padding
--space-4     16px     Standard: card padding, list item gaps, form field spacing
--space-5     20px     Comfortable: between related groups
--space-6     24px     Relaxed: between form sections, card-to-card gap
--space-8     32px     Spacious: section dividers, sidebar padding
--space-10    40px     Large: between major sections in a page
--space-12    48px     Generous: hero section padding, major section breaks
--space-16    64px     Page: page-level vertical padding
--space-20    80px     Screen: between full-screen sections
--space-24    96px     Grand: landing page section spacing
```

Rules:
- Use `4px` for micro-adjustments (icon gaps, tight aligning)
- Use `8px` for inline-level spacing (between words and icons, badge padding)
- Use `16px` for element-level spacing (between form fields, list items)
- Use `24px` for group-level spacing (between card sections, sidebar items)
- Use `32-48px` for section-level spacing (between page sections)
- Use `64-96px` for page-level spacing (between major content areas)

## Gestalt grouping rules

The Gestalt principles determine how users perceive grouping. Apply them intentionally:

### Proximity (most powerful)
Elements closer together are perceived as related.
- Related items: `4-12px` apart
- Unrelated items: `16-32px` apart
- New sections: `32-64px` apart
- **Rule**: If two items feel disconnected, increase the gap. If they feel like a group, decrease it.

### Similarity
Elements that look similar are perceived as related.
- Same font size/weight = related content
- Same background = same level of hierarchy
- Same color = same interactive affordance
- **Rule**: Style related items identically. Style unrelated items differently.

### Enclosure
Elements inside a boundary are perceived as a group.
- Card background = strong grouping
- Divider line = moderate grouping
- Whitespace gap = minimal grouping
- **Rule**: Use the lightest containment that still communicates the grouping.

### Continuity
Elements aligned along a line or curve are perceived as related.
- Left-aligned labels create a strong vertical relationship
- Grid alignment creates order
- **Rule**: Align elements to create clear visual paths.

### Figure-Ground
Users distinguish content from background.
- Content = figure (foreground attention)
- Background = ground (contextual surface)
- **Rule**: Make important content pop by making it the figure. Subtle gray surfaces for ground.

## Containment levels

Choose the lightest containment that communicates the grouping:

| Level | When to use | Visual treatment | Example |
|-------|-------------|-----------------|---------|
| **Whitespace** | Weakly related items, simple layouts | Just spacing, no visual boundary | List items, form fields |
| **Divider** | Sequential groups within a section | 1px line, `--border` color | Settings groups, list sections |
| **Surface** | Distinct sections on the same level | Background fill `--bg-secondary` | Sidebar, alternating rows |
| **Card** | Independent, self-contained content blocks | Background + border + shadow | Project cards, dashboard widgets |
| **Panel** | Major content areas, persistent regions | Background + border + distinct surface | Main content area, detail panel |

Rules:
- Don't use a card when whitespace will do
- Don't use a border when a surface change will do
- Don't use a surface when whitespace will do
- The best UIs use containment sparingly — let whitespace do the work

## Density presets

Different contexts need different density. Define three modes:

### Compact (data tables, admin dashboards, developer tools)
```
--density-compact:
  Element height: 32px
  Padding: 8px 12px
  Gap: 8px
  Line height: 1.3
  Font size: 13-14px
```
- Use when: users need maximum information density
- Warning: harder to read, faster to scan

### Default (most applications, forms, dashboards)
```
--density-default:
  Element height: 40px
  Padding: 12px 16px
  Gap: 12px
  Line height: 1.5
  Font size: 14-16px
```
- Use when: balanced readability and density
- The starting point for most UIs

### Comfortable (landing pages, onboarding, presentations)
```
--density-comfortable:
  Element height: 48px
  Padding: 16px 24px
  Gap: 16-24px
  Line height: 1.6
  Font size: 16-18px
```
- Use when: focusing attention, telling a story, giving information room
- Maximum readability, minimum density

## Vertical rhythm

All vertical spacing should align to the line-height grid:

```
Body line-height: 24px (16px × 1.5)
↓
Paragraph margin: 24px (1 line)
Section margin: 48px (2 lines)
Page padding: 72px (3 lines)
```

This creates a vertical rhythm where every element's vertical position aligns to the text grid. The result is a layout that feels "in sync" — the same way a musical beat creates rhythm.

## Spacing application patterns

### Page layout
```
┌─────────────────────────────────────────┐
│ ← ─ ─ 64px (page padding) ─ ─ →        │
│  ┌─────────────────────────────────┐    │
│  │ Header (32px bottom margin)      │    │
│  ├─────────────────────────────────┤    │
│  │ Section (24px inner padding)     │    │
│  │  ┌───────────────────────────┐  │    │
│  │  │ Card (16px inner padding)  │  │    │
│  │  │  Title (8px below)        │  │    │
│  │  │  Description              │  │    │
│  │  │  (16px below)             │  │    │
│  │  │  Action                   │  │    │
│  │  └───────────────────────────┘  │    │
│  │  (24px between cards)            │    │
│  └─────────────────────────────────┘    │
│                                          │
│ ← ─ ─ 64px (page padding) ─ ─ →        │
└─────────────────────────────────────────┘
```

### Form layout
```
Label (4px below)
Input (16px below)
Label (4px below)
Input (16px below)
Helper text (24px below) — gap between fields
Label (4px below)
Input (8px below) — gap between input and its error
Error text (24px below) — gap before next field
```

### List layout
```
List Item (8px inner vertical padding, 12px horizontal)
  Icon (8px right gap) Text
List Item (8px inner vertical padding)
──────────── Divider (0px gap from items) ────────
List Item (8px inner vertical padding)
```

## Quality checklist

- [ ] Every spacing value maps to a design token
- [ ] No magic numbers (hardcoded pixel values)
- [ ] Containment level is appropriate (not over-carded)
- [ ] Density matches the context (compact for data, comfortable for content)
- [ ] Vertical rhythm is maintained (spacing aligns to line-height grid)
- [ ] Grouping is clear — related items are closer, unrelated items are further
- [ ] Whitespace is used intentionally, not as an afterthought
- [ ] Mobile spacing is adjusted (reduce page padding, keep touch targets)

## Anti-patterns I avoid

- Arbitrary spacing values (13px, 17px, 19px) — use the token scale
- Cards inside cards — if you need nested containment, redesign the grouping
- Walls of text without section breaks — add spacing and headers
- Equal spacing everywhere — hierarchy requires varying gaps
- Too many visual boundaries — let whitespace group where possible
- Ignoring density context — a dashboard and a landing page need different density
- Forgetting touch targets — minimum 44px on mobile, even in compact mode