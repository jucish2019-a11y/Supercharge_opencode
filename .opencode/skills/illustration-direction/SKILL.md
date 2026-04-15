---
name: illustration-direction
description: Direct illustration and visual content — empty states, hero illustrations, icons-as-illustrations, data visualizations, and decorative visual elements with a cohesive style guide
---

## What I do

I create art direction and style guides for visual content in UIs — illustrations, empty states, micro-illustrations, data visualizations, and decorative elements:

- **Empty state illustrations** — Friendly, on-brand visuals that guide users to take action
- **Hero illustrations** — Landing page visuals that communicate the product's value
- **Micro-illustrations** — Small visuals for onboarding, feature highlights, tooltips
- **Data visualization style** — Chart colors, axis styling, tooltip design, legend patterns
- **Visual consistency** — A cohesive illustration style across all visual content

## When to use me

Use this skill when:
- Designing empty states, onboarding, or error illustrations
- Creating hero or landing page visuals
- Establishing an illustration style for a product
- Designing data visualizations (charts, graphs, dashboards)
- Making visuals that match the UI's design language

## How I work

1. **Define the illustration style** — Choose a visual direction before creating any illustration:
   - **Line art**: Thin strokes, minimal fill, clean, technical feel. Great for developer tools.
   - **Flat**: Solid shapes, no outlines, bold colors, geometric. Great for SaaS and productivity.
   - **Isometric**: 3D-ish perspective, precision, professional. Great for infrastructure and tech.
   - **Abstract**: Shapes, gradients, texture, mood-focused. Great for editorial and brand.
   - **Duotone**: Two-color limitation, modern, sophisticated. Great for premium and minimal UIs.
   - **3D/Clay**: Soft, rounded, tactile, approachable. Great for consumer and lifestyle apps.

2. **Set the illustration rules** — Define constraints that ensure consistency:
3. **Create the visual** — Apply the style within the constraints.
4. **Implement in code** — SVG for illustrations, canvas/SVG for data, CSS for decorative.

## Illustration style guide

For every illustration style, define these rules:

### Color rules
- Use the product's primary accent color as the dominant color
- Secondary color: a muted complement or analogous hue
- Neutrals: product's neutral scale (gray-100 to gray-900)
- Maximum 3-4 colors per illustration
- Accent color covers 10-20% of the illustration — it highlights, not overwhelms
- Faces and hands: use the product's skin tone ranges or abstract (no face)

### Shape rules
- Corner radius: consistent throughout (rounded, sharp, or mixed — pick one)
- Level of detail: low detail (flat shapes) vs high detail (realistic)
- Perspective: flat (front-facing), isometric (30° angle), or 3D perspective
- Proportions: consistent limb lengths, head sizes, object scales

### Stroke rules (if applicable)
- Stroke width: 1.5-2px for line art style
- Stroke color: product's text-primary or a muted accent variant
- Fill: none (outline only) or solid fill with strokes for definition
- Consistent line endings: round caps, not square or butt

### Size rules
- Inline micro-illustrations: 24-48px
- Empty state illustrations: 120-200px
- Hero illustrations: 400-600px wide
- Maximum size on mobile: 80vw, with proper padding

### Background rules
- Illustrations sit on `--bg-primary` (transparent or white)
- Never put illustrations on colored backgrounds unless designed for it
- If a background shape is needed: use a soft circle or blob in `--color-primary` at 8% opacity

## Empty state illustrations

Empty states are the most common illustration need. Every empty state follows this structure:

```
┌──────────────────────────────────┐
│                                  │
│        [Illustration]            │
│         120-200px                │
│                                  │
│     Heading (text-lg, 600)       │
│   Description (text-sm, 400)     │
│                                  │
│        [Primary CTA]             │
│     [Secondary link]              │
│                                  │
└──────────────────────────────────┘
```

### Empty state copy guidelines
- **Heading**: 3-5 words, encouraging tone ("No projects yet", "No results found")
- **Description**: 1-2 sentences explaining the state and what to do. Never blame the user.
- **CTA**: Action-oriented button labels ("Create project", "Try a different search")
- **Secondary**: Optional link to related actions ("Import from CSV", "Learn more")

### Empty state illustration subjects
- **No data**: Empty folder, blank document, empty inbox
- **No results**: Magnifying glass with no findings, empty search
- **No connection**: Cloud with a line through it, broken link
- **Error**: Broken robot, warning triangle, spilled drink (lighthearted products)
- **Success**: Checkmark, confetti, high-five
- **Onboarding**: Rocket, map, door opening

Illustration should metaphorically relate to the state but not be literal. Use the product's accent color for 1-2 key elements.

## Data visualization style

Charts and data visuals need their own style system:

### Chart color palette
For categorical data (up to 8 series):
```
--chart-1: var(--color-primary)       /* Primary accent */
--chart-2: #8B5CF6                     /* Purple */
--chart-3: #F59E0B                     /* Amber */
--chart-4: #10B981                     /* Emerald */
--chart-5: #EC4899                     /* Pink */
--chart-6: #6366F1                     /* Indigo */
--chart-7: #14B8A6                     /* Teal */
--chart-8: #F97316                     /* Orange */
```

Rules:
- Never use red/green alone (colorblindness) — pair with shape or pattern
- Use the primary color for the most important series
- Use muted versions for background/historical series
- Highlight (bright saturated) for the series in focus, dim others

### Axis and grid styling
```
Axis line: none (clean look) or 1px --border
Grid lines: 1px dashed, rgba(0,0,0,0.06) / rgba(255,255,255,0.06)
Axis labels: --text-tertiary, --text-xs (12px)
Tick marks: none (minimal) or 4px, --border
```

### Chart tooltip
```
Background: --bg-secondary or --bg-tertiary (dark mode)
Border: 1px solid --border
Border-radius: --radius-md (8px)
Padding: --space-8 (8px) --space-12 (12px)
Shadow: 0 4px 16px rgba(0,0,0,0.12)
Font: --text-sm (14px), --weight-medium (500) for values
```

### Chart annotations
- Direct labels preferred over legends when series < 5
- Legends at the bottom, horizontal row, with colored dot + label
- Annotations for key insights: small callout with leader line

## Hero illustrations

Hero illustrations for landing pages or onboarding:

### Layout patterns
- **Left text, right illustration**: Classic, balanced. Illustration 40-50% viewport width.
- **Centered text, illustration below**: Modern, focused. Illustration 60% viewport width.
- **Illustration as background**: Editorial, immersive. Text overlaid with gradient.

### Hero illustration rules
- Illustration should demonstrate the product's value, not be decorative
- Limit to 1 key message per illustration
- Use the product's UI elements inside the illustration (meta-illustration)
- Animate subtly on scroll or load (entrance animation, 300-500ms)
- On mobile: stack vertically, illustration above text or below (test both)

## Decorative visual elements

### Background patterns
```css
/* Subtle dot grid */
background-image: radial-gradient(circle, var(--border) 1px, transparent 1px);
background-size: 24px 24px;

/* Diagonal lines */
background-image: repeating-linear-gradient(
  -45deg,
  transparent, transparent 10px,
  var(--bg-tertiary) 10px, var(--bg-tertiary) 11px
);
```
- Opacity: 3-8% of accent color — barely visible, atmospheric
- Never compete with content — decoration supports, never distracts

### Gradient backgrounds
```css
/* Soft radial gradient for hero sections */
background: radial-gradient(
  ellipse at 30% 50%,
  var(--color-primary) 0%,
  transparent 70%
);
opacity: 0.06;
```
- Use: hero sections, feature highlights, empty state backgrounds
- Opacity: 4-8% — should feel like a tint, not a colored section
- Direction: radiate from where you want focus

### Dividers and separators
- Thin line: `1px solid var(--border)` — subtle section breaks
- Gradient fade: `linear-gradient(to right, transparent, var(--border), transparent)` — decorative
- Space-only: 48-64px gap — cleanest option, for major section breaks

## Quality checklist

- [ ] Illustration style is defined and documented (style, color, stroke rules)
- [ ] All illustrations use the product's accent color and neutral palette
- [ ] Empty states have illustration + heading + description + CTA
- [ ] Chart colors have sufficient contrast and colorblind-safe alternatives
- [ ] Hero illustrations communicate value, not just decoration
- [ ] Micro-illustrations are consistent in style and sizing
- [ ] All visuals work in both light and dark modes
- [ ] SVGs are optimized, use `currentColor`, and are accessible
- [ ] Decorative elements are subtle (3-8% opacity) and don't distract

## Anti-patterns I avoid

- Stock photos that don't match the UI style — use illustrations instead
- Overly detailed illustrations at small sizes — simplify for 48px and below
- Using red for negative data and green for positive without an additional indicator — 8% of men are colorblind
- More than 4 colors in a single illustration — visual noise
- Illustrations that look like a different product — stay on brand
- Empty states without a CTA — the user is stuck and doesn't know what to do
- 3D illustrations mixed with flat icons — pick one style
- Animated illustrations as page background — performance and distraction