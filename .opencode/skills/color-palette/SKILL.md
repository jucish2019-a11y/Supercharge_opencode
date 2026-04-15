---
name: color-palette
description: Generate cohesive color systems with primaries, accents, surfaces, and semantic colors for beautiful UIs
---

## What I do

I design cohesive color palettes and systems for UIs:

- Generate full-hue palettes with proper contrast
- Create semantic color mappings for UI elements
- Define surface and background hierarchies
- Ensure accessibility across light and dark themes

## When to use me

Use this skill when:
- Choosing colors for a new project or rebrand
- Defining a design token color system
- Creating a dark mode color mapping
- Fixing contrast or color harmony issues
- Generating a palette from a brand color

## How I work

1. **Start with the brand color** — If a brand color is given, build from it. If not, choose based on the product personality:
   - **Trust/professional**: Blue (#3B82F6 family)
   - **Energy/action**: Orange/Red (#F97316 / #EF4444 family)
   - **Growth/health**: Green (#22C55E family)
   - **Creative/premium**: Purple (#8B5CF6 family)
   - **Modern/minimal**: Near-black with a vibrant accent

2. **Generate the hue scale** — From the brand color, generate a 10-step scale (50–950):
   ```
   50:  Very light (backgrounds, hover states)
   100: Light (badges, highlights)
   200: Light accent
   300: Medium light (icons on dark)
   400: Medium (decorative elements)
   500: Base (primary color, borders, icons)
   600: Medium dark (hover over 500)
   700: Dark (text on light backgrounds)
   800: Darker (headings)
   900: Very dark (deep accents)
   950: Near black (extreme contrast)
   ```

   The 500 should be the brand color. Lighter steps increase lightness, darker steps increase darkness. Maintain consistent perceived brightness jumps between steps.

3. **Define neutral scale** — Create a gray scale from white to near-black:
   ```
   0:   #FFFFFF  (base light background)
   50:  #FAFAFA  (subtle background)
   100: #F5F5F5  (secondary background)
   200: #E5E5E5  (borders, dividers)
   300: #D4D4D4  (disabled borders)
   400: #A3A3A3  (placeholder text)
   500: #737373  (secondary text)
   600: #525252  (secondary text dark mode)
   700: #404040  (primary text dark mode)
   800: #262626  (dark surfaces)
   900: #171717  (dark background)
   950: #0A0A0A  (deep dark background)
   ```

4. **Map to semantic tokens** — Assign meaning to colors:
   ```
   --color-primary:     brand-500
   --color-primary-hover: brand-600
   --color-bg-primary:  neutral-0
   --color-bg-secondary: neutral-50
   --color-bg-tertiary: neutral-100
   --color-text-primary:  neutral-900
   --color-text-secondary: neutral-500
   --color-border:      neutral-200
   --color-success:     green-500
   --color-warning:     amber-500
   --color-error:       red-500
   --color-info:        blue-500
   ```

5. **Verify contrast** — Check all text/background combinations meet WCAG AA (4.5:1 for normal text, 3:1 for large text).

## Color harmony rules

- **Limit hues**: 1 primary + 1 accent is enough. Most beautiful UIs use 1 dominant color.
- **Use neutrals generously**: 90% of a UI should be neutral. Color is for emphasis only.
- **Warm vs cool**: Pick a temperature direction. Warm grays (slight yellow/red) or cool grays (slight blue).
- **Consistent saturation**: All accent colors should have similar saturation levels.
- **Test on real content**: Colors look different in large areas vs small elements.

## Guidelines

- Always generate the full scale, not just one shade
- Never use raw colors in component CSS — always use semantic tokens
- Test palettes on real UI layouts, not just color swatches
- Ensure semantic colors (success/error/warning) work for colorblind users — never rely on color alone
- The neutral scale matters more than the accent color for a professional look