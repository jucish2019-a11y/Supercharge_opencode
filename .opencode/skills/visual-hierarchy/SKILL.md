---
name: visual-hierarchy
description: Master spacing, typography scale, color theory, and layout composition for professional UI design
---

## What I do

I apply visual hierarchy principles to make interfaces clear, scannable, and beautiful:

- Establish clear typographic hierarchy so content flows naturally
- Use spacing and grouping to organize information
- Apply color and contrast intentionally to guide attention
- Compose layouts that feel balanced and professional

## When to use me

Use this skill when:
- A UI feels flat, cluttered, or visually confusing
- Designing a page layout from scratch
- Typography and spacing feel inconsistent or arbitrary
- Content doesn't guide the eye to the right places
- A page needs to feel "designed" rather than "assembled"

## How I work

1. **Identify the primary goal** — What should the user see first? What action should they take? Define the visual priority order.
2. **Apply the F-pattern or Z-pattern** — Users scan in F-shapes on content pages and Z-shapes on landing pages. Place the most important elements along these natural scan paths.
3. **Create contrast ratios between levels** — Each level of hierarchy must be clearly distinguishable from the next:

   | Level | Font Size | Font Weight | Color |
   |---|---|---|---|
   | H1 — Page title | 28-32px | 700 | --text-primary |
   | H2 — Section title | 20-24px | 600 | --text-primary |
   | H3 — Subsection | 16-18px | 600 | --text-primary |
   | Body — Main content | 14-16px | 400 | --text-primary |
   | Secondary — Descriptions | 14px | 400 | --text-secondary |
   | Caption — Metadata | 12-13px | 400 | --text-tertiary |

4. **Use spacing to group and separate** — The Law of Proximity: related items are closer together, unrelated items are further apart.
   - Within a group: 4-8px between elements
   - Between groups: 16-24px between sections
   - Between sections: 32-48px between major areas
   - Between pages: 64-96px padding at page edges

5. **Control density** — Match information density to the context:
   - Landing pages: generous whitespace, focused message
   - Dashboards: compact but scannable, data-dense
   - Settings: moderate density, clear labels, comfortable form fields

6. **Add visual anchors** — Use color, size, and weight to create 2-3 visual anchors per view that draw the eye:
   - CTA button (color + size)
   - Page title (size + weight)
   - Key metric or stat (size + contrast)

## Spacing system

Use a consistent scale based on 4px increments:

```
4px   — tight: inline gaps, icon-to-label
8px   — compact: within-component spacing
12px  — snug: related items within a group
16px  — standard: form field gaps, list items
24px  — relaxed: between groups of elements
32px  — spacious: between sections
48px  — large: major section breaks
64px  — generous: page-level padding
```

## Common hierarchy mistakes

- Too many font sizes (stick to 4-5 sizes per page)
- Too many font weights (stick to 2: regular + bold/semibold)
- Too many colors (1 primary accent + neutrals + 2-3 semantic)
- Equal spacing everywhere (variety creates rhythm)
- Everything competing for attention (hierarchy means some things are more important)

## Guidelines

- If you can't tell what's most important in 3 seconds, the hierarchy is wrong
- Contrast is the primary tool: big vs small, bold vs light, colored vs neutral
- Whitespace is not wasted space — it's the pause between notes
- Use a maximum of 3 levels of visual hierarchy per section
- Consistency of spacing > consistency of values (use the same 4px scale everywhere)