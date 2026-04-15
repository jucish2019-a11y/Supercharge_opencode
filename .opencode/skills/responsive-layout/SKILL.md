---
name: responsive-layout
description: Design and implement beautiful adaptive layouts across all screen sizes and breakpoints
---

## What I do

I create layouts that look beautiful and function perfectly at every screen size:

- Define breakpoint systems and responsive strategies
- Implement fluid layouts that adapt gracefully between sizes
- Handle layout shifts and responsive typography
- Ensure touch targets, readability, and navigation work on mobile

## When to use me

Use this skill when:
- Building pages that must work on mobile, tablet, and desktop
- A layout breaks or looks poor at certain sizes
- Creating responsive navigation, grids, or data tables
- Implementing mobile-first or desktop-first designs

## How I work

1. **Identify content requirements** — List what content exists and how it relates. Determine what's essential vs optional at each size.
2. **Choose a responsive strategy** — Based on content and audience:
   - Mobile-first: Start small, expand. Best for content-heavy sites.
   - Desktop-first: Start wide, adapt down. Best for dashboards and data.
   - Component-first: Design each component across sizes, then compose. Best for design systems.
3. **Define breakpoints** — Use the project's breakpoint system. If none exists:
   ```
   sm:  640px   -- Large phones
   md:  768px   -- Tablets portrait
   lg:  1024px  -- Tablets landscape / small laptops
   xl:  1280px  -- Desktops
   2xl: 1536px  -- Large desktops
   ```
4. **Plan the layout at each breakpoint** — Sketch or describe layout changes:
   - Mobile: single column, stacked, collapsible nav
   - Tablet: two columns, sidebar starts
   - Desktop: full layout, sidebar fixed, multi-column grids
5. **Implement with fluid foundations** — Use relative units, flexbox, and grid. Avoid fixed pixel widths where possible.
6. **Polish responsive details** — Handle responsive typography, touch targets (minimum 44x44px), responsive images, and navigation patterns.

## Layout patterns by component

### Navigation
- Mobile: hamburger menu or bottom nav
- Tablet: condensed sidebar or top nav
- Desktop: full sidebar or horizontal nav

### Data tables
- Mobile: card layout or horizontal scroll with sticky first column
- Tablet: show key columns, hide secondary
- Desktop: full table with all columns

### Forms
- Mobile: stacked labels above inputs, full-width
- Tablet/Desktop: side-by-side label + input where space allows

### Grids
- Mobile: 1 column
- Tablet: 2 columns
- Desktop: 3-4 columns

## CSS approach

```css
/* Mobile first */
.grid { display: grid; grid-template-columns: 1fr; gap: 16px; }

@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); gap: 24px; }
}
```

## Guidelines

- Test at every breakpoint, not just the edges
- Never hide critical content on mobile — redesign it for small screens
- Touch targets: minimum 44x44px tap areas on mobile
- Use `clamp()` for fluid typography: `font-size: clamp(1rem, 2.5vw, 1.5rem)`
- Use container queries where supported for component-level responsiveness
- Avoid horizontal scroll except for intentionally scrollable areas (carousels, tables)
- Responsive images: use `srcset` and `<picture>` for art direction