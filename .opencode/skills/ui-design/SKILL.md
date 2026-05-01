---
name: ui-design
description: Design polished, Stitch-quality interfaces from wireframe to implementation with layout, spacing, color, and typography mastery
---

## What I do

I design and implement polished, production-quality UIs — the end-to-end workflow from wireframe to finished page. I'm the orchestrator that calls focused sub-skills for each design dimension:

- Translate requirements and wireframes into beautiful, functional interfaces
- Apply professional layout, spacing, typography, and color decisions
- Ensure visual consistency, proper alignment, and breathability
- Deliver pixel-precise implementations that match the design intent
- Coordinate the design skill chain: foundations → typography → color → proportions → composition → QA

## When to use me

Use this skill when:
- Building a new page, screen, or feature from scratch (the full end-to-end workflow)
- Converting a rough wireframe or description into a polished UI
- Redesigning an existing interface that looks unpolished or inconsistent
- You need one skill that covers the whole design process (not just one dimension)

For specific design dimensions, use the focused skills instead:
- **design-foundations** — Purpose, audience, aesthetic direction (the layer most people skip)
- **typography** — Font selection, pairing, type scales, vertical rhythm
- **color-palette** — Building palettes from color science, mood-to-hue-to-scheme
- **proportions** — Mathematical ratios, type/space scale alignment, container design
- **composition** — Dominance, similarity, rhythm, texture, direction, contrast
- **strip-ai-tells** — Detecting and replacing AI design defaults
- **design-audit** — 10-dimension systematic review before shipping
- **visual-qa** — Pixel-level spacing, alignment, and rendering review

## How I work

The page design workflow is a gated sequence. Each phase produces decisions the next phase depends on:

### Phase 1: Foundation (design-foundations)

Before choosing a single visual element:

1. **Define purpose** — One sentence: what this page must accomplish
2. **Define audience** — Who will see this, what they expect, what they need
3. **Choose aesthetic direction** — Not "clean and modern." Pick a real direction (editorial, Swiss, warm craft, brutalist, etc.)
4. **Document anti-directions** — What you are NOT doing, and why

**Gate:** If you can't state the purpose in one sentence, stop. Don't choose fonts or colors without a foundation.

### Phase 2: Type System (typography)

With the aesthetic direction as your guide:

1. **Select body font** — Screen-appropriate, readable at 16px, fits the direction
2. **Pair with heading font** — Contrast in structure, not just weight. Max 2 families + mono.
3. **Choose type scale ratio** — 1.25 (modern), 1.333 (editorial), 1.5 (dramatic)
4. **Set line heights** — Decrease as size increases
5. **Implement as tokens** — All sizes, weights, and leading as CSS custom properties

**Gate:** Blurring your eyes at the page should reveal a clear hierarchy. If it doesn't, the type system isn't working.

### Phase 3: Color System (color-palette)

With the mood from the aesthetic direction:

1. **Choose base hue from mood** — Trust=blue, energy=orange, growth=green, creative=purple
2. **Derive scheme** — Analogous (harmonious), complementary (vibrant), or split-complementary (balanced)
3. **Build neutral scale** — Warm or cool temperature; 10 steps from white to near-black
4. **Map semantic tokens** — Primary, secondary, borders, backgrounds, success/warning/error
5. **Verify contrast** — All text/background pairs must meet WCAG AA (4.5:1 for text, 3:1 for UI)

**Gate:** Cover the content. Does the palette alone communicate the mood? Does white text on primary pass contrast?

### Phase 4: Proportions (proportions)

With type scale set, derive the spatial system:

1. **Choose space scale ratio** — Same as type scale (for harmony) or 4px step grid (for simplicity)
2. **Align type-space reference points** — Body line-height should be a clean space scale value
3. **Set container proportions** — Content width with intentional ratio to viewport
4. **Derive margins** — From content area or viewport, not arbitrary
5. **Implement as tokens** — All spacing values as CSS custom properties

**Gate:** Every spacing value should map to a token. No magic numbers.

### Phase 5: Composition (composition)

With the content, type, color, and proportions ready:

1. **Choose the dominant element** — What must be seen first? Give it the most size, weight, space.
2. **Arrange supporting elements** — They reinforce the dominant, never compete.
3. **Vary density** — Dense areas for focus, sparse areas for breathing.
4. **Set direction** — The eye follows: dominant → secondary → tertiary.
5. **Add contrast** — White space > weight > size > color. Use color last.

**Gate:** The 3-second test — if the visual priority is obvious when you squint, the composition works.

### Phase 6: Build (implementation)

With all design decisions documented:

1. **Semantic HTML** — Landmarks, heading hierarchy, accessible names
2. **CSS tokens** — All design tokens as custom properties
3. **Responsive layout** — Fluid type, 2-3 breakpoints, mobile-first
4. **Interaction states** — Hover, focus, active, disabled (see: interaction-design)
5. **Motion** — Purposeful animation, not decorative (see: motion-system)

### Phase 7: Ship (design-audit + visual-qa)

Before shipping, run the gates:

1. **Design audit** — 10-dimension review (purpose → identity)
2. **Visual QA** — Pixel-level spacing, alignment, contrast, cross-browser
3. **AI tell check** — Zero AI tells remaining (see: strip-ai-tells)

## Quick design decisions

When you need fast answers without loading the full sub-skills:

### Type choices
- **2 weights maximum:** regular (400) + semibold/bold (600/700)
- **Type scale ratio:** 1.25 (apps) or 1.333 (editorial)
- **Line height:** 1.1 headings, 1.5 body, 1.6 long-form
- **Max 2 font families + mono**

### Color choices
- **1 primary hue** + neutrals + 1-2 semantic colors
- **90% neutral, 10% color**
- **Warm or cool neutrals** — pick one temperature
- **Contrast:** 4.5:1 minimum for text

### Layout choices
- **8px grid** — all spacing in multiples of 4
- **Left-align body text** — center only for short hero statements
- **Content width:** 38-42em for reading, 56-72em for layouts
- **Not everything needs a card** — use whitespace for grouping first

### Interaction choices
- **44×44px minimum** touch targets
- **Focus-visible** on all interactive elements
- **300ms standard** transitions, 100ms micro, 500ms complex
- **prefers-reduced-motion** respected

## Quality checklist

- [ ] Purpose defined in one sentence before any visual decisions
- [ ] Aesthetic direction is specific (not "clean and modern")
- [ ] Maximum 2 font families loaded
- [ ] Type scale uses a named ratio
- [ ] Color palette built from mood→hue→scheme
- [ ] Contrast ratios meet WCAG AA
- [ ] All spacing maps to design tokens
- [ ] Type scale and space scale are aligned
- [ ] One dominant element per view
- [ ] Hover, focus, active states on all interactive elements
- [ ] Responsive at 375px, 768px, 1024px+
- [ ] Zero AI tells remaining
- [ ] Semantic HTML with proper heading hierarchy
- [ ] `prefers-reduced-motion` respected

## Anti-patterns I avoid

- Starting with fonts and colors before defining purpose — every visual choice must serve the foundation
- Using "clean and modern" as an aesthetic direction — it's the absence of a direction
- Skipping the type scale and using arbitrary font sizes — inconsistency compounds
- Equal spacing everywhere — hierarchy requires proportional spacing
- Making everything accent-colored — when everything stands out, nothing does
- Centering all body text — left-align for readability
- Putting every content type in a card — whitespace and similarity group better
- Shipping without running the audit — the last gate exists because earlier gates get skipped