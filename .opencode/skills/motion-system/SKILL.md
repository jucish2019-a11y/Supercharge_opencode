---
name: motion-system
description: Design cohesive motion systems with choreography, easing curves, duration scales, and orchestrated transitions that feel intentional and alive
---

## What I do

I design systematic motion languages that make interfaces feel physically coherent, emotionally resonant, and premium-quality — matching the polish level of Google's Stitch:

- **Easing system** — Custom cubic-bezier curves for different motion intents
- **Duration scale** — Consistent timing that maps to physical intuition
- **Choreography** — Orchestrated entry/exit sequences for groups and pages
- **State transitions** — Meaningful morphing between element states
- **Physics-based motion** — Springs, momentum, and natural deceleration curves

## When to use me

Use this skill when:
- Creating or refining a motion/animation design system
- Making transitions feel cohesive across a product instead of ad-hoc
- Designing page transitions, list animations, or staggered reveals
- Making an interface feel "premium" or "polished" through motion
- Animating layout changes, element appearances, or state changes

## How I work

1. **Define the motion language** — Choose what motion means in this product. Is it playful? Efficient? Bold? Calm? Motion personality must match brand personality.
2. **Build the easing library** — Define a small set of easing curves, each with a purpose:
3. **Set the duration scale** — Map durations to interaction types:
4. **Define choreography primitives** — Stagger, cascade, and orchestration patterns.
5. **Implement physics-based animations** — Spring animations for interactive elements.
6. **Apply motion systematically** — Every animation in the product uses the same system.

## Easing library

Every easing curve has a name, a purpose, and a CSS value:

```
--ease-standard:  cubic-bezier(0.4, 0.0, 0.2, 1.0)   /* Default UI transitions — smooth, natural */
--ease-decelerate: cubic-bezier(0.0, 0.0, 0.2, 1.0)    /* Entering elements — fast start, gentle stop */
--ease-accelerate: cubic-bezier(0.4, 0.0, 1.0, 1.0)   /* Exiting elements — slow start, quick exit */
--ease-sharp:     cubic-bezier(0.4, 0.0, 0.6, 1.0)    /* Quick state changes — snappy, responsive */
--ease-spring:    cubic-bezier(0.175, 0.885, 0.32, 1.275) /* Playful overshoot — bouncy, energetic */
--ease-elastic:   cubic-bezier(0.68, -0.55, 0.265, 1.55) /* Emphatic overshoot — attention-grabbing */
```

Usage rules:
- **Standard** for most things: color changes, opacity, background shifts
- **Decelerate** for things entering the screen: modals, dropdowns, page content
- **Accelerate** for things leaving: dismissals, collapses, fade-outs
- **Sharp** for toggles, checkboxes, quick state flips
- **Spring** only for playful/brand moments — not for utilitarian interactions
- **Elastic** almost never — only for dramatic emphasis (like a successful action celebration)

## Duration scale

```
--duration-instant:  50ms    /* Micro-feedback: ripples, toggles, color flashes */
--duration-fast:    100ms   /* Quick response: hover, focus, icon state changes */
--duration-normal:  200ms   /* Standard transitions: dropdowns, tooltips, expands */
--duration-moderate: 300ms  /* Noticeable motion: slide-ins, modals, panel reveals */
--duration-slow:    500ms   /* Complex motion: page transitions, layout shifts */
--duration-scenic:   700ms  /* Decorative motion: hero animations, onboarding reveals */
```

Rules:
- Never use durations above 700ms for interactive elements — it feels sluggish
- Never use durations below 50ms — it's imperceptible and wastes performance budget
- Small elements (icons, badges, buttons): use fast–normal
- Medium elements (cards, panels, menus): use normal–moderate
- Large elements (pages, modals, full-screen): use moderate–slow
- Duration should be proportional to distance traveled: far = longer, near = shorter

## Choreography patterns

### Stagger (sequential entry)
Elements enter one after another with a fixed delay:
```css
.item { animation: fadeSlideIn 300ms var(--ease-decelerate) both; }
.item:nth-child(1) { animation-delay: 0ms; }
.item:nth-child(2) { animation-delay: 50ms; }
.item:nth-child(3) { animation-delay: 100ms; }
```
- Stagger delay: 40-80ms between items
- Use `--ease-decelerate` for natural entry
- Maximum total stagger duration: 500ms (cap the cascade)

### Cascade (hierarchical entry)
Parent enters first, then children, with grouped stagger:
```css
.container { animation: fadeIn 300ms var(--ease-decelerate) both; }
.container .header { animation-delay: 100ms; }
.container .content { animation-delay: 200ms; }
.container .footer { animation-delay: 300ms; }
```
- Parent establishes context before children appear
- Each group has its own stagger within the cascade

### Orchestrated exit
Exits should be faster than entries — people want to leave quickly:
- Entry duration: 300ms (ease-decelerate)
- Exit duration: 200ms (ease-accelerate)
- Exit simultaneously, not staggered — don't make users wait for departure

## State transition patterns

### Morph (shared element transitions)
An element transforms from one state to another with shared geometry:
```css
.button {
  transition: all 200ms var(--ease-standard);
}
```
- Use for: button → loading, card → expanded, thumbnail → full image
- Animate `transform` and `opacity` for performance
- Match border-radius, background, and shadow simultaneously

### Cross-fade (content replacement)
Old content fades out while new content fades in:
- Fade out: 150ms ease-accelerate
- Fade in: 200ms ease-decelerate
- Slight overlap (50ms) for seamless blend
- Use for: tab content, carousel slides, filter results

### Slide + fade (contextual entrance)
Elements slide in from the direction of their source with a fade overlay:
- From below (sheets, modals): translateY(20px) → 0
- From right (pages, details): translateX(20px) → 0
- From center (alerts, toasts): scale(0.95) → 1
- Always combine with opacity: 0 → 1

## Physics-based motion

For interactive, continuous motion (drag, spring-back, momentum scrolling):
- Use spring animations with `stiffness: 300, damping: 30` for snappy UI
- Use `stiffness: 200, damping: 20` for playful/elastic UI
- Spring duration should feel natural: 300-600ms settle time
- Only use spring for direct manipulation (drag, pull-to-refresh, dismissible sheets)
- All other motion uses the easing library above

## Motion principles (Stitch quality)

1. **Purposeful** — Every animation communicates. If it doesn't inform or guide, remove it.
2. **Coherent** — All motion in the product follows the same timing and easing system. No outliers.
3. **Responsive** — Interactive feedback must arrive within 100ms. Perception of instant = trust.
4. **Natural** — Objects decelerate when entering, accelerate when leaving. Real physics metaphor.
5. **Efficient** — Animate only `transform` and `opacity`. Composite-layer properties only. No layout thrashing.
6. **Inclusive** — Always respect `prefers-reduced-motion`. Reduce to opacity-only, never remove all feedback.
7. **Layered** — Background, content, and overlay layers move at different speeds for parallax depth.

## Anti-patterns I avoid

- Animating `width`, `height`, `margin`, `padding`, `top`, `left` — causes layout reflow
- Using `linear` easing for any UI motion — it feels robotic
- Stagger delays above 100ms per item — feels laggy
- Multiple competing animations on the same view — visual chaos
- Duration above 700ms for any interactive element — feels broken
- Ignoring `prefers-reduced-motion` — accessibility violation
- Bouncing or overshooting on dismissive actions (delete, close, cancel) — semantically wrong