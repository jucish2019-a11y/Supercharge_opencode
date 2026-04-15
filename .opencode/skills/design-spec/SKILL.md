---
name: design-spec
description: Generate production-ready design specifications from requirements — detailed enough for pixel-perfect implementation with all states, tokens, and edge cases
---

## What I do

I produce complete design specifications that leave zero ambiguity for implementation — the level of detail Stitch uses to generate pixel-perfect UI:

- **Visual specification** — Exact values for every property: spacing, color, typography, radii, shadows
- **State catalog** — Every element's hover, focus, active, disabled, loading, error, empty state
- **Layout specification** — Grid, flexbox, and responsive behavior at every breakpoint
- **Interaction specification** — Transitions, animations, timing, choreography
- **Edge cases** — Overflow, truncation, min/max content, loading, error, empty, disabled
- **Token mapping** — Every value references a design token, never magic numbers

## When to use me

Use this skill when:
- Building a new page, screen, or feature and you need a complete spec before coding
- Handing off design intent to someone else (or your future self)
- The implementation must be pixel-perfect and cover all states
- You want to generate a comprehensive "design document" for a UI

## How I work

1. **Gather requirements** — What is this UI for? Who uses it? What actions can they take? What data does it show?
2. **Define the layout structure** — Break the page into regions (header, sidebar, main content, footer). Identify the grid/flex structure.
3. **Specify every component** — For each component, define tokens, states, sizing, and behavior.
4. **Define all states** — Default, hover, focus, active, disabled, loading, error, empty, success.
5. **Specify interactions** — Transitions, animations, hover effects, click responses.
6. **Handle edge cases** — Long text, no data, many items, slow network, permissions.
7. **Reference tokens** — Every value maps to a design token. No magic numbers.

## Output format

Every design spec follows this exact structure:

```markdown
## [Component/Page Name]

### Purpose
[1-2 sentences: what this does and why]

### Layout
- Container: [max-width, padding, background token]
- Grid/flex: [structure]
- Responsive: [behavior at each breakpoint]

### Typography
- Heading: [token] — [weight], [size], [line-height], [color token]
- Body: [token] — [weight], [size], [line-height], [color token]
- Caption: [token] — [weight], [size], [line-height], [color token]

### Spacing
- Section gap: [token] ([value])
- Inner padding: [token] ([value])
- Element gap: [token] ([value])

### States

#### [Component Name]
| State | Background | Border | Text Color | Shadow | Other |
|-------|-----------|--------|-----------|--------|-------|
| Default | --bg-primary | --border | --text-primary | none | |
| Hover | --bg-secondary | --border | --text-primary | --shadow-sm | |
| Active | --bg-tertiary | --border-strong | --text-primary | none | scale(0.97) |
| Focus | --bg-primary | --color-primary | --text-primary | none | 2px outline |
| Disabled | --bg-primary | --border | --text-tertiary | none | opacity: 0.5 |
| Loading | --bg-primary | --border | --text-tertiary | none | spinner |

### Interactions
- Hover transition: [duration] [easing]
- Click transition: [duration] [easing]
- Enter animation: [description] [duration] [easing]
- Exit animation: [description] [duration] [easing]

### Edge Cases
- Long text: [truncation strategy]
- Empty state: [illustration + message + CTA]
- Error state: [inline message + retry]
- Loading: [skeleton pattern]
- Many items: [virtualization/pagination]

### Accessibility
- [ARIA labels]
- [Keyboard navigation]
- [Focus order]
- [Screen reader announcements]
```

## Example: Card Component Spec

### Purpose
Display a summary of an item with a primary action. Used in grid layouts for project listings.

### Layout
- Container: max-width none, padding `--space-24` (24px), background `--bg-primary`
- Border: 1px solid `--border`, radius `--radius-lg` (12px)
- Shadow: `0 1px 3px rgba(0,0,0,0.08)`
- Grid: vertical flex, gap `--space-16` (16px)

### Typography
- Card title: `--text-lg` (20px), `--weight-semibold` (600), `--leading-tight` (1.25), color `--text-primary`
- Card description: `--text-sm` (14px), `--weight-regular` (400), `--leading-normal` (1.5), color `--text-secondary`
- Card metadata: `--text-xs` (12px), `--weight-regular` (400), `--leading-normal` (1.5), color `--text-tertiary`

### Spacing
- Card padding: `--space-24` (24px)
- Section gap (title → description): `--space-8` (8px)
- Section gap (description → metadata): `--space-16` (16px)
- Section gap (metadata → action): `--space-24` (24px)
- Actions gap (between buttons): `--space-12` (12px)

### States

| State | Container | Shadow | Transform | Other |
|-------|-----------|--------|-----------|-------|
| Default | bg-primary, border | 0 1px 3px rgba(0,0,0,0.08) | none | |
| Hover | bg-primary, border | 0 4px 12px rgba(0,0,0,0.1) | translateY(-2px) | cursor: pointer |
| Active | bg-secondary | 0 1px 2px rgba(0,0,0,0.05) | scale(0.98) | |
| Focus-within | bg-primary | 0 1px 3px rgba(0,0,0,0.08) | none | 2px ring --color-primary |
| Disabled | bg-tertiary | none | none | opacity: 0.6, cursor: not-allowed |

### Interactions
- Hover: transition `200ms ease-out`, shadow lifts, card raises 2px
- Active: transition `100ms ease`, scale down to 0.98, shadow flattens
- Focus-within: 2px ring `--color-primary` with `2px` offset
- Entry: fade in + translateY(8px) → 0, `300ms ease-decelerate`

### Edge Cases
- Title > 2 lines: truncate with ellipsis, `--weight-semibold` on first line only
- Description > 3 lines: truncate with "Show more" inline link
- No description: skip section, reduce spacing to `--space-8`
- No image: show placeholder icon in accent color at 40% opacity
- Many cards (>12): use pagination, not infinite scroll

### Accessibility
- Card is an `<article>` with `aria-label` from card title
- Interactive cards use `<a>` or `<button>` as the title element
- Focus ring visible on keyboard nav, hidden on mouse click
- Card content is in logical reading order for screen readers

## Spec quality checklist

- [ ] Every color is a token, not a hex value
- [ ] Every spacing value is a token, not a pixel value
- [ ] Every font size/weight/line-height is a token
- [ ] All interactive states are specified (hover, focus, active, disabled)
- [ ] All data states are specified (empty, loading, error, populated)
- [ ] Edge cases are addressed (long text, no data, overflow)
- [ ] Transitions have duration and easing specified
- [ ] Animation choreography is defined (stagger, cascade)
- [ ] Accessibility requirements are listed
- [ ] Responsive behavior at each breakpoint is specified

## Anti-patterns I avoid

- Specs with only the "happy path" — every state must be designed
- Magic numbers — every value must be a token reference
- Vague descriptions like "nice shadow" — specify exact values
- Missing responsive behavior — must define layout at every breakpoint
- Assuming implementation details — spec the what, not the how
- Skipping empty/error/loading states — these are 50% of user experience
- Specs without accessibility — if it's not accessible, it's not done