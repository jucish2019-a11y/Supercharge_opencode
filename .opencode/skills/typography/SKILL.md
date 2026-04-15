---
name: typography
description: Master type systems — font pairing, modular scales, responsive sizing, vertical rhythm, andOpenType features for publication-quality UI typography
---

## What I do

I design type systems that make interfaces readable, hierarchically clear, and visually refined:

- **Font selection** — Pair display and body fonts for personality and readability
- **Modular type scales** — Mathematically consistent sizing that creates visual harmony
- **Vertical rhythm** — Line heights and spacing that create a consistent vertical grid
- **Responsive typography** — Fluid sizing that adapts across viewports
- **OpenType features** — Ligatures, tabular figures, fractions, and other fine-tuning

## When to use me

Use this skill when:
- Choosing fonts or setting up a type system for a new project
- Typography feels inconsistent, cramped, or hard to read
- Creating a type scale that works across mobile to desktop
- Fonts aren't pairing well or hierarchy isn't clear
- Setting up CSS custom properties or design tokens for typography

## How I work

1. **Define typographic personality** — What feeling should the type communicate? This drives font selection.
   - **Trustworthy/Professional**: Inter, IBM Plex Sans, Source Sans, system stack
   - **Modern/Clean**: Plus Jakarta Sans, General Sans, Manrope
   - **Warm/Friendly**: Nunito, Softens, DM Sans
   - **Editorial/Serious**: Source Serif, Lora, Newsreader for headings; system sans for body
   - **Technical/Mono**: JetBrains Mono, Fira Code, IBM Plex Mono

2. **Select and pair fonts** — Maximum 2 font families per project:
   - **Option A (safest)**: One variable font with weight range 400-700. Use weight + size for hierarchy.
   - **Option B (personality)**: One display font for headings + one neutral font for body.
   - **Option C (editorial)**: Serif for headings + sans-serif for body. Classic and sophisticated.
   - Never use 3+ font families — it creates visual noise.

3. **Build the type scale** — Choose a ratio and generate consistent sizes:

   **Major Third (1.25) — Modern apps, dashboards:**
   ```
   --text-xs:    0.64rem  (10.24px)  — captions, metadata
   --text-sm:    0.8rem   (12.8px)   — secondary text, labels
   --text-base:  1rem     (16px)     — body text
   --text-lg:    1.25rem  (20px)     — emphasized body
   --text-xl:    1.563rem (25px)     — section headings
   --text-2xl:   1.953rem (31.25px)  — page headings
   --text-3xl:   2.441rem (39.06px) — hero headings
   --text-4xl:   3.052rem (48.83px) — display text
   ```

   **Perfect Fourth (1.333) — Bold, editorial:**
   ```
   --text-xs:    0.75rem  (12px)     — captions
   --text-sm:    0.875rem (14px)     — secondary text
   --text-base:  1rem     (16px)     — body
   --text-lg:    1.333rem (21.33px)  — subheadings
   --text-xl:    1.777rem (28.43px)  — section headings
   --text-2xl:   2.369rem (37.9px)   — page headings
   --text-3xl:   3.157rem (50.5px)   — hero headings
   ```

4. **Set line heights** — Line height must decrease as font size increases:
   ```
   --leading-none:   1.0     -- display text, large headings
   --leading-tight:  1.2     -- h2, h3
   --leading-snug:   1.35    -- h4, h5, h6
   --leading-normal: 1.5     -- body text
   --leading-relaxed: 1.65   -- long-form body, documentation
   --leading-loose:  1.8     -- sparse content, poetry
   ```

5. **Establish vertical rhythm** — All spacing should be multiples of the base line height:
   - Base line height: 24px (16px × 1.5)
   - Paragraph spacing: 24px (1 line)
   - Section spacing: 48px (2 lines)
   - Margins and padding: multiples of 4px, aligned to the 24px grid

6. **Implement with CSS custom properties** — Generate tokens for size, weight, line-height, and letter-spacing together.

## Typography tokens

```css
:root {
  /* Font families */
  --font-display: 'Plus Jakarta Sans', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* Size scale */
  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.125rem;
  --text-xl:   1.25rem;
  --text-2xl:  1.5rem;
  --text-3xl:  1.875rem;
  --text-4xl:  2.25rem;

  /* Line heights */
  --leading-none: 1;
  --leading-tight: 1.25;
  --leading-snug: 1.375;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;

  /* Weights */
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;

  /* Letter spacing — tighter as size increases */
  --tracking-tighter: -0.02em;
  --tracking-tight: -0.01em;
  --tracking-normal: 0;
  --tracking-wide: 0.025em;
}
```

## Font loading strategy

Performance-critical — do not skip this:

```css
/* 1. Preconnect to font origin */
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

/* 2. Use font-display: swap */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
}

/* 3. Define system font stack as fallback */
--font-body: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

- Use `font-display: swap` — never block rendering for fonts
- Subset fonts to only needed characters if self-hosting
- Use variable fonts (one file, multiple weights) when available
- Preconnect to font CDN for faster loading
- Ensure the system font fallback has similar metrics to the web font (minimize CLS)

## OpenType features

Enable these for polished typography:

```css
body {
  font-feature-settings: 'kern' 1, 'liga' 1, 'calt' 1;
}

/* Tabular figures for numbers in tables, prices, data */
.tabular-nums {
  font-feature-settings: 'tnum' 1;
  font-variant-numeric: tabular-nums;
}

/* Old-style figures for body text (lowercase-height numbers) */
.oldstyle-nums {
  font-feature-settings: 'onum' 1;
}
```

## Responsive typography

```css
/* Fluid type with clamp — scales between mobile and desktop */
html {
  font-size: clamp(16px, 0.9375rem + 0.25vw, 18px);
}

h1 {
  font-size: clamp(2rem, 1.5rem + 2.5vw, 3.5rem);
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tighter);
}
```

## Font pairing rules

| If your body is... | Your heading can be... | Because... |
|---|---|---|
| Humanist sans (Inter, Source Sans) | Geometric sans (Plus Jakarta, Manrope) | Contrast in structure |
| Geometric sans (DM Sans, Manrope) | Same font, bold weight | Clean, unified |
| System stack | Any web font | System body is invisible, any heading works |
| Serif (Source Serif, Lora) | Clean sans (Inter, system) | Professional contrast |
| Mono (JetBrains Mono) | Same mono, different weight | Technical aesthetic |

## Quality checklist

- [ ] Maximum 2 font families loaded
- [ ] Type scale uses a consistent ratio (1.25 or 1.333)
- [ ] Line heights decrease as sizes increase
- [ ] Letter spacing tightens as sizes increase
- [ ] All sizes defined as design tokens, never magic numbers
- [ ] System font fallback matches web font metrics
- [ ] `font-display: swap` on all `@font-face` declarations
- [ ] Body text: 16px minimum, 60-75 characters per line
- [ ] Tabular figures on all numeric data columns
- [ ] Headings have consistent weight (one weight for all headings)

## Anti-patterns I avoid

- More than 2 font families — kills performance and cohesion
- Using `!important` on font properties — override at the token level
- Body text below 16px — unreadable on mobile
- Line lengths over 75 characters — too wide for comfortable reading
- Using `letter-spacing: 0.5px` on body text — typewriter effect
- Mixing too many weights — stick to 2 (regular + semibold/bold)
- Skipping font-display: swap — causes FOIT (flash of invisible text)
- Decorative fonts for body text — readability always wins