---
name: dark-mode
description: Design beautiful dual-theme UIs with proper contrast, elevation, and semantic color mapping
---

## What I do

I design and implement dual-theme (light/dark) UIs that look beautiful in both modes:

- Create semantic color systems that map to both themes
- Handle elevation and surface hierarchy in dark mode
- Ensure proper contrast ratios in both themes
- Implement smooth theme switching

## When to use me

Use this skill when:
- Adding dark mode to an existing light-only UI
- Building a new app that must support both themes
- Fixing contrast or readability issues in dark mode
- Implementing theme toggle and persistence

## How I work

1. **Map semantic colors** — Define every color as a semantic token with light and dark values:

   | Token | Light | Dark |
   |---|---|---|
   | `--bg-primary` | #FFFFFF | #0A0A0A |
   | `--bg-secondary` | #F5F5F5 | #141414 |
   | `--bg-tertiary` | #EBEBEB | #1E1E1E |
   | `--text-primary` | #171717 | #F5F5F5 |
   | `--text-secondary` | #525252 | #A3A3A3 |
   | `--text-tertiary` | #A3A3A3 | #666666 |
   | `--border` | rgba(0,0,0,0.06) | rgba(255,255,255,0.06) |
   | `--border-strong` | rgba(0,0,0,0.15) | rgba(255,255,255,0.15) |

2. **Handle elevation in dark mode** — In dark mode, elevation is expressed through lighter surfaces, not darker shadows:
   - Level 0 (base): `#0A0A0A`
   - Level 1 (card): `#141414`
   - Level 2 (raised card): `#1A1A1A`
   - Level 3 (modal/popover): `#222222`
   - Level 4 (dialog/alert): `#2A2A2A`

   Shadow in dark mode should be very subtle: `0 2px 8px rgba(0,0,0,0.4)`.

3. **Desaturate accent colors** — Bright, saturated colors cause eye strain on dark backgrounds. For dark mode:
   - Reduce saturation of primary colors by 10-20%
   - Use slightly softer variants of brand colors
   - Increase lightness of text on dark backgrounds less than you'd think — the contrast is already there

4. **Adjust images and media** — Reduce brightness and increase warmth on images in dark mode using CSS: `filter: brightness(0.9)`. Consider dimming or adding a slight overlay.

5. **Implement the toggle** — Use `data-theme` attribute on `<html>`, persist to `localStorage`, respect `prefers-color-scheme`.

## CSS implementation

```css
:root, [data-theme="light"] {
  --bg-primary: #FFFFFF;
  --bg-secondary: #F5F5F5;
  --text-primary: #171717;
  --border: rgba(0,0,0,0.06);
}

[data-theme="dark"] {
  --bg-primary: #0A0A0A;
  --bg-secondary: #141414;
  --text-primary: #F5F5F5;
  --border: rgba(255,255,255,0.06);
}

/* Transition */
body {
  transition: background-color 0.2s, color 0.2s;
}
```

## Guidelines

- Never use absolute black (#000000) for backgrounds — use near-black (#0A0A0A to #141414)
- Never use absolute white (#FFFFFF) for text on dark — use off-white (#F5F5F5)
- Semantic tokens only in component CSS — never raw colors
- Test contrast ratios: 4.5:1 for normal text, 3:1 for large text (WCAG AA)
- Adjust images and borders for dark mode — they shouldn't look washed out
- Respect the user's system preference on first visit, then persist their manual choice
- Avoid large areas of pure saturated color in dark mode