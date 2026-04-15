---
name: iconography
description: Design consistent icon systems — grid sizing, stroke weights, optical adjustments, naming conventions, and implementation for scalable icon libraries
---

## What I do

I design and implement icon systems that are visually consistent, pixel-perfect, and scalable:

- **Icon grid system** — Consistent bounding boxes, alignment, and optical sizing
- **Stroke weight consistency** — Matching line widths across the entire set
- **Optical adjustments** — Adjusting icons that appear off-center due to visual weight
- **Naming conventions** — Semantic, consistent, searchable icon names
- **Implementation** — SVG optimization, sprite sheets, React components, Figma export

## When to use me

Use this skill when:
- Creating an icon set or icon component library
- Icons look inconsistent in size, weight, or alignment
- Building icon usage guidelines for a design system
- Implementing SVG icons in a web application
- Deciding between icon libraries (Lucide, Phosphor, Heroicons) vs custom icons

## How I work

1. **Choose an icon style** — Match the brand personality:
   - **Outlined (recommended)**: 1.5px stroke, clean, modern. Default for most apps. (Lucide, Phosphor regular)
   - **Filled**: Solid shapes, high visibility, good for small sizes and dense UIs. (Material Symbols filled)
   - **Duotone**: Two-tone, layered depth. Premium feel, good for empty states and illustrations. (Phosphor duotone)
   - **Rounded**: Soft corners on paths. Friendly, approachable. (Heroicons outline)
   - Pick ONE style. Never mix outlined and filled icons in the same context.

2. **Define the grid system**:
   - **Base size**: 24x24px canvas (most versatile)
   - **Content area**: 20x20px (2px padding on each side for stroke overhang)
   - **Optical center**: The effective visual center, not the geometric center
   - **Stroke width**: 1.5px for default (outlined style)
   - **Corner radius**: 2px for small icons, rounded caps on strokes

3. **Establish sizing scale**:
   ```
   --icon-xs:   16px  — inline with text, tight spaces
   --icon-sm:   20px  — buttons, inputs, compact UI
   --icon-md:   24px  — default size, most UI elements
   --icon-lg:   32px  — empty states, feature highlights
   --icon-xl:   48px  — hero illustrations, onboarding
   ```

4. **Set stroke weight per size** (optical consistency):
   - 16px icons: 1px stroke (thinner at small size)
   - 20px icons: 1.25px stroke
   - 24px icons: 1.5px stroke (default)
   - 32px icons: 2px stroke
   - 48px icons: 2.5px stroke

5. **Apply optical adjustments** — Icons that are geometrically centered often look off-center:
   - Triangles and arrows pointing down: shift 0.5-1px down
   - Circles: center is correct
   - Objects with more mass on top (lock, pin): shift weight down
   - When in doubt, squint your eyes — if it looks off, adjust it

6. **Implement icons as components** — SVG as React components with size and color props.

## Icon naming convention

Use `[category]/[name]-[variant]` pattern:

```
arrows/
  arrow-left.svg
  arrow-right.svg
  arrow-up.svg
  chevron-down.svg
  chevron-right.svg

communication/
  mail.svg
  message-circle.svg
  phone.svg

editor/
  pencil.svg
  trash.svg
  copy.svg
  check.svg

navigation/
  home.svg
  menu.svg
  search.svg
  x.svg (close)

media/
  play.svg
  pause.svg
  volume.svg
  image.svg

status/
  check-circle.svg
  alert-triangle.svg
  info.svg
  x-circle.svg
```

Rules:
- Use kebab-case for file names
- Name by what the icon IS, not what it's USED FOR (`pencil` not `edit`)
- Exception: icons with established semantic meaning (`search`, `close`, `menu`)
- Suffix variants: `-filled`, `-outline`, `-duotone`

## SVG implementation

### As React components (recommended)

```tsx
interface IconProps {
  size?: 16 | 20 | 24 | 32 | 48;
  color?: string;
  className?: string;
}

function Icon({ size = 24, color = 'currentColor', className, children }: IconProps) {
  return (
    <svg
      width={size}
      height={size}
      viewBox="0 0 24 24"
      fill="none"
      stroke={color}
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      className={className}
    >
      {children}
    </svg>
  );
}
```

### SVG optimization rules

1. Remove all metadata, editor data, and comments
2. Convert styles to attributes
3. Round decimals to 1 place maximum
4. Remove invisible paths (opacity="0", fill="none" strokes)
5. Remove unused definitions
6. Prefer `currentColor` over hardcoded fills
7. Use `strokeLinecap="round"` and `strokeLinejoin="round"` for consistency
8. Keep paths as absolute coordinates (easier to read)
9. ViewBox should always be `0 0 24 24` for the base size
10. Never use `<use>` or `<symbol>` in component icons — inline the path

## Icon usage rules

### In buttons
- Icon + text: 8px gap between icon and label
- Icon-only button: 32px minimum tap target, `aria-label` required
- Icon position: left of text (standard), right of text (navigation/overflow)
- Size matches text: 16px icons with 14px text, 20px icons with 16px text

### In lists and tables
- 16-20px icons aligned to text baseline
- Use consistent alignment: left column for status icons, right for actions
- Status icons: colored circles or check/x marks, not decorative

### In navigation
- 20-24px icons in sidebar navigation
- Icon above or left of label (consistent across the entire nav)
- Active state: filled icon variant + accent color
- Inactive state: outlined icon variant + secondary color

### Empty states and illustrations
- 48-64px icons in empty states
- Use duotone or filled style for visual weight
- Pair with heading + description + CTA button

## Choosing an icon library

| Library | Style | Size | Stroke | Best for |
|---------|-------|------|--------|----------|
| Lucide | Outlined | 24px | 1.5px | Most apps, clean & modern |
| Phosphor | 6 variants | 24px | 1.5px | Flexible, has duotone |
| Heroicons | Outlined/Solid | 24px | 1.5/2px | Tailwind projects |
| Tabler Icons | Outlined | 24px | 2px | Dense, line-heavy UIs |
| Material Symbols | Outlined/Filled/Rounded | Variable | Variable | Material Design apps |

When not to use a library:
- You need brand-specific icons (logo, product-specific metaphors)
- The library is missing icons you need and you'd have a mix of styles
- You need pixel-level control over small (16px) sizes

## Quality checklist

- [ ] All icons on 24x24 grid with 2px padding
- [ ] Consistent stroke width (1.5px at 24px)
- [ ] Consistent corner radius (2px) and end caps (round)
- [ ] Optical centering applied where needed
- [ ] All paths optimized (no editor bloat)
- [ ] `currentColor` for fill/stroke, never hardcoded colors
- [ ] Icon-only elements have `aria-label`
- [ ] Icon + text pairs have consistent gap (8px)
- [ ] All sizes defined in design tokens
- [ ] Hover/active/disabled states defined for icon buttons

## Anti-patterns I avoid

- Mixing icon styles (outlined + filled) in the same context
- Using icons without text labels for critical actions
- Icons larger than 24px in inline text context
- Hardcoded colors in SVG — always use `currentColor`
- Using PNG/IMG icons instead of SVG — not scalable
- Icons without accessible labels — screen readers see nothing
- Overly detailed icons at 16px — simplify for small sizes
- 3+ icon colors in one view — use 1 accent color + 1 neutral