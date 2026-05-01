---
name: proportions
description: Design with intentional ratios — type scales, space scales, container proportions, and the mathematical relationships that create visual harmony
---

## What I do

I design proportional systems where every dimension relates to every other through intentional ratios — the mathematical foundation that separates harmonious design from "spacing that looks okay":

- **Type scale ratios** — Choose and apply mathematical ratios for font sizes
- **Space scale ratios** — Derive spacing from the same system as type
- **Container proportions** — Set page width, content width, and margins with purpose
- **Tschichold's margin method** — Derive margins from the page itself
- **Type-space harmony** — Ensure type scale and space scale work together

## When to use me

Use this skill when:
- Setting up a design system from scratch (type scale + space scale)
- Spacing feels arbitrary or inconsistent across a product
- Layout proportions look "off" but you can't name why
- Choosing between type scale ratios (1.25 vs 1.333 vs 1.5)
- Deriving page margins, gutters, and column widths
- Audit reveals "magic number" spacing values with no system

## How I work

### Checker mode (auditing existing proportions)

1. **Collect all font sizes** — List every font-size value in the codebase.
2. **Collect all spacing values** — List every margin, padding, and gap value.
3. **Test for a ratio** — Divide each size by the previous. Are they consistent? Do they follow a named ratio?
4. **Check type-space alignment** — Does the space scale use the same ratio as the type scale? Are they aligned at the base?
5. **Check container proportion** — Does the content width have an intentional ratio to the page width?
6. **Check margin derivation** — Were margins chosen arbitrarily, or do they relate to the content area?
7. **Report ratio violations** — Any value that doesn't fit the system is a magic number.

### Applier mode (building from scratch)

1. **Choose the primary ratio** — Based on the aesthetic direction (see: design-foundations).
2. **Generate the type scale** — Derive from base size and ratio.
3. **Generate the space scale** — Derive from the same base and ratio (or a complementary one).
4. **Set container proportions** — Content width, margins, gutters.
5. **Implement as tokens** — CSS custom properties or design tokens.
6. **Verify with measurements** — Check actual rendered sizes against the system.

## The proportional ratios

Every intentional design system is built on a named ratio. Choose one — not "whatever looks good."

### Common ratios and their character

| Ratio | Name | Character | Best for |
|-------|------|-----------|----------|
| 1.067 | Minor second | Subtle, compact | Data-dense UIs, fine-grained scales |
| 1.125 | Major second | Tight, controlled | Admin panels, dashboards |
| 1.200 | Minor third | Moderate, efficient | Functional apps, forms |
| 1.250 | Major third | Modern, balanced | **Most common for digital UI** |
| 1.333 | Perfect fourth | Bold, editorial | **Landing pages, marketing, expressive** |
| 1.414 | Augmented fourth | Dramatic, wide range | Hero-heavy pages, large displays |
| 1.500 | Perfect fifth | Very bold, large jumps | Poster layouts, minimal content |
| 1.618 | Golden ratio | Classical, organic | Artistic layouts, traditional print |

**Key insight:** The ratio determines how dramatic the size jumps are. A small ratio (1.125) gives you many similar sizes — good for data-dense UIs where hierarchy is subtle. A large ratio (1.5) gives you fewer, more distinct sizes — good for editorial layouts where hierarchy is dramatic.

## Type scale generation

From a base size and ratio, generate the full scale:

### Perfect Fourth (1.333) — Bold, editorial

```
Base: 16px

Scale:
  --s-xs:   0.750 × base = 12px    (captions, metadata)
  --s-sm:   0.875 × base = 14px    (secondary text, labels)
  --s-base: 1.000 × base = 16px    (body text)
  --s-md:   1.333 × base = 21px    (subheadings, emphasized body)
  --s-lg:   1.777 × base = 28px    (section headings)
  --s-xl:   2.369 × base = 38px    (page headings)
  --s-2xl:  3.157 × base = 50px    (hero headings)
  --s-3xl:  4.209 × base = 67px    (display text)
```

### Major Third (1.250) — Modern, balanced

```
Base: 16px

Scale:
  --s-xs:   0.640 × base = 10px    (tiny labels)
  --s-sm:   0.800 × base = 13px    (secondary text)
  --s-base: 1.000 × base = 16px    (body text)
  --s-md:   1.250 × base = 20px    (subheadings)
  --s-lg:   1.563 × base = 25px    (section headings)
  --s-xl:   1.953 × base = 31px    (page headings)
  --s-2xl:  2.441 × base = 39px    (hero headings)
  --s-3xl:  3.052 × base = 49px    (display text)
```

### Golden Ratio (1.618) — Classical, dramatic

```
Base: 16px

Scale:
  --s-sm:   0.618 × base = 10px    (captions)
  --s-base: 1.000 × base = 16px    (body text)
  --s-md:   1.618 × base = 26px    (subheadings)
  --s-lg:   2.618 × base = 42px    (page headings)
  --s-xl:   4.236 × base = 68px    (display text)
```

Note: Golden ratio produces very dramatic jumps. Only use when you need 3-4 very distinct levels.

## Space scale generation

The space scale should be derived from the same base as the type scale, using either the same ratio or a complementary system.

### Option A: Same ratio as type (recommended for consistency)

```
Using Perfect Fourth (1.333), base 16px:

  --sp-1:  0.500 × base = 8px     (tight: icon gaps, inline)
  --sp-2:  0.667 × base = 11px    (compact: label-to-input)
  --sp-3:  1.000 × base = 16px   (standard: form gaps, padding)
  --sp-4:  1.333 × base = 21px   (relaxed: between groups)
  --sp-5:  1.777 × base = 28px   (spacious: section dividers)
  --sp-6:  2.369 × base = 38px   (large: major sections)
  --sp-7:  3.157 × base = 50px   (generous: page padding)
  --sp-8:  4.209 × base = 67px   (grand: hero spacing)
```

### Option B: 4px grid (recommended for implementation simplicity)

```
Step-based, 4px increments:

  --sp-1:  4px    (tight: icon-to-text, badge padding)
  --sp-2:  8px    (compact: within-component)
  --sp-3:  12px   (snug: related items in a group)
  --sp-4:  16px   (standard: form fields, list items)
  --sp-5:  20px   (comfortable: between groups)
  --sp-6:  24px   (relaxed: between form sections)
  --sp-7:  32px   (spacious: section dividers)
  --sp-8:  48px   (large: major section breaks)
  --sp-9:  64px   (generous: page-level padding)
  --sp-10: 96px   (grand: hero sections)
```

**When to use which:**
- Option A (ratio-based) when the design is editorial, expressive, or the proportions need to be obviously harmonious
- Option B (step-based) when the design is functional, data-dense, or implementation simplicity matters more

## Container proportions

### Content width

The content area should have an intentional ratio to the page:

```
Common content widths and their character:

  38em (608px at 16px)    — Comfortable reading, focused content
  42em (672px)             — Standard reading width
  48em (768px)             — Relaxed reading, allows sidebars
  56em (896px)             — Wide layout, dashboard, multi-column
  72em (1152px)            — Full layout, data tables, wide grids
  
The content-to-page ratio should be intentional:
  - 2:3 content:page feels focused and editorial
  - 3:5 content:page feels balanced
  - 3:4 content:page feels spacious
```

### Tschichold's margin method

Jan Tschichold (the typographer) derived page margins from the content area itself:
- Inner margin: smallest (text needs to be close to the spine)
- Top margin: slightly larger (header breathing room)
- Outer margin: larger than inner (thumb room + visual balance)
- Bottom margin: largest (white space as a design element)

For a centered web layout, this translates to:
```
Page width:   100%
Content width: 60-70% of page (intentional ratio)
Side margins:  15-20% each (equal, or inner slightly less)
Top margin:    5-10% of viewport height
Bottom margin: 10-15% of viewport height (more than top)
```

### Grid proportions

For multi-column layouts:
```
2-column:  2:3 or 1:2 (sidebar:content) — not equal 1:1
3-column:  1:2:1 or 1:3:1 — center column dominates
4-column:  1:1:2:1 or asymmetric based on content hierarchy
```

Never use equal-width columns without a reason. Asymmetric column ratios create visual interest and direct attention.

## Type-space harmony

The most common proportional mistake: type scale and space scale don't relate.

### The alignment rule

If your type scale uses ratio 1.333 and your space scale uses 4px steps, check that the key alignment points match:

```
Type scale point:  --s-md: 21px
Space scale point:  --sp-5: 20px or 21px (near enough)

The "near enough" rule: type and space don't need exact matching,
but key reference points should align or be close.

Critical alignments:
  - Line height of body text should be a clean space scale value
    (if body is 16px/1.5 = 24px line height, space scale should include 24px)
  
  - Paragraph spacing should be 1 line height
    (if line height is 24px, paragraph margin = 24px)
  
  - Section spacing should be a multiple of line height  
    (if line height is 24px, section spacing = 48px or 72px)
```

### The vertical rhythm system

```
Base: 16px font-size, 1.5 line-height = 24px rhythm unit

All vertical spacing is a multiple of 24px:
  1 unit (24px):  paragraph spacing, related items
  1.5 units (36px): between groups  
  2 units (48px): between sections
  3 units (72px): major section breaks
  4 units (96px): page-level spacing
```

This is why the type scale ratio and space scale ratio should share a mathematical relationship — they both need to align to the same vertical grid.

## Implementation as tokens

```css
:root {
  /* Perfect Fourth scale (1.333) — both type and space derived from same ratio */
  
  /* Type scale */
  --s-xs:   0.75rem;    /* 12px */
  --s-sm:   0.875rem;   /* 14px */
  --s-base: 1rem;       /* 16px */
  --s-md:   1.333rem;   /* 21px */
  --s-lg:   1.777rem;   /* 28px */
  --s-xl:   2.369rem;   /* 38px */
  --s-2xl:  3.157rem;   /* 50px */
  --s-3xl:  4.209rem;   /* 67px */
  
  /* Space scale — derived from same base with intentional relationship */
  --sp-1:  0.5rem;     /* 8px   — tight */
  --sp-2:  0.667rem;   /* 11px  — compact */
  --sp-3:  1rem;       /* 16px  — standard */
  --sp-4:  1.333rem;   /* 21px  — relaxed */
  --sp-5:  1.777rem;   /* 28px  — spacious */
  --sp-6:  2.369rem;   /* 38px  — large */
  --sp-7:  3.157rem;   /* 50px  — generous */
  --sp-8:  4.209rem;   /* 67px  — grand */
  
  /* Line height aligned to space scale */
  --lh-tight:  1.15;  /* headings — close, but note: not a space value */
  --lh-body:   1.5;   /* body — 1.5 × 16px = 24px, matches sp-1.5 if we add it */
  --lh-relaxed: 1.6;  /* long-form reading */
  
  /* Container proportions */
  --content-width: 42rem;  /* 672px — intentional 2:3ish ratio to common viewports */
  --page-max: 72rem;       /* 1152px — full layout width */
}
```

## Proportional audit checklist

```
PROPORTIONS AUDIT

[ ] All font sizes map to the type scale (no magic numbers)
[ ] Type scale follows a named ratio (1.25, 1.333, 1.5, or golden)
[ ] Line heights decrease as font sizes increase
[ ] All spacing values map to the space scale (no magic numbers)
[ ] Space scale follows the same ratio as type OR a stated alternative with reasoning
[ ] Key alignment points between type and space are close or matched
[ ] Vertical rhythm is consistent (spacing is multiples of line-height)
[ ] Content width has an intentional proportion to page width
[ ] Column ratios are asymmetric unless equal columns are justified
[ ] Margins are derived, not arbitrary
[ ] Section spacing varies (not all sections have the same padding)
[ ] The ratio between spacing levels is at least 1.5:1 (not all similar)
```

## Anti-patterns I avoid

- Using arbitrary pixel values for font sizes (14px, 17px, 22px) — they don't relate to each other
- Using different ratios for type and space without documenting why — inconsistency looks accidental
- Equal spacing between all sections — hierarchy requires proportional spacing
- Equal-width columns without justification — asymmetry is more interesting
- Choosing a ratio because "it looks good" — ratios create mathematical harmony whether or not the viewer can articulate it
- Skipping container proportions — the empty space around content is as important as the content itself
- Using the golden ratio without understanding its character — it creates dramatic jumps, not subtle gradation
- 4px/8px spacing that doesn't align to the type scale — two independent systems create visual discord