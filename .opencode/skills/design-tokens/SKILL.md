---
name: design-tokens
description: Architect token systems that scale — naming conventions, multi-theme resolution, token-to-code pipelines, platform-specific outputs, and governance for design-to-engineering handoff at scale
---

## What I do

I architect design token systems that bridge design and engineering at scale:

- **Token architecture** — Three-tier hierarchy: global → alias → component
- **Naming conventions** — Consistent, searchable, predictable token names
- **Multi-theme resolution** — Light, dark, high-contrast, brand variants from one token set
- **Token-to-code pipelines** — Automated generation of CSS variables, Tailwind, iOS, Android, Figma
- **Platform outputs** — CSS custom properties, SCSS maps, JS/TS objects, Swift, Kotlin, JSON
- **Governance** — How to add, modify, and deprecate tokens without breaking consumers

## When to use me

Use this skill when:
- Setting up a new design token system for a product or design system
- Scaling an existing token system across multiple themes or platforms
- Building a token pipeline that generates CSS, iOS, and Android outputs
- Defining naming conventions and governance for tokens
- Adding dark mode, high-contrast mode, or brand variant themes
- Migrating from hardcoded values to token-based design

## How I work

1. **Audit existing values** — Extract every color, spacing, font size, radius, and shadow currently in use. Identify duplicates and near-duplicates.
2. **Define the token hierarchy** — Three tiers: global, alias, component.
3. **Establish naming conventions** — Consistent, predictable, machine-friendly.
4. **Build multi-theme support** — Semantic tokens that resolve differently per theme.
5. **Set up the build pipeline** — Source of truth → transformed outputs for each platform.
6. **Create governance** — Rules for when and how to add, change, or remove tokens.

## Token architecture

### Three-tier hierarchy

```
┌─────────────────────────────────────────┐
│ GLOBAL TOKENS (raw values)              │
│ The complete palette — every possible   │
│ value. These never change per theme.     │
│                                         │
│ --color-blue-500: #3B82F6              │
│ --space-4: 4px                          │
│ --font-size-14: 14px                    │
│ --radius-8: 8px                         │
└──────────────┬──────────────────────────┘
               │ references
┌──────────────▼──────────────────────────┐
│ ALIAS TOKENS (semantic meaning)          │
│ Map raw values to purpose.              │
│ These CAN change per theme.             │
│                                         │
│ --color-primary: var(--color-blue-500) │
│ --color-bg-surface: var(--neutral-0)    │
│ --space-padding-md: var(--space-16)     │
│ --radius-card: var(--radius-8)         │
└──────────────┬──────────────────────────┘
               │ references
┌──────────────▼──────────────────────────┐
│ COMPONENT TOKENS (specific use)          │
│ Override-able, scoped to a component.   │
│                                         │
│ --button-bg: var(--color-primary)      │
│ --button-radius: var(--radius-6)       │
│ --button-padding: var(--space-12) var(… │
│ --card-shadow: var(--shadow-md)        │
└─────────────────────────────────────────┘
```

**Rule**: Global tokens are the palette. Alias tokens are the meaning. Component tokens are the application. Never reference global tokens from components — always go through alias tokens.

### Why three tiers?

| Tier | Changes per theme? | Changes per component? | Who uses it? |
|------|--------------------|-----------------------|-------------|
| Global | Never | Never | Token pipeline, alias token definitions |
| Alias | Yes (light vs dark) | Never | Component token definitions, component code |
| Component | Yes (per-variant) | Yes | Component implementation code |

If you skip alias tokens, changing from light to dark requires changing every component token individually. With alias tokens, you change 30 alias tokens and every component adapts.

## Token naming conventions

### Syntax

```
--{category}-{property}-{variant}-{state}-{scale}
```

### Categories

| Category | Prefix | Examples |
|----------|--------|---------|
| Color | `color` | `--color-primary`, `--color-bg-surface` |
| Spacing | `space` | `--space-4`, `--space-padding-md` |
| Typography | `font` | `--font-size-base`, `--font-weight-semibold` |
| Border radius | `radius` | `--radius-sm`, `--radius-card` |
| Shadow | `shadow` | `--shadow-sm`, `--shadow-card` |
| Opacity | `opacity` | `--opacity-disabled`, `--opacity-hover` |
| Motion | `motion` | `--motion-duration-normal`, `--motion-ease-standard` |
| Z-index | `z` | `--z-dropdown`, `--z-modal` |
| Breakpoint | `breakpoint` | `--breakpoint-md`, `--breakpoint-lg` |

### Naming rules

1. **Lowercase kebab-case** — `--color-text-primary` not `--colorTextPrimary` or `--color_text_primary`
2. **No abbreviations** — `--color-background` not `--color-bg` (except in alias tokens where brevity matters — `bg` is acceptable as an alias shorthand)
3. **Property-first** — `--font-size-base` not `--base-font-size`
4. **Scale is numeric** — `--space-4` not `--space-xs` (numbers scale better than names)
5. **Semantic over literal** — `--color-primary` not `--color-blue-500` in alias and component tokens
6. **No component names in alias tokens** — `--color-bg-surface` not `--color-card-bg` (alias tokens are shared)
7. **Component prefix for component tokens** — `--button-bg` not `--bg` (those are scoped)

### Naming at each tier

**Global tokens** (raw + category + scale):
```
--color-blue-500
--color-neutral-100
--space-4
--space-16
--font-size-14
--radius-8
```

**Alias tokens** (category + semantic-meaning):
```
--color-primary
--color-bg-primary
--color-bg-secondary
--color-text-primary
--color-text-secondary
--color-border
--space-gap-sm
--space-gap-md
--space-padding-sm
--space-padding-md
--font-size-body
--font-size-heading
--radius-interactive
--radius-container
```

**Component tokens** (component + property):
```
--button-bg
--button-bg-hover
--button-bg-active
--button-color
--button-radius
--button-padding-x
--button-padding-y
--card-bg
--card-border-color
--card-radius
--card-padding
--card-shadow
--input-bg
--input-border-color
--input-border-color-focus
--input-radius
--input-padding
```

## Multi-theme resolution

### Themes

A theme is a set of alias token values. Global tokens never change. Alias tokens map to different global tokens per theme.

```css
/* ─── Global tokens (never change) ─── */
:root {
  --color-blue-500: #3B82F6;
  --color-blue-600: #2563EB;
  --color-neutral-0: #FFFFFF;
  --color-neutral-50: #F9FAFB;
  --color-neutral-900: #171717;
  --color-neutral-800: #1F2937;
  --color-neutral-100: #F3F4F6;
  --color-neutral-200: #E5E7EB;
  --color-neutral-500: #6B7280;
  --color-neutral-400: #9CA3AF;
  --color-neutral-700: #374151;
  --color-neutral-600: #4B5563;
  --color-neutral-950: #0A0A0A;
}

/* ─── Alias tokens: Light theme ─── */
:root,
[data-theme="light"] {
  --color-primary:          var(--color-blue-500);
  --color-primary-hover:    var(--color-blue-600);
  --color-bg-primary:       var(--color-neutral-0);
  --color-bg-secondary:     var(--color-neutral-50);
  --color-bg-tertiary:      var(--color-neutral-100);
  --color-text-primary:     var(--color-neutral-900);
  --color-text-secondary:   var(--color-neutral-500);
  --color-text-tertiary:    var(--color-neutral-400);
  --color-border:           var(--color-neutral-200);
  --color-border-strong:    var(--color-neutral-400);
  --color-surface-raised:   var(--color-neutral-0);
  --shadow-card:            0 1px 3px rgba(0,0,0,0.08);
  --shadow-card-hover:      0 4px 12px rgba(0,0,0,0.12);
}

/* ─── Alias tokens: Dark theme ─── */
[data-theme="dark"] {
  --color-primary:          var(--color-blue-500);
  --color-primary-hover:    var(--color-blue-400);
  --color-bg-primary:       var(--color-neutral-950);
  --color-bg-secondary:     var(--color-neutral-900);
  --color-bg-tertiary:      var(--color-neutral-800);
  --color-text-primary:     var(--color-neutral-50);
  --color-text-secondary:   var(--color-neutral-400);
  --color-text-tertiary:    var(--color-neutral-600);
  --color-border:           rgba(255,255,255,0.06);
  --color-border-strong:    rgba(255,255,255,0.15);
  --color-surface-raised:   var(--color-neutral-800);
  --shadow-card:            0 1px 3px rgba(0,0,0,0.3);
  --shadow-card-hover:      0 4px 12px rgba(0,0,0,0.4);
}

/* ─── Alias tokens: High contrast ─── */
[data-theme="high-contrast"] {
  --color-primary:          #0050FF;
  --color-primary-hover:    #003FCC;
  --color-bg-primary:       #FFFFFF;
  --color-bg-secondary:     #FFFFFF;
  --color-text-primary:     #000000;
  --color-text-secondary:   #1A1A1A;
  --color-text-tertiary:    #333333;
  --color-border:           #000000;
  --color-border-strong:    #000000;
  --shadow-card:            none;
  --shadow-card-hover:      0 0 0 2px #000000;
}
```

### Brand variants

If multiple brands share the system:

```css
/* Brand A uses blue */
[data-brand="a"] {
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-600);
}

/* Brand B uses purple */
[data-brand="b"] {
  --color-primary: var(--color-purple-500);
  --color-primary-hover: var(--color-purple-600);
}
```

## Complete token set reference

### Color tokens

**Semantic alias tokens (always use these in components):**
```
--color-primary                  Primary brand color
--color-primary-hover            Primary hover state
--color-primary-active           Primary active/pressed state
--color-primary-subtle           Primary at low opacity (badges, highlights)
--color-secondary                Secondary accent
--color-secondary-hover
--color-bg-primary               Page background
--color-bg-secondary             Card/sidebar background
--color-bg-tertiary              Nested surface background
--color-bg-inverse               Inverted background (for inverted text)
--color-text-primary             High-emphasis text
--color-text-secondary           Medium-emphasis text
--color-text-tertiary            Low-emphasis text
--color-text-inverse             Text on inverted backgrounds
--color-text-on-primary          Text on primary color backgrounds
--color-border                   Default borders
--color-border-strong            Emphasized borders
--color-success                  Success semantic
--color-success-subtle           Success at low opacity
--color-warning                  Warning semantic
--color-warning-subtle           Warning at low opacity
--color-error                    Error semantic
--color-error-subtle             Error at low opacity
--color-info                     Info semantic
--color-info-subtle              Info at low opacity
--color-overlay                  Modal/backdrop overlay
--color-skeleton                 Loading skeleton shimmer
--color-disabled                 Disabled element background
--color-selection                Text selection highlight
```

### Spacing tokens

```
--space-0:    0
--space-1:    4px
--space-2:    8px
--space-3:    12px
--space-4:    16px
--space-5:    20px
--space-6:    24px
--space-8:    32px
--space-10:   40px
--space-12:   48px
--space-16:   64px
--space-20:   80px
--space-24:   96px
```

### Typography tokens

```
--font-family-sans:      'Inter', system-ui, -apple-system, sans-serif
--font-family-serif:     'Source Serif', Georgia, serif
--font-family-mono:      'JetBrains Mono', ui-monospace, monospace

--font-size-xs:          0.75rem    /* 12px */
--font-size-sm:          0.875rem   /* 14px */
--font-size-base:        1rem       /* 16px */
--font-size-lg:          1.125rem   /* 18px */
--font-size-xl:          1.25rem    /* 20px */
--font-size-2xl:         1.5rem     /* 24px */
--font-size-3xl:         1.875rem   /* 30px */
--font-size-4xl:         2.25rem    /* 36px */

--font-weight-regular:   400
--font-weight-medium:    500
--font-weight-semibold:  600
--font-weight-bold:      700

--line-height-none:      1
--line-height-tight:     1.25
--line-height-snug:      1.375
--line-height-normal:    1.5
--line-height-relaxed:   1.625
--line-height-loose:     2

--letter-spacing-tighter: -0.02em
--letter-spacing-tight:   -0.01em
--letter-spacing-normal:   0
--letter-spacing-wide:     0.025em
--letter-spacing-wider:    0.05em
```

### Border radius tokens

```
--radius-none:   0
--radius-sm:     4px
--radius-md:     6px
--radius-lg:     8px
--radius-xl:     12px
--radius-2xl:    16px
--radius-3xl:    24px
--radius-full:   9999px
```

### Shadow tokens

```
--shadow-xs:    0 1px 2px rgba(0,0,0,0.04)
--shadow-sm:    0 1px 3px rgba(0,0,0,0.08)
--shadow-md:    0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.04)
--shadow-lg:    0 10px 15px rgba(0,0,0,0.08), 0 4px 6px rgba(0,0,0,0.04)
--shadow-xl:    0 20px 25px rgba(0,0,0,0.1), 0 8px 10px rgba(0,0,0,0.04)
--shadow-inner: inset 0 2px 4px rgba(0,0,0,0.04)
```

### Motion tokens

```
--motion-duration-instant:  50ms
--motion-duration-fast:    100ms
--motion-duration-normal:  200ms
--motion-duration-moderate: 300ms
--motion-duration-slow:    500ms

--motion-ease-standard:    cubic-bezier(0.4, 0, 0.2, 1)
--motion-ease-decelerate:  cubic-bezier(0, 0, 0.2, 1)
--motion-ease-accelerate:  cubic-bezier(0.4, 0, 1, 1)
--motion-ease-sharp:       cubic-bezier(0.4, 0, 0.6, 1)
--motion-ease-spring:      cubic-bezier(0.175, 0.885, 0.32, 1.275)
```

### Z-index tokens

```
--z-base:       0
--z-dropdown:   100
--z-sticky:     200
--z-overlay:    300
--z-modal:      400
--z-popover:    500
--z-toast:      600
--z-tooltip:    700
```

### Breakpoint tokens

```
--breakpoint-sm:  640px
--breakpoint-md:  768px
--breakpoint-lg:  1024px
--breakpoint-xl:  1280px
--breakpoint-2xl: 1536px
```

## Token pipeline

### Source of truth

Tokens live in a single source format and are transformed into platform outputs:

```
tokens/
  color.json        ← Source of truth
  spacing.json
  typography.json
  radii.json
  shadows.json
  motion.json
  z-index.json
  breakpoints.json
```

### Source format (JSON)

```json
{
  "color": {
    "blue": {
      "500": { "$value": "#3B82F6", "$type": "color" },
      "600": { "$value": "#2563EB", "$type": "color" }
    },
    "primary": {
      "$value": "{color.blue.500}",
      "$type": "color",
      "$description": "Primary brand color"
    },
    "background": {
      "primary": {
        "$value": "{neutral.0}",
        "$type": "color",
        "$description": "Main page background"
      }
    }
  }
}
```

### Platform transforms

| Platform | Output format | Tool |
|----------|--------------|------|
| CSS | Custom properties in `:root` | Style Dictionary, Tokens Studio |
| SCSS | `$token-name: value` maps | Style Dictionary |
| Tailwind | `extend` config in `tailwind.config.js` | Custom transform |
| TypeScript | `export const tokens = { ... }` | Style Dictionary |
| iOS (Swift) | `static let colorPrimary = UIColor(...)` | Style Dictionary |
| Android (Kotlin) | `val colorPrimary = Color(...)` | Style Dictionary |
| Figma | Variables via Tokens Studio plugin | Tokens Studio |

### Build pipeline (Style Dictionary)

```js
// config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'dist/css/',
      files: [{
        destination: 'tokens.css',
        format: 'css/variables',
        options: { outputReferences: true }
      }]
    },
    js: {
      transformGroup: 'js',
      buildPath: 'dist/js/',
      files: [{
        destination: 'tokens.js',
        format: 'javascript/es6'
      }]
    },
    ios: {
      transformGroup: 'ios-swift-separate',
      buildPath: 'dist/ios/',
      files: [{
        destination: 'ColorTokens.swift',
        format: 'ios-swift/any',
        className: 'ColorTokens',
        type: 'color'
      }]
    },
    android: {
      transformGroup: 'android',
      buildPath: 'dist/android/',
      files: [{
        destination: 'colors.xml',
        format: 'android/colors'
      }]
    }
  }
};
```

## Token governance

### Adding a new token

1. **Is there an existing token?** Search the token set. 80% of "new token" requests are satisfied by an existing one.
2. **Is it a global or alias token?** If it's a new raw value → global. If it's a new semantic mapping → alias. If it's component-specific → component.
3. **Does it follow naming rules?** Lowercase kebab-case, no abbreviations in global tokens, property-first.
4. **Is it in the scale?** Spacing must use the 4px scale. Colors must use existing hue scales. Typography must use the type scale. No off-scale values.
5. **Does it have a description?** Every token needs a `$description` explaining its purpose.
6. **Is it themed?** Every alias token must have values in all supported themes (light, dark, high-contrast).

### Modifying a token

1. **What consumes it?** Find all references. A global token change affects every alias token that references it.
2. **Is it a breaking change?** If the semantic meaning changes (e.g., `--color-primary` changes from blue to purple), it's breaking. Announce and version it.
3. **Is it a value adjustment?** If only the value changes not the meaning (e.g., `--color-primary` goes from `blue-500` to `blue-600`), it's non-breaking. Deploy and verify.
4. **Deprecate, don't delete.** Mark old token as deprecated in documentation. Keep it resolving for 2 major versions. Remove in the third.

### Deprecation process

```
Version N:     Token is active
Version N+1:   Token marked deprecated, new token recommended
               Warning logged if deprecated token is used (dev mode)
Version N+2:   Token removed, throws error if referenced
```

## Quality checklist

- [ ] Every token has a `$type` and `$description`
- [ ] Global tokens are separate from alias tokens (no mixing)
- [ ] Components reference alias tokens, never global tokens directly
- [ ] Naming follows consistent convention (lowercase kebab-case, property-first)
- [ ] Every alias token has a value in all supported themes
- [ ] Token pipeline generates outputs for all target platforms
- [ ] Spacing tokens follow the 4px scale (no off-scale values)
- [ ] Color tokens reference the full hue scale, not raw hex values
- [ ] No duplicate tokens (same value, different names)
- [ ] No unused tokens (tokens with zero references)
- [ ] Token deprecation follows the N+2 version process
- [ ] CSS output uses `outputReferences: true` (shows token references, not just values)
- [ ] Breakpoint tokens are available as both CSS and JS (for runtime queries)
- [ ] Motion tokens cover all easing curves and durations used in the product

## Anti-patterns I avoid

- Mixing global and alias tokens in the same CSS variable namespace — keep them separate
- Components referencing global tokens directly — always go through alias tokens
- Adding tokens outside the scale (e.g., --space-7) — stay on the 4px grid
- Token names that describe the value not the purpose (--color-blue not --color-primary)
- Duplicate tokens with different names (--spacing-md and --gap-md both being 16px)
- Hardcoded values in component code when a token exists
- Creating component tokens when an alias token is sufficient
- Generating CSS without `outputReferences` — you lose traceability
- Skipping dark mode/high-contrast when adding new alias tokens
- Deleting tokens without a deprecation period — consumers will break