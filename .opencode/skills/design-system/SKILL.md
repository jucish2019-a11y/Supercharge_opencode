---
name: design-system
description: Build design token systems, component libraries, and style guides for consistent UI at scale
---

## What I do

I build design systems that ensure visual consistency across an entire application:

- Define design tokens (colors, spacing, typography, radii, shadows)
- Create reusable component patterns with documented variants
- Establish naming conventions and composition rules
- Generate style guides that teams and AI agents can follow

## When to use me

Use this skill when:
- Starting a new project that needs a consistent look
- An existing project has inconsistent styling (different button styles, spacing, colors)
- Building a component library or UI kit
- Defining or migrating to CSS custom properties / design tokens
- Creating documentation for how UI should look and behave

## How I work

1. **Audit what exists** — Collect all colors, font sizes, spacing values, shadows, and border radii currently in use. Identify duplicates and inconsistencies.
2. **Define tokens** — Create a token hierarchy:

   **Global tokens** (raw values):
   ```
   --color-blue-500: #3B82F6;
   --space-4: 4px;
   --font-size-14: 14px;
   --radius-6: 6px;
   ```

   **Alias tokens** (semantic meaning):
   ```
   --color-primary: var(--color-blue-500);
   --color-bg-surface: var(--color-neutral-0);
   --color-text-primary: var(--color-neutral-900);
   --space-padding-md: var(--space-16);
   --radius-card: var(--radius-8);
   ```

   **Component tokens** (specific use):
   ```
   --button-bg: var(--color-primary);
   --button-radius: var(--radius-6);
   --button-padding: var(--space-12) var(--space-24);
   ```

3. **Document the scale** — Define consistent scales:
   - **Spacing**: 4, 8, 12, 16, 24, 32, 48, 64 (8px base grid with 4px increments)
   - **Typography**: 12, 14, 16, 20, 24, 32, 40 (modular scale ~1.25)
   - **Color**: Full palette for each hue (50-950) + semantic mappings
   - **Radius**: 0, 4, 6, 8, 12, 16, 24, full
   - **Shadow**: sm, md, lg, xl (consistent elevation metaphor)
   - **Opacity**: 0, 4, 8, 12, 16, 24, 32, 48, 64, 80, 100 (percentage steps)

4. **Define component patterns** — For each component, document:
   - Variants (primary, secondary, ghost, danger)
   - Sizes (sm, md, lg)
   - States (default, hover, active, focus, disabled, loading)
   - Composition rules (what can go inside, what wraps it)

5. **Create the style guide** — Generate a reference page showing every token and component variant.

## Token file structure

```
tokens/
  colors.css      -- color palette and semantic mappings
  spacing.css     -- spacing scale
  typography.css   -- font families, sizes, weights, line heights
  radii.css        -- border radius scale
  shadows.css      -- elevation/shadow scale
  motion.css       -- transitions, durations, easings
```

## Guidelines

- Never use raw hex values or magic numbers — always reference tokens
- Prefer semantic aliases over global tokens in component code
- Keep the token set small and opinionated — add new tokens deliberately, not freely
- Every token should have a name that explains its purpose, not its value
- Document the "why" behind each decision, not just the "what"
- Test tokens in both light and dark themes before finalizing