---
name: visual-qa
description: Pixel-perfect visual review — catch spacing inconsistencies, alignment drift, font rendering issues, cross-browser differences, color mismatches, and subtle visual defects that separate professional from amateur UIs
---

## What I do

I perform systematic visual quality assurance to catch the 1px-5px issues that make the difference between "looks good" and "looks professional":

- **Spacing audits** — Find inconsistent gaps, padding, and margin values across a UI
- **Alignment checks** — Verify elements align to the grid and to each other
- **Typography consistency** — Size, weight, line-height, and letter-spacing drift
- **Color accuracy** — Token compliance, contrast ratio verification, color drift
- **Cross-browser rendering** — Font rendering, sub-pixel differences, scrollbar behavior
- **Component state coverage** — Every element has hover, focus, active, disabled, loading, error, empty
- **Visual regression** — Detect unintended visual changes between versions

## When to use me

Use this skill when:
- A UI looks "slightly off" but you can't pinpoint why
- Reviewing a page or component for pixel-perfect quality before shipping
- Setting up visual regression testing
- Debugging cross-browser rendering differences
- Auditing an existing product for visual consistency
- Preparing a design system reference page

## How I work

1. **Screenshot the current state** — Capture the UI at 1x and 2x (retina) resolution.
2. **Overlay a grid** — Place an 8px grid overlay to check spacing alignment.
3. **Inspect by category** — Systematically check: spacing, alignment, typography, colors, borders, shadows, states.
4. **Compare against tokens** — Every value should match a design token. Flag any magic numbers.
5. **Test at all breakpoints** — Mobile, tablet, desktop.
6. **Test across browsers** — Chrome, Safari, Firefox, Edge (and their mobile equivalents).
7. **Generate a defect report** — List each issue with location, expected vs. actual, and severity.

## Spacing audit

### Method

1. Take a screenshot of the full page
2. Overlay an 8px grid
3. Measure every gap between elements
4. Check: does every measurement map to a spacing token?

### Common spacing defects

| Defect | Symptom | Cause | Fix |
|--------|---------|-------|-----|
| **Inconsistent gap** | Two adjacent sections have 15px gap vs 16px gap | Hardcoded values | Replace with `var(--space-4)` |
| **Double margin** | Gap between two elements is 32px instead of 16px | Both elements have `margin-bottom: 16px` | Use margin on one side only (bottom), or use gap |
| **Misaligned grid** | Elements don't start at the same x-position | Inconsistent left padding | Use grid system or same padding token |
| **Orphan spacing** | A 13px, 17px, or 19px value appears | Developer eyeballed the spacing | Map to the nearest token (12, 16, 20) |
| **Compounding gaps** | Gaps grow larger where components nest | Each wrapper adds its own padding | Remove redundant wrapper padding |
| **Asymmetric padding** | 16px left, 20px right in the same container | One side accounts for scrollbar, other doesn't | Use `scrollbar-gutter: stable` or equal padding |
| **Tight nesting** | Inner card has same padding as outer card | No spacing scale reduction for depth | Inner padding = outer padding × 0.75 (e.g., 24 → 16) |

### Spacing measurement checklist

Check these specific measurements on every page:

```
Page edge to content:
  -- Desktop: var(--space-16) or var(--space-24) (64 or 96px)
  -- Mobile: var(--space-6) or var(--space-8) (24 or 32px)

Content to section heading:
  -- Between sections: var(--space-12) or var(--space-16) (48 or 64px)

Between adjacent elements:
  -- Within a group: var(--space-2) to var(--space-4) (8-16px)
  -- Between groups: var(--space-6) to var(--space-8) (24-32px)

Card/container inner padding:
  -- Default: var(--space-6) or var(--space-8) (24 or 32px)
  -- Compact: var(--space-4) (16px)
  -- Tight: var(--space-3) (12px)
```

## Alignment audit

### Horizontal alignment

```
Check on every page:
  □ All left edges of content align to the same column or grid line
  □ All right edges of full-width sections end at the same point
  □ Labels and inputs in forms align vertically
  □ Icons within buttons align vertically with button text
  □ Avatar + name pairs align horizontally
  □ Table column headers align with column data
  
Common alignment issues:
  □ Icon + text baseline mismatch (icon is optically centered but not aligned to text baseline)
     Fix: use `align-items: baseline` for text, `center` for icon alignment,
     or apply -1px to -2px vertical adjust on icon via `translateY`
  □ "More items" count aligned differently than item text
     Fix: Ensure count badge uses the same baseline alignment
  
  □ Card titles jump position between cards (different avatar sizes)
     Fix: Set fixed height for the title row, center-align vertically
```

### Vertical alignment

```
Check on every page:
  □ All section headings start at the same Y position relative to their content
  □ Navigation items are vertically centered in the header
  □ Form rows (label + input + error) maintain consistent height
  □ List items are the same height for the same content density
  □ Footer content aligns at the bottom consistently
  
Common vertical issues:
  □ Form rows with errors are taller than rows without errors
     Fix: Reserve space for the error message (always show the row, hide with visibility: hidden)
  □ List items without secondary text are shorter than items with it
     Fix: Set a min-height on list items (64px for two-line, 48px for single-line)
  □ Icons vertically misaligned in a vertical list
     Fix: Use the same alignment method (center or baseline) consistently
```

### Optical alignment

Geometric alignment ≠ optical alignment. The eye perceives some shapes as misaligned even when mathematically centered:

```
Shapes that need optical adjustment:
  □ Triangles (carets, arrows): shift 1-2px toward the base
  □ Circular icons in square containers: center is correct
  □ Text next to icons: use baseline alignment, not center
  □ Capital letters at the start of lines: no adjustment needed
  □ Descenders (g, p, q, y): may cause perceived misalignment in centered text
  □ Numbers in data tables: use tabular figures (font-variant-numeric: tabular-nums)

Test: Squint your eyes. If something looks off, it needs optical adjustment.
```

## Typography audit

### Size consistency

```
Check on every page:
  □ Page title: same size everywhere it appears (--text-2xl or --text-3xl)
  □ Section headings: same size (--text-xl or --text-lg)
  □ Body text: same size (--text-base or --text-sm depending on density)
  □ Caption/metadata: same size (--text-xs)
  
Common issues:
  □ "This heading looks bigger" — actually it's bold vs semibold, same size
  □ "These two lines are different sizes" — actually different line-heights, same font-size
  □ Font-size drift: the same component uses 14px in one place, 13.5px in another
     Fix: Check for compounding font-size (child with 0.875rem inside parent with 0.875rem)
```

### Weight consistency

```
Check on every page:
  □ All headings use the same weight (600 or 700, pick one)
  □ All body text uses the same weight (400)
  □ All emphasized text uses the same weight (500 or 600)
  □ No accidental weight changes due to @font-face not loading
  
Common issues:
  □ "Bold text looks normal" — web font didn't load, fallback doesn't have that weight
     Fix: Check Network tab for font loading errors, check font-display: swap
  □ "Weight looks different in Safari vs Chrome" — different font rendering
     Fix: Use -webkit-font-smoothing: antialiased on Safari for consistency
  □ "This text looks bolder" — color contrast makes weight appear different
     Fix: Use the same weight, but color creates the perceived difference (intentional)
```

### Line-height consistency

```
Check on every page:
  □ Body text: same line-height (1.5 or 1.625)
  □ Headings: same line-height per level (tight: 1.2-1.3)
  □ Button text: same line-height (usually 1.0 or implicit from height)
  □ No text overlap due to insufficient line-height
  
Common issues:
  □ Line-height: 1 applied to body text — descenders overlap ascenders
     Fix: minimum line-height: 1.25 for any readable text
  □ Different line-heights on the same text between pages
     Fix: Ensure line-height is defined in the typography token, not per-component
```

## Color audit

### Token compliance

```
For every element on the page:
  □ Background color is a --color-bg-* token (not a raw hex)
  □ Text color is a --color-text-* token (not a raw hex)
  □ Border color is a --color-border* token
  □ Primary color is --color-primary (not --color-blue-500 in component code)
  
How to audit:
  1. Open DevTools
  2. Search for #[0-9a-fA-F]{3,6} in all stylesheets
  3. Any raw hex in component CSS = violation
  4. Search for rgb(, hsl(, rgba( — same check
  5. Allowed: rgba(0,0,0,0.04) for shadows (opacity variation of black is acceptable)
```

### Contrast ratio verification

```
WCAG AA requirements:
  □ Normal text (< 18px or < 14px bold): 4.5:1 minimum
  □ Large text (≥ 18px or ≥ 14px bold): 3:1 minimum
  □ UI components and graphical objects: 3:1 minimum
  □ Focus indicators: 3:1 against adjacent colors

Test with:
  □ Chrome DevTools: color picker shows contrast ratio
  □ axe DevTools extension: automated contrast checks
  □ WebAIM Contrast Checker: manual verification
  □ Stark: Figma/browser plugin for contrast

Common failures:
  □ --text-tertiary on --bg-primary (often barely passes)
  □ Placeholder text (too light)
  □ Disabled text (by definition low contrast — ensure "disabled" is apparent from other attributes too)
  □ White text on --color-primary (check the specific brand color — many fail)
  □ Links in body text (must be distinguishable from surrounding text by more than color alone)
```

### Color drift

```
Color drift happens when the "same" color is slightly different:
  □ --color-primary is #3B82F6 but a component still uses #3B9AF6
  □ Border color is rgba(0,0,0,0.06) in one place, rgba(0,0,0,0.08) in another
  □ Shadow color: rgba(0,0,0,0.1) in one card, rgba(0,0,0,0.08) in the next

Audit method:
  1. Screenshot the page
  2. Use a color picker on every colored element
  3. Compare against the token values
  4. Flag any value that doesn't match its token
```

## Cross-browser rendering differences

### Font rendering

| Browser | Render engine | Font quirks |
|---------|--------------|-------------|
| Chrome (Windows) | DirectWrite | Slightly bolder, wider | 
| Safari (macOS) | Core Text | Thinner, more refined |
| Firefox (Windows) | DirectWrite | Similar to Chrome |
| Safari (iOS) | Core Text | Slightly different metrics than macOS |
| Chrome (Android) | Skia | Varies by device and font stack |

```
Common font issues:
  □ Text wraps to 2 lines in Safari but 1 line in Chrome (different width calculations)
     Fix: Use percentage-based widths or max-width with overflow handling
  □ Font appears bolder in Chrome on Windows
     Fix: -webkit-font-smoothing: antialiased (Safari), -moz-osx-font-smoothing: grayscale (Firefox)
  □ Line height differs by 1-2px between browsers
     Fix: Set explicit line-height in px or rem, never "normal"
  □ Custom font doesn't load in Firefox (CORS issue)
     Fix: Add Access-Control-Allow-Origin header on font server
```

### Sub-pixel rendering

```
Browsers handle sub-pixel positioning differently:
  □ Chrome: rounds to full pixels (sharper but can be 1px off)
  □ Firefox: renders at sub-pixel level (more accurate but can appear blurry)
  □ Safari: depends on transform (GPU vs CPU compositing)

Fix: 
  □ Use whole pixel values for borders and positioned elements
  □ Add translateZ(0) or will-change: transform to force GPU compositing
  □ Avoid fractional pixel values for 1px borders
  □ Use outline instead of border for focus rings (outline doesn't affect layout)
```

### Scrollbar behavior

```
| Browser | Default scrollbar | Overlay? | Width |
|---------|-------------------|----------|-------|
| Chrome (Windows) | Classic | No | 17px |
| Chrome (macOS) | Overlay | Yes | 0px (appears on scroll) |
| Firefox (Windows) | Classic | No | 17px |
| Safari (macOS) | Overlay | Yes | 0px |

Issues:
  □ Content shifts 17px left when scrollbar appears (Windows)
     Fix: scrollbar-gutter: stable (modern browsers) or always show scrollbar
  □ Overlay scrollbar covers content (macOS)
     Fix: Add right padding equal to scrollbar width on scrollable containers
  □ Custom scrollbar styling doesn't work in Firefox
     Fix: Use scrollbar-width: thin and scrollbar-color for Firefox
```

### Form element differences

```
| Element | Chrome | Safari | Firefox |
|---------|--------|--------|---------|
| Select dropdown | Custom rendering | System rendering, z-index issues | Custom |
| Date input | Calendar picker | Text input (no picker in older) | Partial |
| Color input | Color picker | Color wheel | Color picker |
| Range input | Different track/thumb | Different track/thumb | Different |
| File input | "Choose File" | "Choose File" | "Browse..." |

Fix:
  Always apply a CSS reset for form elements, then define custom styles.
  Never trust browser defaults for form elements.
```

## Component state coverage audit

For every interactive element on the page, verify these states exist and are visually distinct:

### Button states

| State | Visual | How to trigger |
|-------|--------|---------------|
| Default | Brand-colored background, white text | — |
| Hover | Darker background OR slight shadow lift | Mouse hover |
| Focus | 2px outline ring, offset 2px | Tab to focus |
| Active | Slightly darker than hover, or scale(0.97) | Mousedown |
| Disabled | 50% opacity, not-allowed cursor | :disabled attribute |
| Loading | Spinner replaces or prefixes text, button is non-interactive | loading state |
| Icon button | Same states, but no text — needs aria-label | — |

```
Common state defects:
  □ Focus ring is the same color as the button (invisible)
     Fix: Use a contrasting outline color (white ring on brand button, or use outline-offset)
  □ Disabled button looks identical to enabled (just lower opacity)
     Fix: Also change to gray background + gray text + cursor: not-allowed
  □ No hover state on mobile (irrelevant) — remove hover-only styles with @media (hover: hover)
  □ Focus-visible shows on mouse click (shouldn't)
     Fix: Use :focus-visible instead of :focus for ring styles
```

### Input states

| State | Visual | How to trigger |
|-------|--------|---------------|
| Default | Border color --border, bg --bg-primary | — |
| Placeholder | Text in --text-tertiary | Empty input |
| Filled | Normal text | Has value |
| Focus | Border --color-primary, optional ring | Tab or click |
| Error | Border --color-error, error text below | Invalid value |
| Disabled | Background --color-disabled, lower opacity | :disabled |
| Read-only | Same as default but no cursor change | readonly |

### Card states

| State | Visual | How to trigger |
|-------|--------|---------------|
| Default | Flat bg, subtle border | — |
| Hover | Shadow lift + translateY(-2px) | Mouse hover |
| Active/pressed | Shadow flatten + scale(0.98) | Mousedown |
| Selected | Primary border, checkmark | Selected |
| Disabled | Opacity 0.6, not-allowed | Disabled |
| Loading | Skeleton shimmer overlay | Data loading |
| Empty | Placeholder content | No data |

## Visual regression testing

### Approach

```
1. Capture reference screenshots ("baseline") of every page and component
2. On every change, capture new screenshots ("comparison")
3. Pixel-diff the comparison against baseline
4. Flag any difference > threshold (usually 0.1% or 1px)
5. Review flagged differences: intentional change vs. regression
6. Update baseline if change is intentional
```

### Tools

| Tool | Type | Best for |
|------|------|----------|
| Chromatic | Cloud service | Storybook component visual testing |
| Percy | Cloud service | Page-level visual testing in CI |
| Playwright | Library | Screenshot comparison in E2E tests |
| Cypress | Plugin | Visual regression during E2E |
| BackstopJS | Standalone | Independent visual regression |
| reg-suit | CLI | CI integration with cloud storage |

### What to test

```
Always capture:
  □ Every page at desktop (1440px) and mobile (375px)
  □ Every component in Storybook (all variants and states)
  □ Dark mode versions of the above
  □ Empty states
  □ Error states
  □ Long-content/overflow states
  
Skip:
  □ Third-party embeds (dynamic content)
  □ Ads (they always change)
  □ Timestamps and dates (use frozen time for screenshots)
  □ Loading states with spinners (non-deterministic)
```

### Handling flaky tests

```
Common causes of false-positive diffs:
  □ Font rendering differences between CI and local
     Fix: Install the same fonts on CI; or increase threshold for text-containing areas
  □ Anti-aliasing differences between renders
     Fix: Increase diff threshold to 0.1% 
  □ Scroll position differences
     Fix: Scroll to top before capturing, use full-page screenshots
  □ Animation frame capture at different points
     Fix: Disable animations before capturing (add .no-animation class)
  □ Dynamic content (ads, random images)
     Fix: Stub dynamic content with fixtures before capture
```

## Pixel-perfect checklist

Run this checklist before shipping any page or component:

### Spacing
- [ ] All gaps map to spacing tokens (no magic numbers)
- [ ] No inconsistent gaps between the same type of elements
- [ ] Padding follows the nesting rule (inner < outer)
- [ ] Page edge padding is consistent across all pages
- [ ] No double-margin issues (use gap instead of margin)

### Alignment
- [ ] Left edges align to the grid
- [ ] Icons are optically aligned with text (not mathematically centered)
- [ ] Form labels and inputs align vertically
- [ ] Table columns align consistently
- [ ] No 1px misalignment from sub-pixel rendering

### Typography
- [ ] All font sizes use type scale tokens
- [ ] All font weights are consistent per hierarchy level
- [ ] Line heights are consistent per text level
- [ ] Tabular figures on all numeric data
- [ ] Font renders correctly in all target browsers

### Color
- [ ] All colors reference design tokens (no raw hex)
- [ ] Contrast ratios meet WCAG AA (4.5:1 for text, 3:1 for UI)
- [ ] Colors are consistent across light and dark themes
- [ ] Semantically colored elements (success/error) use the correct tokens
- [ ] No color drift (slightly different shades of the same token)

### Borders and shadows
- [ ] Border radius is consistent per element type (cards, buttons, inputs)
- [ ] Border color is consistent (uses --color-border)
- [ ] Shadow values are consistent per elevation level
- [ ] No 1px dark borders where a subtle border should be used
- [ ] Box shadow renders identically across browsers

### States
- [ ] Every interactive element has hover, focus, active, disabled states
- [ ] Focus rings use :focus-visible (not :focus)
- [ ] Focus ring color contrasts with the element background
- [ ] Disabled elements have both visual AND behavioral indicators
- [ ] Loading states exist for all async operations
- [ ] Empty states exist for all data-driven elements

### Responsiveness
- [ ] Page looks correct at 375px, 768px, 1024px, 1440px
- [ ] Text doesn't overflow containers at any width
- [ ] Touch targets are minimum 44px on mobile
- [ ] Navigation adjusts properly at each breakpoint
- [ ] No horizontal scrollbar (unless intentional for tables)

### Cross-browser
- [ ] Checked in Chrome, Safari, Firefox (desktop + mobile)
- [ ] Font rendering is acceptable in all browsers
- [ ] Scrollbar behavior doesn't cause layout shift
- [ ] Form elements have consistent styling
- [ ] No browser-specific visual bugs

## Anti-patterns I avoid

- Shipping without testing focus states — they're invisible in normal use but critical for accessibility
- Ignoring 1px inconsistencies — they compound and create a "something's off" feeling
- Testing only in Chrome — 30% of users are on Safari/Firefox
- Not testing empty and error states — they're 50% of user experience
- Relying on manual visual checks only — automated visual regression catches what eyes miss
- Using `!important` to fix visual issues — fix the root cause, not the symptom
- Skipping retina/2x testing — everything sharp at 1x can look blurry at 2x
- Not freezing time/dynamic content for screenshots — causes flaky visual regression tests
- Ignoring scrollbar layout differences — 17px offset breaks alignment on Windows
- Using `outline: none` without a replacement focus style — removes keyboard accessibility