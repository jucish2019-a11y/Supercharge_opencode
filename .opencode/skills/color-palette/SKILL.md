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
- Build palettes from color science (mood → hue → scheme), not vibes

## When to use me

Use this skill when:
- Choosing colors for a new project or rebrand
- Defining a design token color system
- Creating a dark mode color mapping
- Fixing contrast or color harmony issues
- Generating a palette from a brand color
- The existing palette feels generic or "AI-made"

## How I work

### Checker mode (auditing an existing palette)

1. **Check mood-hue alignment** — Does the palette's mood match the product's intended feeling? A "trustworthy financial tool" shouldn't have cyan-on-dark.
2. **Verify color wheel relationships** — Are the hues related by a named scheme (analogous, complementary, triadic), or are they random?
3. **Audit contrast** — Every text/background pair must meet WCAG AA (4.5:1 normal text, 3:1 large text).
4. **Check neutral temperature** — Are the grays consistently warm or cool? Mixed temperatures look muddy.
5. **Count hues** — How many distinct hues are in use? More than 2 (primary + accent) usually means chaos.
6. **Test for AI tells** — Cyan-on-dark, purple-to-blue gradients, neon accents on dark backgrounds.

### Applier mode (building from scratch)

1. **Start from mood** — What feeling should the product communicate? Mood drives hue.
2. **Choose the base hue** — Map mood to hue using the mood-hue mapping below.
3. **Derive the scheme** — Use a color wheel relationship (analogous, complementary, split-complementary).
4. **Generate the hue scale** — 10 steps from 50 to 950.
5. **Define the neutral scale** — Choose warm or cool temperature. Generate 10 steps.
6. **Map semantic tokens** — Assign meaning to every color.
7. **Verify contrast** — Every text/background pair must pass WCAG AA.

## Mood-to-hue mapping

Colors communicate feeling before content. Start from the mood the product needs:

| Mood | Hue direction | Example palette anchor |
|------|---------------|----------------------|
| Trust / Professional | Blue | #3B82F6 family |
| Energy / Action | Orange / Red | #F97316 / #EF4444 family |
| Growth / Health | Green | #22C55E family |
| Creative / Premium | Purple | #8B5CF6 family |
| Warm / Approachable | Amber / Terracotta | #D97706 / #C45D3E family |
| Calm / Reliable | Teal / Sage | #0D9488 / #6B7C6E family |
| Modern / Minimal | Near-black + single vibrant accent | #171717 + #22D3EE family |
| Luxurious / Refined | Deep neutrals + gold/bronze | #1C1917 + #B8860B family |

**Why this matters:** If you skip this step and go straight to "what looks good," you'll default to cyan-on-dark or purple-to-blue — the two most common AI-generated palettes. Mood → hue is intentional; vibe → color is convergent.

## Color wheel schemes

Once you have your base hue, derive the palette from a named color wheel relationship:

### Analogous (harmonious, unified)
```
Base: 30° (orange)
Neighbors: 15° (yellow-orange), 45° (red-orange)

Use when: You want a unified, subtle, calming palette.
Avoid when: You need contrast or distinct call-to-actions.
```

### Complementary (vibrant, high contrast)
```
Base: 210° (blue)
Complement: 30° (orange)

Use when: You need a strong accent that pops against the primary.
Avoid when: The contrast feels jarring (split-complementary is gentler).
```

### Split-complementary (balanced, versatile)
```
Base: 210° (blue)
Split: 30° (orange) and 50° (amber)

Use when: You want contrast without the tension of full complementary.
This is the most versatile scheme for product UIs.
```

### Triadic (diverse, playful)
```
Base: 210° (blue), 330° (pink), 90° (green)

Use when: You need 3 distinct colors with equal weight (data visualization).
Avoid when: You need a cohesive, focused UI — 3 hues is a lot.
```

## Hue scale generation

From the brand color, generate a 10-step scale (50–950):

```
50:  Very light (backgrounds, hover states)
100: Light (badges, highlights)
200: Light accent
300: Medium light (icons on dark)
400: Medium (decorative elements)
500: Base (primary color, borders, icons) ← your brand color goes here
600: Medium dark (hover over 500)
700: Dark (text on light backgrounds)
800: Darker (headings)
900: Very dark (deep accents)
950: Near black (extreme contrast)
```

**Generation method:** Start with your brand color at 500. For each step up (lighter), increase lightness while slightly reducing saturation. For each step down (darker), decrease lightness while maintaining or increasing saturation. The perceived brightness jump between steps should be consistent.

### Tools for generation
- Use HSL color space — adjust L (lightness) and S (saturation) for each step
- Or use a tool like Tailwind CSS color generator, Happy Hues, or Realtime Colors
- Verify each step against a neutral gray background — the steps should appear evenly spaced

## Neutral scale with temperature

Neutrals are 90% of a UI. Their temperature matters more than most people think:

### Warm neutrals (yellow/red tint)
```
0:   #FFFFFF  (base background)
50:  #FAFAF5  (warm subtle background)
100: #F5F5F0  (warm secondary background)
200: #E8E5DE  (warm borders)
300: #D4D0C8  (warm disabled)
400: #A8A29E  (warm placeholder)
500: #78716C  (warm secondary text)
600: #57534E  (warm body text dark mode)
700: #44403C  (warm primary text dark mode)
800: #292524  (warm dark surface)
900: #1C1917  (warm dark background)
950: #0C0A09  (warm deep dark)
```

### Cool neutrals (blue tint)
```
0:   #FFFFFF  (base background)
50:  #F8FAFC  (cool subtle background)
100: #F1F5F9  (cool secondary background)
200: #E2E8F0  (cool borders)
300: #CBD5E1  (cool disabled)
400: #94A3B8  (cool placeholder)
500: #64748B  (cool secondary text)
600: #475569  (cool body text dark mode)
700: #334155  (cool primary text dark mode)
800: #1E293B  (cool dark surface)
900: #0F172A  (cool dark background)
950: #020617  (cool deep dark)
```

**Why temperature matters:** Mixing warm and cool grays creates a "muddy" feeling. The entire neutral scale should share the same temperature direction. Pick warm for approachable/organic products, cool for technical/professional products.

## Semantic token mapping

```css
:root {
  /* Primary — your brand color */
  --color-primary:       brand-500;
  --color-primary-hover:  brand-600;
  --color-primary-active: brand-700;
  --color-primary-light:  brand-100;
  --color-primary-text:   brand-700;  /* on light backgrounds */
  
  /* Surfaces — background hierarchy */
  --color-bg-primary:     neutral-0;
  --color-bg-secondary:   neutral-50;
  --color-bg-tertiary:    neutral-100;
  --color-bg-inverse:     neutral-900;
  
  /* Text — readability hierarchy */
  --color-text-primary:   neutral-900;
  --color-text-secondary: neutral-500;
  --color-text-tertiary:  neutral-400;
  --color-text-inverse:   neutral-0;
  
  /* Borders */
  --color-border:         neutral-200;
  --color-border-focus:   brand-500;
  
  /* Semantic — meaning colors */
  --color-success:  green-600;
  --color-warning:  amber-500;
  --color-error:    red-600;
  --color-info:     blue-500;
}
```

## Dark mode mapping

```css
@media (prefers-color-scheme: dark) {
  :root {
    /* Surfaces — inverted hierarchy */
    --color-bg-primary:     #0F172A;  /* was neutral-0 → now deep dark */
    --color-bg-secondary:   #1E293B;  /* was neutral-50 → now dark surface */
    --color-bg-tertiary:    #334155;  /* was neutral-100 → now mid surface */
    --color-bg-inverse:     #F8FAFC;  /* was neutral-900 → now light */
    
    /* Text — inverted */
    --color-text-primary:   #F1F5F9;  /* light text on dark bg */
    --color-text-secondary: #94A3B8;  /* medium text */
    --color-text-tertiary:  #64748B;  /* subtle text */
    
    /* Borders — subtle on dark */
    --color-border:         #334155;
    
    /* Primary — desaturated for dark mode */
    --color-primary:       brand-400;  /* step lighter for visibility */
    --color-primary-hover:  brand-500;
    --color-primary-light:  brand-900;  /* deep tint for dark bg accents */
  }
}
```

**Dark mode rules:**
- Don't just invert — desaturate accent colors by 10-20% to avoid "glowing neon" effect
- Reduce contrast slightly from light mode (pure white on pure black causes eye strain)
- Use elevation to create surface hierarchy (lighter surfaces = higher elevation)
- Keep semantic colors (success/warning/error) but verify they contrast on dark backgrounds

## Hue-shifted shadows

Instead of pure black shadows (which look flat), shift shadow color toward the primary hue:

```css
/* Instead of: */
box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);

/* Use: */
box-shadow: 0 4px 12px rgba(59, 130, 246, 0.12); /* blue-shifted for blue primary */
box-shadow: 0 4px 12px rgba(220, 38, 38, 0.10); /* red-shifted for red primary */
box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);       /* neutral for non-opinionated */
```

**Why this matters:** Hue-shifted shadows feel more natural and integrated. Pure black shadows look harsh in warm palettes; pure black shadows look flat in cool palettes. A hint of color in the shadow ties it to the palette.

## Tonal palette and elevation system

Premium interfaces don't use drop shadows to indicate elevation. They use tonal shifts — lighter or darker steps within the same hue — to create depth hierarchies that feel natural and cohesive.

### Generating a 10-step tonal palette

For every role color, generate 10 tonal steps from lightest to darkest:

```
Step 5:  Near-white tint (backgrounds, surfaces on dark)
Step 10: Very light tint (surface-container-low)
Step 20: Light tint (surface-container)
Step 30: Soft tint (surface-container-high)
Step 40: Medium tint (borders on dark, muted text on light)
Step 50: Mid (disabled text, dividers)
Step 60: Medium shade (secondary text on dark)
Step 70: Dark shade (primary text on dark)
Step 80: Deep shade (headings on dark, borders on light)
Step 90: Near-black shade (text on light, dark backgrounds)
```

Use HSL manipulation: keep the same hue, adjust lightness in 10% increments, slightly reduce saturation as lightness increases to prevent washed-out tints.

### Elevation through tonal shifts (not shadows)

Instead of `box-shadow` for elevation, use surface tonal steps:

```
Level 0: surface             (base layer, e.g. #FFFFFF or neutral-50)
Level 1: surface-container-low   (+1 tonal step, e.g. neutral-100)
Level 2: surface-container       (+2 tonal steps, e.g. neutral-200)
Level 3: surface-container-high  (+3 tonal steps, e.g. neutral-300)
Level 4: surface-container-max   (+4 tonal steps, for highest elevation)
```

For dark mode, invert the direction:
```
Level 0: surface             (e.g. #121212 or neutral-900)
Level 1: surface-container-low   (neutral-800, slightly lighter)
Level 2: surface-container       (neutral-700)
Level 3: surface-container-high  (neutral-600)
Level 4: surface-container-max   (neutral-500)
```

Why this works: tonal elevation creates depth without the visual noise of shadows. It scales to any density or context. Material Design 3 uses this system, and it is the approach Stripe, Linear, and Vercel all use.

### Color as information architecture

Use color to mark structure, not just decorate:

- **Product categories**: Payments = blue, Billing = green, Connect = purple (Stripe's approach)
- **Navigation sections**: Dashboard = neutral, Settings = accent tint, Alerts = warning
- **Density levels**: Compact sections get a subtle background tint, comfortable sections stay on base surface
- **Interactive vs. static**: Interactive elements get primary color borders/backgrounds; static elements use neutrals

## Contextual color modes

Beyond light and dark, premium interfaces support contextual modes:

| Mode | Purpose | Key changes |
|------|---------|-------------|
| Light | Default daytime use | Full chromatic, standard contrast |
| Dark | Low-light, OLED-friendly | Recalculated tones (not inverted), desaturated accents |
| High contrast | WCAG AAA, visual impairment | Augmented borders, stronger text contrast (7:1+) |
| Reduced transparency | Difficulty with glass/blur effects | Replace `backdrop-filter: blur()` with solid surfaces |

Each mode remaps the same semantic tokens to different tonal values. The token names stay the same; only the values change.

## Quality checklist

- [ ] Palette built from mood → hue → scheme (not "what looks good")
- [ ] Neutral scale has consistent temperature (all warm or all cool)
- [ ] Maximum 2 hues in use (primary + accent)
- [ ] All text/background pairs meet WCAG AA contrast (4.5:1 for text, 3:1 for UI)
- [ ] Semantic tokens defined for all meanings (primary, success, error, warning, info)
- [ ] Dark mode palette mapped (not just inverted)
- [ ] Accent color is desaturated in dark mode (no neon glow)
- [ ] Shadows use hue-shifted color, not pure black
- [ ] Colorblind safety: no information conveyed by color alone (use icons + text too)
- [ ] No AI color tells: no cyan-on-dark, no purple-to-blue gradients, no neon accents on dark
- [ ] 10-step tonal palette generated for primary hue
- [ ] Elevation uses tonal shifts, not drop shadows
- [ ] Color marks structural information (categories, sections, density levels)
- [ ] Contextual modes defined (light, dark, high-contrast, reduced-transparency)

## Anti-patterns I avoid

- Starting with "what looks good" instead of mood → hue → scheme — leads to convergence
- Using more than 2 hues — creates visual noise; use neutrals and 1-2 accents
- Mixing warm and cool neutrals — creates a muddy, unprofessional feel
- Pure black shadows on warm palettes — looks flat and harsh
- Inverting colors for dark mode without desaturating — creates neon glow
- Relying on color alone for meaning — 8% of men are colorblind; use icons + text + color
- Cyan-on-dark or purple-to-blue gradients — the fastest AI color tells
- Using the brand color at full saturation for large backgrounds — overwhelming; use lighter tints
- Generating a palette from the brand color without checking contrast — many brand colors fail AA on white
- Using drop shadows for elevation instead of tonal shifts — premium interfaces use surface color, not shadows
- Using a single surface color for all containers — tonal variation creates depth and hierarchy
- Treating dark mode as a simple inversion — dark mode requires recalculated tones and desaturated accents