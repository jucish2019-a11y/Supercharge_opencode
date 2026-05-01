---
name: strip-ai-tells
description: Detect convergent AI design defaults and replace them with choices that have character — stop shipping designs that scream "AI made this"
---

## What I do

I identify and replace the visual patterns that make AI-generated designs instantly recognizable as AI-generated — the convergence signals that turn "clean and modern" into a red flag:

- **AI tell detection** — Systematically identify the 8 common AI design defaults
- **Tell replacement** — Substitute each default with a defensible, character-full alternative
- **Convergence audit** — Check whether a design could be mistaken for any other AI-generated page
- **Character injection** — Add deliberate choices that give design identity and intentionality

## When to use me

Use this skill when:
- A design looks "too clean" or "too generic" and you suspect AI tells
- Reviewing AI-generated UI output before shipping
- You want a design to feel intentionally crafted, not procedurally generated
- A client says "it looks nice but something's off"
- You're starting a new design and want to avoid convergence from the beginning

## How I work

### Checker mode (detect AI tells)

1. **Scan for each tell** — Run through the 8 common AI tells one by one.
2. **Rate tell severity** — How prominently does each tell appear? (Subtle / Moderate / Obvious)
3. **Count total tells** — 0-1: likely human or very good AI. 2-3: AI-influenced. 4+: clearly AI-generated.
4. **Generate replacement options** — For each tell, suggest 2-3 defensible alternatives.

### Applier mode (build without tells)

1. **Start with design foundations** — Purpose, audience, aesthetic direction (see: design-foundations skill).
2. **Choose anti-defaults** — For each decision point, pick the opposite of what AI would default to.
3. **Add character through constraints** — Deliberately limit options to force creative solutions.
4. **Verify with the tell checklist** — Before shipping, confirm zero unaddressed tells.

## The 8 common AI tells

### Tell 1: Default sans-serif fonts

**What AI does:** Picks Inter, Roboto, Open Sans, or system-ui as the body typeface.

**Why it's a tell:** These fonts are designed to be invisible — maximum legibility, minimum personality. Using them says "I chose the default" louder than any other decision.

**Replacements:**
- Choose a typeface with a point of view: a serif for headings (Source Serif, Lora, Freight), a humanist sans (Spectral, Avenir, Gill Sans), or a geometric with character (Space Grotesk, Cabinet Grotesk)
- If you must use a neutral sans for body text, pair it with a distinctive heading face
- Medium matters: Garamond is timeless on paper, blurry on screen. Choose for the medium

**Test:** Squint at the page. Can you identify the typeface family? If not, it's a default.

### Tell 2: Cyan-on-dark or purple-to-blue gradients

**What AI does:** Selects a palette from the "tech startup" generator: dark backgrounds with cyan accents, or purple→blue gradients, or both.

**Why it's a tell:** These palettes have become synonymous with AI output. They signal "I used a color scheme that reads as technological" without being tied to any actual mood or purpose.

**Replacements:**
- Start from mood, not from "what looks techy"
- Earthy palettes: warm neutrals (#faf7f2) with terracotta accents (#c45d3e)
- Editorial palettes: near-black text on cream, one saturated accent
- Restrained palettes: 90% neutral, one hue used at one saturation
- Build from color wheel relationships (see: color-palette skill), not from "vibes"

**Test:** Cover the content. Does the palette alone communicate a specific mood? If it just says "modern tech," it's a default.

### Tell 3: Identical card grids

**What AI does:** Arranges content in uniform cards — icon + heading + description, repeated in a 3-column grid — with equal spacing, equal sizing, equal weight.

**Why it's a tell:** This is the path of least resistance. Cards are easy to generate and hard to make distinctive. When every item gets equal visual weight, nothing has weight.

**Replacements:**
- Give one card (the most important) more space, larger type, or a different layout
- Break the grid: use a featured item that spans 2 columns, or a different visual treatment
- Replace some cards with simple list items, emphasized text, or full-bleed sections
- Not everything gets a card. Some things are better as paragraphs, some as data tables, some as nothing at all

**Test:** Remove the card borders and backgrounds. Does the content still make sense? If not, the cards are doing layout work that spacing could do better.

### Tell 4: Everything centered

**What AI does:** Centers headings, body text, CTAs, and images. Aligns nothing to a consistent left edge.

**Why it's a tell:** Center alignment is the default because it "looks balanced." But balanced ≠ better. Left-alignment creates a consistent axis that the eye can follow. Center creates a ragged left edge that forces the eye to find the starting point of every line.

**Replacements:**
- Left-align body text. Always. Line lengths over 40 characters should be left-aligned.
- Left-align headings when they're over one line
- Center only: short hero statements (1-2 lines), single-word labels, CTA buttons
- Use asymmetric layouts: one wide column, one narrow. Grid offset. Deliberate imbalance.

**Test:** Draw a vertical line at the most common left edge. How many elements ignore it? If most do, you have a centering problem.

### Tell 5: Equal spacing everywhere

**What AI does:** Applies the same margin/padding between all sections and elements. No rhythm, no grouping, no variation.

**Why it's a tell:** Equal spacing says "I couldn't decide what's important." Spacing is the most powerful hierarchy tool. Tight within groups, generous between groups. Equal spacing everywhere = no grouping.

**Replacements:**
- Use the proportional spacing rule: tight within groups (4-8px), medium between related groups (16-24px), generous between sections (32-96px)
- The ratio between spacing levels should be at least 2:1 (not 16px/20px/24px — that's 1.25:1, which reads as "same")
- Vary section padding. Hero can be 120px, content sections 80px. Not everything gets the same padding.
- Use borders and rules to separate where spacing alone isn't enough

**Test:** Measure the gap between any two adjacent sections. If they're the same, ask: should they be?

### Tell 6: Glassmorphism and gradient text

**What AI does:** Applies backdrop-blur glass cards, frosted overlays, and gradient text (usually blue→purple or blue→green) as decoration.

**Why it's a tell:** These are decorative effects that communicate "I wanted visual interest" without serving the content. They were novel in 2020. Now they're the fastest AI tell.

**Replacements:**
- Use flat color backgrounds with intentional contrast instead of glass
- If you want depth, use elevation through shadow (subtle, not dramatic)
- Replace gradient text with: font weight, size, or a single accent color
- If you must use a gradient, make it serve the mood (warm→warm, not rainbow), and use it on backgrounds, not text

**Test:** Remove the glass/gradient effect. Is information lost? If not, it was decoration. Remove it permanently.

### Tell 7: Dark mode with glowing neon accents

**What AI does:** Presents a dark background (#0a0a0a or #1a1a2e) with bright neon accents (cyan, purple, lime) that glow or have box-shadows.

**Why it's a tell:** This combination says "I'm making a tech product that looks futuristic" without actually being futuristic. It's the visual equivalent of saying "synergy."

**Replacements:**
- If dark mode is needed for the medium (code editor, dashboard, nighttime use), use warm dark backgrounds and desaturated accents
- Dark backgrounds: navy (#1a1a2e→#1c1917), warm gray (#292524), deep green (#0f1a14)
- Dark accents: muted versions of the accent hue, not full saturation neon
- Avoid glow effects (box-shadow with spread) — use flat color or subtle shadows instead
- Consider: do you need dark mode at all? Or are you using it because it "looks techy"?

**Test:** View the design in light mode. Is it still recognizable? If not, the dark mode is carrying the identity — and that identity is "neon on black."

### Tell 8: Everything fades in from below with the same timing

**What AI does:** Every section, card, and text block animates in with `translateY(20px)` → `0` + `opacity: 0` → `1` at 300-500ms, staggered by 100-200ms.

**Why it's a tell:** This animation pattern is the most common AI animation because it's safe, easy, and "feels polished." But when every element enters identically, the animation becomes noise, not signal. Motion should guide attention, not wallpaper the scroll.

**Replacements:**
- Vary animation type: some elements fade, some slide from the side, some scale, some simply appear
- Vary timing: fast for micro-interactions (100ms), standard for reveals (300ms), slow for emphasis (500ms+)
- Choose what to animate: animate the first element in a group, not every element. Animate the dominant element differently.
- Consider no animation at all — the boldest choice. Static design with intentional spacing and hierarchy communicates more than uniform motion.
- Always respect `prefers-reduced-motion`

**Test:** Remove all animations. Does the page lose information? If yes, those animations are functional. If no, the animations were decoration — either remove them or make them functional.

## Tell detection checklist

Run this checklist on any design before shipping:

```
AI TELL DETECTION

[ ] Font choice: Is the body font Inter, Roboto, Open Sans, or system-ui?
    → If yes: pick a font with personality or pair a distinctive heading face

[ ] Color palette: Does the primary palette involve cyan, purple-to-blue, or neon on dark?
    → If yes: rebuild from mood→hue→scheme (see color-palette skill)

[ ] Card grid: Is every content item in an identical card with icon+heading+text?
    → If yes: differentiate or replace some cards with other layouts

[ ] Alignment: Is most content center-aligned?
    → If yes: left-align body text and headings over 1 line

[ ] Spacing: Are all section gaps approximately the same size?
    → If yes: create proportional spacing (tight within groups, generous between)

[ ] Effects: Are glassmorphism, gradient text, or frosted overlays present?
    → If yes: replace with flat color and intentional contrast

[ ] Mode: Is this dark mode with bright neon/glowing accents?
    → If yes: warm the dark background, desaturate accents, remove glow

[ ] Motion: Does every element fade in from below with identical timing?
    → If yes: vary animation type and timing, or remove most animations
```

Scoring:
- 0 tells: Clean, likely intentional or very good AI-assisted work
- 1-2 tells:轻度 convergence — fix the tells you have and ship
- 3-4 tells: Moderate convergence — systematic replacement needed
- 5+ tells: Heavy convergence — redesign with anti-defaults in mind

## The anti-default mindset

Every AI tell shares the same root cause: **choosing the path of least resistance.** The fix isn't to pick the opposite of every default. The fix is to make **every choice defensible:**

1. "I chose this typeface because..." (if the reason is "it's popular" or "it looks clean," pick again)
2. "I used this color because..." (if the reason is "it's trendy" or "it looks techy," pick again)
3. "I laid this out this way because..." (if the reason is "it's the standard pattern," pick again)
4. "I animated this because..." (if the reason is "it looks polished," remove it or find its purpose)

The goal isn't to be weird for weird's sake. The goal is intentionality. When 75% of credibility judgments are design-based, and AI defaults converge, **intentional choices are the only way to establish credibility.**

## Quality checklist

- [ ] Zero AI tells remaining in the design
- [ ] Every major visual choice has a written justification
- [ ] At least 3 design choices are things AI would not default to
- [ ] The design has recognizable personality — someone could describe it in a few words
- [ ] Typefaces are chosen for the medium (screen, not print)
- [ ] Color palette maps to a mood, not a trend
- [ ] Motion is functional (removing it loses information), not decorative
- [ ] Layout has at least one asymmetric or unexpected element

## Anti-patterns I avoid

- Replacing AI defaults with different defaults — the goal is intentionality, not contrarianism
- Adding "character" through decoration alone — character should come from structural choices (type, color, composition)
- Going so far anti-default that the design becomes unusable — accessibility and clarity always win
- Checking only visual tells — AI convergence also happens in information architecture and content structure
- Assuming "not AI-generated" means "free of tells" — hand-coded designs can still have convergent patterns