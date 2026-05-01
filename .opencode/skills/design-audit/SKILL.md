---
name: design-audit
description: Theory-backed design diagnosis across 10 dimensions — finds what's wrong and explains why it violates design principles
---

## What I do

I perform systematic design audits that diagnose problems at the principle level, not just the pixel level — the difference between "something looks off" and "this violates the principle of dominance because no single element commands attention":

- **10-dimension review** — Purpose through design identity, each checked against design theory
- **Principle-based diagnosis** — Not "this looks bad" but "this violates [specific principle] because [reason]"
- **Severity classification** — Critical (accessibility/broken), major (principle violation), minor (refinement)
- **Prioritized fix order** — What to fix first for the highest impact

## When to use me

Use this skill when:
- A design feels "off" but it's hard to articulate why
- Reviewing a design before shipping (the last gate)
- Assessing whether AI-generated design output passes quality standards
- Comparing a design against the principles it claims to follow
- Getting a structured list of issues rather than vague feedback

## How I work

### Checker mode (full 10-dimension audit)

Run every dimension. Rate each Pass / Pass with issues / Fail. Document every violation with the principle it breaks.

### Applier mode (building with audit awareness)

Use the 10 dimensions as a build checklist — design each dimension correctly from the start rather than fixing later. After building, run the Checker audit as the final gate.

## The 10 audit dimensions

### 1. Purpose and foundation

**Principle:** Design without stated purpose is decoration.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Purpose clarity | Can state purpose in one sentence | "Clean and modern" or "looks good" |
| Audience definition | Primary audience named with needs | "For everyone" |
| Aesthetic direction | Specific direction (editorial, Swiss, etc.) | "Minimal" or "professional" |
| Foundation-to-design mapping | Every visual choice traces to a decision | Typography and color chosen independently |

**Why this matters:** Without foundation, every later choice is arbitrary. Arbitrary choices converge to defaults. Defaults converge to AI output.

### 2. Typography

**Principle:** Medium matters. Type must serve its medium and audience.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Font choice | Deliberate, appropriate for screen/print | Inter, Roboto, or Open Sans without justification |
| Font count | Maximum 2 families + mono | 3+ families, or 2 families that don't pair |
| Type scale | Named ratio (1.25, 1.333, 1.5) | Arbitrary sizes that don't relate |
| Line height | Decreases as size increases | Same line-height on headings and body |
| Line length | 50-75 characters per line | Over 80 characters or under 40 |
| Pairing | Contrast in structure (geometric + humanist, serif + sans) | Two similar sans-serifs that compete |

**Why this matters:** Type carries 95% of web content. Bad typography isn't ugly — it's inhospitable. The squint test: blur your eyes. If you can still see the hierarchy, it works.

### 3. Proportions and layout

**Principle:** Dimensions should relate through intentional ratios, not arbitrary pixel values.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Spacing scale | Follows a named ratio or consistent system | Random pixel values (13px, 17px, 19px) |
| Type-space harmony | Space scale uses the same ratio as type scale (or a stated alternative) | Type scale is 1.333 but space scale follows no system |
| Container proportion | Intentional ratio (2:3, golden, etc.) | Default container width with no reasoning |
| Margin derivation | Margins relate to content area (Tschichold method) or to type scale | Margins are whatever looks okay |
| Grid alignment | Elements align to a consistent grid | Columns drift 1-2px from the grid |

**Why this matters:** Intentional proportions create harmony the eye registers even when the mind can't name it. Arbitrary proportions create subtle wrongness.

### 4. Composition

**Principle:** One element commands attention. The eye is guided, not scattered.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Dominance | One element is clearly the visual anchor | Multiple elements competing equally |
| Hierarchy flow | Eye moves naturally from dominant → secondary → tertiary | Eye jumps randomly |
| Asymmetry | Asymmetric layout with intentional tension | Everything centered or evenly distributed |
| Grouping | Related items are near, unrelated items are far | Uniform spacing everywhere |
| Visual variety | Different sized elements, different densities | All sections same height, same layout |

**Why this matters:** "Not everything gets a card." Composition is how you direct attention. Without it, everything screams equally and nothing is heard.

### 5. Visual hierarchy

**Principle:** White space is the most powerful tool. Then weight, then size, then color.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Spacing hierarchy | Tight within groups, generous between | Equal spacing everywhere |
| Section differentiation | Not all sections look the same | All h2s same size, same spacing above |
| Emphasis tools | Uses spacing + weight first, color sparingly | Color as primary differentiator |
| Primary content | One clear focus per view | Everything at the same visual weight |
| Breathing room | Generous whitespace around key content | Content crammed edge-to-edge |

**Why this matters:** Tufte's 1+1=3 — two elements plus the space between them creates a third informational entity. Proportional spacing communicates grouping.

### 6. Color

**Principle:** Mood drives hue. Hue drives scheme. Accessibility is non-negotiable.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Palette derivation | Built from color wheel relationships | Random hex values with no stated relationship |
| Hue count | 1 primary + 1 accent is enough | 4+ hues with no clear system |
| Mood-hue mapping | Color mood matches purpose | Cyan-on-dark because "it looks techy" |
| Contrast ratios | WCAG AA minimum (4.5:1 text, 3:1 large text) | Text-light on background fails contrast |
| Colorblind safety | Redundant cues (not color alone) | Red/green status indicators with no icon |
| Temperature | Consistent warm or cool neutrals | Mixed warm and cool grays |

**Why this matters:** Color is the least reliable tool for hierarchy (10% of men are colorblind) but the most emotional. Use it for mood, not structure.

### 7. Semantic HTML and SEO

**Principle:** Structure serves machines as well as humans.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Landmarks | `<main>`, `<nav>`, `<header>`, `<footer>` present | All content in `<div>` |
| Heading hierarchy | Single `<h1>`, logical h2→h3 descent | Multiple h1s, skipped levels |
| Meta tags | Description, OG tags, viewport present | Missing meta description or OG |
| Accessible name | Interactive elements have labels | Icon buttons with no aria-label |
| Skip link | "Skip to content" link exists | No keyboard skip path |

**Why this matters:** The invisible architecture. A design that looks right but has no semantic structure is a veneer — drawing ponies, as Kadavy says.

### 8. Motion and interaction

**Principle:** Remove an animation. Did you lose information? If not, it was decorative. Eight states every interactive element needs.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Animation purpose | Every animation communicates information | Everything fades in from below |
| Duration | 100ms micro, 300ms standard, 500ms complex | 1-second animation on simple state change |
| Easing | Named easing curves per purpose | `linear` on UI motion, or all same `ease` |
| Interaction states | 8 states defined: default, hover, focus, active, disabled, loading, error, success | Only default state exists |
| Focus-visible | Keyboard focus shows ring | No focus indicator, or :focus instead of :focus-visible |
| Reduced motion | `prefers-reduced-motion` respected | Animations run regardless of user preference |
| Touch targets | Minimum 44×44px | Small text links with 20px hit area |

**Why this matters:** Motion is the most common AI tell ("everything fades in from below with the same timing"). Interaction completeness separates professional from amateur.

### 9. Responsive design

**Principle:** Content must be usable at every viewport.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| Fluid type | `clamp()` on base and heading sizes | Fixed px sizes that don't scale |
| Breakpoints | 2-3 meaningful breakpoints | Single breakpoint or no breakpoints |
| Touch targets | 44px minimum at mobile widths | Same small targets on mobile as desktop |
| Content reflow | Grids collapse, sidebars stack | Horizontal scroll, or content hidden |
| Testing | Verified at 375px, 768px, 1024px+ | Only tested at desktop width |

**Why this matters:** Responsive isn't a feature — it's the baseline expectation. Failing mobile = failing most users.

### 10. Design identity

**Principle:** If your design could be mistaken for AI output, it has no identity.

| Check | Pass criteria | Fail signal |
|-------|---------------|-------------|
| AI tell check | None of the common AI tells present | 3+ AI tells (see: strip-ai-tells skill) |
| Distinctive choices | At least 3 design choices that AI wouldn't make by default | Every choice is a common default |
| Defensibility | Every major choice can be defended with reasoning | "It looked good" is the only justification |
| Character | The design has recognizable personality | Feels like it could be any competitor's site |

**Why this matters:** 75% of website credibility judgments are design-based (Fogg, Stanford). Generic = not credible. Identity = credible.

## Audit report template

```
DESIGN AUDIT REPORT

Date: [date]
Design: [what was audited]
Auditor: [who]

DIMENSION RESULTS:

1. Purpose & Foundation: [Pass / Pass- / Fail]
   - [issue if any]: [principle violated] — [why it matters]

2. Typography: [Pass / Pass- / Fail]
   - [issue]: [principle violated] — [why it matters]

[... continue for all 10 ...]

SUMMARY:
  Critical issues: [count] — [list]
  Major issues: [count] — [list]
  Minor issues: [count] — [list]

FIX PRIORITY:
  1. [most impactful fix]
  2. [next most impactful]
  3. [etc.]

AI TELLS DETECTED:
  - [tell]: [where] — [replacement suggestion]
```

## Quality checklist

- [ ] All 10 dimensions reviewed
- [ ] Every issue includes the principle it violates
- [ ] Every issue explains why the violation matters
- [ ] Critical issues (accessibility) flagged first
- [ ] Fixes are prioritized by impact, not by dimension order
- [ ] AI tells are specifically called out (cross-reference strip-ai-tells)
- [ ] The report could be understood by someone who didn't run the audit

## Anti-patterns I avoid

- Vague feedback ("this doesn't feel right") — always name the violated principle
- Pixel-level critique without principle-level reasoning — "move this 4px left" without "because it breaks alignment to the grid"
- Auditing only the visual layer — semantic HTML, interaction states, and accessibility are design
- Skipping the foundation check — if purpose is unclear, nothing else can be properly evaluated
- Treating all issues as equal — a contrast failure is more important than a spacing inconsistency
- Giving feedback that amounts to "make it look better" — actionable, specific, principle-based fixes only
- Auditing with personal taste instead of design principles — personal preference ≠ design violation