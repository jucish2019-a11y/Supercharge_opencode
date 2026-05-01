---
name: ui-design
description: Design polished, Stitch-quality interfaces from wireframe to implementation — the comprehensive orchestrator that produces breathtaking UI through a gated 9-phase workflow
---

## What I do

I design and implement polished, production-quality UIs at the level of Stripe, Linear, and Vercel. I am the orchestrator that coordinates every design dimension through a gated workflow, calling specialized sub-skills at each phase.

The difference between a generic interface and a breathtaking one is not any single visual choice. It is the cumulative effect of: purpose defined before visuals, content structured as narrative, type scales with dramatic ratios, color systems with tonal depth, spatial grids with mathematical precision, compositions with clear dominance, state machines with 12+ states, and spring physics that feel alive.

I coordinate all of this in sequence, with gates that prevent moving forward until the current phase is solid.

- Translate requirements into purpose-driven, narrative-structured interfaces
- Apply professional layout, spacing, typography, color, and composition decisions
- Define complete interaction state machines (not just :hover and :focus)
- Specify physics-based motion systems (not just CSS transitions)
- Deliver implementations that feel inevitable, as if no other design could be right
- Coordinate the design skill chain through 9 gated phases

## When to use me

Use this skill when:
- Building a new page, screen, or feature from scratch (the full end-to-end workflow)
- Converting a rough wireframe or description into a polished, breathtaking UI
- Redesigning an existing interface that looks generic or feels mechanical
- You need one skill that covers the whole design process at Stitch quality
- You want every dimension (type, color, space, motion, states, content) coordinated

For specific design dimensions, use the focused skills instead:
- **design-foundations** — Purpose, audience, aesthetic direction (the layer most people skip)
- **editorial-design** — Content architecture, narrative arc, progressive disclosure, data display grammar
- **typography** — Font selection, pairing, type scales, vertical rhythm
- **color-palette** — Building palettes from color science, mood-to-hue-to-scheme, tonal systems
- **proportions** — Mathematical ratios, type/space scale alignment, container design
- **composition** — Dominance, similarity, rhythm, texture, direction, contrast
- **interaction-states** — 12-state taxonomy, frequency-novelty matrix, transition timing
- **premium-motion** — Spring physics, stagger cascades, container transforms, scroll-linked animation
- **strip-ai-tells** — Detecting and replacing AI design defaults
- **design-audit** — 10-dimension systematic review before shipping
- **visual-qa** — Pixel-level spacing, alignment, and rendering review

## How I work

The page design workflow is a gated sequence of 9 phases. Each phase produces decisions the next phase depends on. You cannot skip a phase.

### Phase 1: Foundation (design-foundations)

Before choosing a single visual element:

1. **Define purpose** — One sentence: what this page must accomplish
2. **Define audience** — Who will see this, what they expect, what they need
3. **Choose aesthetic direction** — Not "clean and modern." Pick a real direction (editorial, Swiss, warm craft, brutalist, etc.)
4. **Document anti-directions** — What you are NOT doing, and why

**Gate**: If you cannot state the purpose in one sentence, stop. Do not choose fonts or colors without a foundation.

### Phase 2: Content Architecture (editorial-design)

With the foundation set, define what content goes where and why:

1. **Choose narrative arc** — Resolution, Revelation, Authority, Education, or Immersion
2. **Map information hierarchy** — L0 (immediate) through L4 (post-decision)
3. **Assign disclosure levels** — What is visible at first glance, what requires scroll, what requires engagement
4. **Set section-to-section tonal shifts** — Each section should have a different emotional register: bold → quiet → precise → warm → urgent
5. **Choose data display grammar** — Numbers for authority, logos for proof, diagrams for explanation, mockups for demonstration
6. **Set density mode per section** — Compact for data, standard for tasks, comfortable for reading

**Gate**: Read the section headers in sequence. Do they tell a story with rising action? If it reads like a flat list, the content architecture is not working.

### Phase 3: Type System (typography)

With content structured, define the typographic hierarchy:

1. **Select body font** — Screen-appropriate, readable at 16px, fits the aesthetic direction
2. **Pair with heading font** — Contrast in structure, not just weight. Maximum 2 families + 1 mono.
3. **Choose type scale ratio** — 1.25 (modern), 1.333 (editorial), 1.5 (dramatic)
4. **Define role-based sizes** — Display (96px+), Title 1-3 (64-36px), Headline (28px), Body Large (20px), Body (16px), Caption (14px), Overline (12px)
5. **Set line heights** — Display: 1.05x, Titles: 1.15x, Body: 1.5x, Caption: 1.4x
6. **Set letter-spacing** — Tight (-0.02em) for display/title, Normal (0) for body, Wide (0.02em) for caption, Wider (0.08em) for overline
7. **Implement as tokens** — All sizes, weights, and leading as CSS custom properties

**Gate**: Blur your eyes at the page. If the typographic hierarchy is not immediately obvious, the type system needs work. The ratio between display and body should be at least 3:1 for dramatic hierarchy, 2:1 minimum for any hierarchy.

### Phase 4: Color System (color-palette)

With the mood from the aesthetic direction and the typography set:

1. **Choose base hue from mood** — Trust=blue, energy=orange, growth=green, creative=purple, warmth=amber, authority=indigo
2. **Derive scheme** — Analogous (harmonious), complementary (vibrant), or split-complementary (balanced)
3. **Generate 10-step tonal palette** — From lightest (tone 5) to darkest (tone 95) for the base hue
4. **Build neutral scale** — Warm or cool temperature; 10 steps from white to near-black
5. **Map role-based semantic tokens** — primary, secondary, surface, surface-container (low through high), outline, error, success
6. **Define elevation-through-tone** — Higher elevation = lighter tonal step on surface (not drop shadows)
7. **Build dark mode palette** — Recalculate tones for dark backgrounds, do not invert
8. **Verify contrast** — All text/background pairs meet WCAG AA (4.5:1 text, 3:1 UI)

**Gate**: Cover the content. Does the palette alone communicate the mood? Does white text on primary pass contrast? Does the elevation system work without shadows?

### Phase 5: Spatial System (proportions + spatial-design)

With type and color decisions made, derive the spatial grid:

1. **Choose base unit** — 4px or 8px grid (all spacing is a multiple)
2. **Build space scale** — xs(4), sm(8), md(16), lg(24), xl(32), 2xl(48), 3xl(64), 4xl(96), 5xl(128)
3. **Align type-space reference** — Body line-height should be a clean space scale value
4. **Set container proportions** — Content width with intentional ratio (38-42em for reading, 56-72em for layouts)
5. **Define density modes** — Compact (32px rows), Standard (40px), Comfortable (48px)
6. **Set section separator scale** — 64px internal, 96px standard, 128px major sections
7. **Implement as tokens** — All spacing values as CSS custom properties

**Gate**: Every spacing value maps to a token. No magic numbers. Type scale and space scale are aligned (body line-height is a space value). Section separators create proportional breathing room.

### Phase 6: Composition (composition)

With content, type, color, and space ready:

1. **Choose the dominant element** — What must be seen first? Give it the most size, weight, and space.
2. **Arrange supporting elements** — They reinforce the dominant, never compete with it.
3. **Vary density** — Dense areas for focus, sparse areas for breathing. No uniform density.
4. **Set direction** — The eye follows: dominant → secondary → tertiary.
5. **Apply contrast hierarchy** — White space > weight > size > color. Use color last.
6. **Use grouping principles** — Proximity (closer = related), similarity (same style = same type), enclosure (borders for strong grouping), continuity (alignment for weak grouping).

**Gate**: The 3-second test. Squint at the page. If the visual priority is obvious, the composition works. If everything competes for attention, it does not.

### Phase 7: Interaction States (interaction-states)

With the visual foundation set, define how every element responds to input:

1. **Identify interactive elements** — List every button, card, input, toggle, link, and navigation item
2. **Assign state machine type** — Button (core+extended), Card (core+advanced), Input (core+extended), etc.
3. **Define visual expressions per state** — Color, opacity, scale, shadow, border for default, hover, active, focused, selected, disabled, loading, error
4. **Set transition timing per state change** — Entry duration/curve, exit duration/curve (often asymmetric)
5. **Apply frequency-novelty matrix** — High-frequency/low-novelty = instant; Low-frequency/high-novelty = invested
6. **Set hover intent** — 50ms delay on enter, 0ms on exit
7. **Brand selection highlighting** — Primary color at 20% opacity for text selection

**Gate**: Every interactive element has at minimum 6 core states defined. Hover and focus are never the same visual. High-frequency interactions have fast (60-120ms) transitions.

### Phase 8: Motion System (premium-motion + motion-system)

With states defined, specify how transitions feel:

1. **Choose motion personality** — Gentle (floating), Snappy (precise), Bouncy (playful), or Stiff (immediate)
2. **Define spring configurations** — Standard, Entrance, and Exit springs with mass/tension/friction values
3. **Set duration scale** — Instant (0ms), Quick (80-120ms), Standard (200-300ms), Emphasized (400-500ms), Scenic (600ms+)
4. **Define stagger patterns** — List (30ms), Grid (50ms), Menu (20ms), Dashboard (60ms)
5. **Choose container transition patterns** — Card→Detail (hero expand), List→Detail (slide reveal), Tab switch (shared axis), Unrelated (fade through)
6. **Determine scroll behavior** — Triggered (entrances) vs. linked (parallax, progress)
7. **Set reduced-motion fallbacks** — Every animation has an instant alternative

**Gate**: No animation uses generic `ease-in-out` for prominent interactions. Spring physics are specified for all entrance/exit transitions. Stagger patterns are defined for grouped elements. Reduced-motion alternatives exist for every animation.

### Phase 9: Polish and Ship (design-audit + visual-qa + strip-ai-tells)

Before shipping, run the final checks:

1. **Design audit** (design-audit) — 10-dimension review: purpose, typography, proportions, composition, visual hierarchy, color, semantic HTML, motion, responsive, design identity
2. **Visual QA** (visual-qa) — Pixel-level spacing check, alignment verification, contrast testing, cross-browser review
3. **AI tell check** (strip-ai-tells) — Zero AI tells remaining (default sans-serif, cyan-on-dark, identical cards, everything centered, equal spacing, glassmorphism/gradient text, neon accents, uniform fade-in animation)
4. **Interaction review** — Every state machine works as specified. No state is visually identical to another.
5. **Motion review** — Animations feel intentional and alive. High-frequency interactions are fast. Low-frequency are expressive. Springs feel physical.
6. **Density review** — Page has variation between compact, standard, and comfortable sections. No uniform density throughout.
7. **Narrative review** — Sections tell a story in order. No two adjacent sections have the same tonal register or layout pattern.

**Gate**: Zero AI tells. All contrast passes WCAG AA. Every interactive element has distinct states. No two adjacent sections feel the same. The page feels inevitable, as if no other design could be right.

## Quick reference: Breathtaking UI cheat sheet

### Type choices
- **2 weights maximum**: regular (400) + semibold/bold (600/700)
- **Type scale ratio**: 1.25 (apps) or 1.333 (editorial) or 1.5 (dramatic)
- **Line height**: 1.05 display, 1.15 titles, 1.5 body, 1.4 caption
- **Letter-spacing**: tight display, normal body, wide caption, wider overline
- **Maximum 2 font families + 1 mono**
- **Display-to-body ratio**: minimum 3:1 for dramatic, 2:1 minimum

### Color choices
- **1 primary hue** with 10-step tonal palette
- **90% neutral, 10% color** in most sections
- **Warm or cool neutrals** — pick one temperature
- **Elevation through tonal shifts**, not shadows
- **Contrast**: 4.5:1 minimum for text, 3:1 for UI
- **Semantic color as navigation** — products, sections, or categories get distinct hues

### Layout choices
- **8px grid** — all spacing in multiples of 4
- **Left-align body text** — center only for short hero statements
- **Content width**: 38-42em reading, 56-72em layouts
- **Not everything in a card** — whitespace groups better than borders
- **Section separators**: 64px internal, 96px standard, 128px major
- **Density varies**: compact for data, standard for tasks, comfortable for reading

### Interaction choices
- **12 states minimum** for full interaction model (6 core + 4 extended + 2 advanced as needed)
- **Asymmetric transitions**: entries often slower than exits
- **Hover intent**: 50ms delay on enter, 0ms on exit
- **44x44px minimum** touch targets
- **Focus-visible** on all interactive elements
- **High-frequency = instant**: context menus, command palettes, button presses = 0-60ms
- **Low-frequency = invested**: page transitions, onboarding, first-time reveals = 400-800ms

### Motion choices
- **Springs, not easing curves** for element transitions
- **Stagger cascades**, never simultaneous group animations
- **Scroll-linked** for parallax and progress; **scroll-triggered** for entrances
- **Container transforms** for page navigation; **fade-through** for unrelated content
- **Interruptible transitions** — every animation can be reversed mid-flight
- **prefers-reduced-motion**: all animations → instant state changes
- **Brand the selection color**: primary at 20% opacity for `::selection`

### Content choices
- **Narrative arc** — every page tells a story with rising action, not a flat list
- **Progressive disclosure** — L0 (hero) through L4 (footer), each level earns its scroll
- **Specific data** — "$2,847" not "$3000", "ENG-2703" not "#1234", "99.999%" not "100%"
- **Tonal shifts** between sections — no two adjacent sections feel the same
- **Data display grammar** — numbers for authority, logos for proof, diagrams for explanation, mockups for demonstration

## Quality checklist

- [ ] Purpose defined in one sentence before any visual decisions
- [ ] Aesthetic direction is specific (not "clean and modern")
- [ ] Sections follow a narrative arc with tonal shifts
- [ ] Maximum 2 font families + 1 mono loaded
- [ ] Type scale uses a named ratio with display-to-body minimum 2:1
- [ ] Color palette has 10-step tonal palette and role-based semantic tokens
- [ ] Contrast ratios meet WCAG AA
- [ ] Elevation through tonal shifts, not shadows
- [ ] All spacing maps to design tokens on an 8px grid
- [ ] Type scale and space scale are aligned
- [ ] One dominant element per view
- [ ] No two adjacent sections have same tonal register or layout pattern
- [ ] Every interactive element has 6+ states defined with distinct visuals
- [ ] Transition timing is asymmetric where appropriate (faster exits)
- [ ] Spring physics specified for element transitions
- [ ] Stagger patterns defined for grouped elements
- [ ] `prefers-reduced-motion` respected with instant fallbacks
- [ ] Responsive at 375px, 768px, 1024px+
- [ ] Zero AI tells remaining
- [ ] All placeholder content replaced with specific, credible data
- [ ] Semantic HTML with proper heading hierarchy

## Anti-patterns I avoid

- Starting with fonts and colors before defining purpose — every visual choice must serve the foundation
- Using "clean and modern" as an aesthetic direction — it is the absence of a direction
- Generic section order (hero → features → testimonials → CTA) — content must follow a narrative arc
- Skipping the type scale and using arbitrary font sizes — inconsistency compounds
- Equal spacing everywhere — hierarchy requires proportional spacing
- Uniform density across all sections — compact/standard/comfortable create rhythm
- Making everything accent-colored — when everything stands out, nothing does
- Centering all body text — left-align for readability
- Putting every content type in a card — whitespace and similarity group better
- Using `ease-in-out` for all transitions — springs feel alive, generic curves feel mechanical
- Animating all elements simultaneously — stagger cascades create intentionality
- Animating high-frequency interactions heavily — context menus and command palettes should appear instantly
- Defining only :hover and :focus — premium interfaces need 6+ states per element
- Same duration for enter and exit — exits should often be faster
- Ignoring elevation-through-tone — premium sites use tonal shifts, not drop shadows
- Circular numbers and generic data — "$2,847" is more credible than "$3000"
- Two adjacent sections with the same layout pattern — vary bento grids, full-bleed sections, contained cards, data tables
- Shipping without running the full audit — the 9th phase exists because phases 1-8 are never perfect on the first pass