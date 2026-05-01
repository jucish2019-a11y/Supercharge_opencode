---
name: micro-interactions
description: Design delightful animations, transitions, hover states, and loading experiences
---

## What I do

I design and implement delightful micro-interactions that make interfaces feel alive and responsive:

- Button, link, and interactive element hover/active/focus states
- Page and component transitions
- Loading states and progress indicators
- Success, error, and notification animations
- Scroll-triggered reveals and motion choreography

## When to use me

Use this skill when:
- Adding hover and interaction feedback to buttons, cards, and links
- Designing page transitions or component enter/exit animations
- Creating loading spinners, skeleton states, or progress bars
- Making notifications, toasts, and alerts feel polished
- Adding scroll-based reveals or staggered list animations
- You need a systematic approach to animation timing and easing
- For complex multi-element choreography, use the **motion-system** skill instead

## How I work

### Checker mode (auditing existing micro-interactions)

1. **Check interaction state coverage** — Does every interactive element have default, hover, focus, active, and disabled states?
2. **Check timing** — Are durations within the standard range? (50-100ms micro, 200-300ms standard, 500ms+ complex)
3. **Check easing** — Is `linear` used anywhere except progress bars? Are entries and exits using different easings?
4. **Check reduced motion** — Is `prefers-reduced-motion` respected?
5. **Check purpose** — Remove each animation. Do you lose information? If not, it was decoration.

### Applier mode (implementing micro-interactions)

1. **Identify interaction moments** — Map out every state for each interactive element: default, hover, active, focus, disabled, loading, success, error.
2. **Choose meaningful motion** — Every animation should communicate something:
   - Hover: "this is interactive"
   - Active/pressed: "you've engaged this"
   - Loading: "something is happening"
   - Transition: "where you came from and where you're going"
   - Success: "your action worked"
   - If the animation doesn't communicate, remove it.
3. **Apply timing principles**:
   - Micro-interactions (hover, press): 100-200ms
   - Transitions (expand, slide): 200-400ms
   - Elaborate motion (page transitions): 300-500ms
   - Easing: `ease-out` for entrances, `ease-in` for exits, `ease-in-out` for state changes
4. **Implement progressively** — Add the base state first, then hover, then focus, then transitions. Motion is an enhancement.
5. **Respect preferences** — Check `prefers-reduced-motion` and disable/reduce animations accordingly.

## Eight states every interactive element needs

Every button, link, input, and card should have these states defined:

| State | Visual | Trigger | Duration |
|-------|--------|---------|----------|
| Default | Resting appearance | — | — |
| Hover | Subtle background shift or shadow lift | Mouse hover | 150ms |
| Focus | 2px outline ring, offset 2px from element | Keyboard tab | 0ms (instant) |
| Active | Slightly darker than hover, or scale(0.97) | Mouse down / key press | 100ms |
| Disabled | 50% opacity, not-allowed cursor, gray text | :disabled attribute | — |
| Loading | Spinner replaces or prefixes text, button is non-interactive | Async action | Persistent |
| Error | Red border or shake animation | Validation failure | 300ms |
| Success | Brief checkmark or green flash | Action completed | 500ms |

**Why this matters:** Most UIs only implement default and hover. That means keyboard users see nothing on focus, screen readers get no feedback on actions, and users have no confirmation that their action succeeded. The 8-state model covers every user interaction path.

## Interaction patterns

### Buttons

```css
.button {
  transition: background-color 150ms ease-out,
              transform 100ms ease-out,
              box-shadow 150ms ease-out;
}
.button:hover {
  background-color: var(--color-primary-hover);
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
.button:active {
  transform: scale(0.97);
  background-color: var(--color-primary-active);
}
.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}
```

### Cards

```css
.card {
  transition: box-shadow 200ms ease-out, transform 200ms ease-out;
}
.card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transform: translateY(-2px);
}
```

### Page transitions

- Fade + slight translate for page entries
- Stagger list item entries by 50ms each
- Use `transform` and `opacity` only (GPU-accelerated, no layout thrashing)

### Loading states

```
States by wait time:

<1s:  Inline spinner next to button text (small, 16px)
1-3s: Skeleton screen (preferred for content loading)
3-10s: Progress bar or percentage indicator
>10s: Progress bar with status message + estimated time

Why this matters: An unresponsive UI feels broken. A spinner on
a 5-second load feels worse than a skeleton that shows progress.
Match the feedback to the wait time.
```

### Error/Success feedback

```
Error: Shake horizontally (translateX oscillation, 300ms)
  - The shake says "no" — it's physically intuitive

Success: Scale up slightly then settle (scale 1 → 1.05 → 1, 200ms)  
  - The scale says "yes" — a small celebration
  - Or: brief checkmark icon fade-in (300ms)
```

## Duration and easing reference

```
MICRO (50-100ms):
  - Color changes on hover
  - Ripple effects
  - Toggle switches
  - Focus ring appearance

STANDARD (150-300ms):
  - Button hover/active
  - Dropdown open/close
  - Tooltip appear/disappear
  - Card hover lift
  - Collapse/expand

COMPLEX (300-500ms):
  - Page transitions
  - Modal appear/disappear
  - Toast slide-in
  - Skeleton → content swap

NEVER:
  - >500ms for interactive elements (feels sluggish)
  - <50ms (imperceptible, wastes performance budget)
  - linear easing for UI motion (feels robotic)
```

Easing rules:
- **Entrances** (things appearing): ease-out — fast start, gentle stop
- **Exits** (things disappearing): ease-in — slow start, quick departure
- **State changes** (color, size): ease-in-out — smooth both ways
- **Spring/overshoot**: only for playful/brand moments — not for utilitarian UI

```css
--ease-standard:    cubic-bezier(0.4, 0.0, 0.2, 1.0);
--ease-decelerate:  cubic-bezier(0.0, 0.0, 0.2, 1.0);
--ease-accelerate:  cubic-bezier(0.4, 0.0, 1.0, 1.0);
```

## Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**Why this matters:** About 5% of users experience vestibular disorders that make animations cause nausea or dizziness. `prefers-reduced-motion` isn't optional — it's an accessibility requirement. Reduce to opacity-only transitions, never remove all feedback.

## Performance rules

- Only animate `transform` and `opacity` — they're GPU-composited and don't cause layout reflow
- Never animate `width`, `height`, `margin`, `padding`, `top`, `left` — these cause layout thrashing
- Use `will-change` sparingly (only on elements about to animate, remove after)
- Keep animated element count under 20 simultaneous animations
- Use `contain: layout` on animated containers to limit paint scope

## Quality checklist

- [ ] Every interactive element has default, hover, focus, active, disabled states
- [ ] Focus states use `:focus-visible` (not `:focus`) — avoids showing on click
- [ ] Focus ring color contrasts with the element background
- [ ] Disabled elements have both visual AND behavioral indicators (opacity + cursor)
- [ ] Loading states exist for all async operations
- [ ] Skeleton screens preferred over spinners for content loading (>1s)
- [ ] Error feedback includes both visual (shake, red) and text message
- [ ] Success feedback is brief but noticeable (checkmark, 500ms)
- [ ] `prefers-reduced-motion` is respected — reduce, don't remove all feedback
- [ ] Only `transform` and `opacity` are animated (never layout properties)
- [ ] Durations are appropriate (100ms micro, 200-300ms standard, 300-500ms complex)
- [ ] Easings match intent: ease-out entrances, ease-in exits
- [ ] No animation runs longer than 500ms for interactive elements
- [ ] Remove each animation: if information isn't lost, it was decoration

## Anti-patterns I avoid

- Animating layout properties (width, height, margin, padding, top, left) — causes reflow and jank
- Using `linear` easing for any UI motion — it feels robotic
- Stagger delays above 100ms per item — feels laggy
- Multiple competing animations on the same view — visual chaos
- Duration above 500ms for any interactive element — feels broken
- Ignoring `prefers-reduced-motion` — accessibility violation
- Bouncing or overshooting on dismissive actions (delete, close, cancel) — semantically wrong
- Adding animation that doesn't communicate information — if removing it changes nothing, remove it
- Using `!important` to override animation — fix the specificity, don't override
- Hover-only states on mobile — there's no hover on touch; use `@media (hover: hover)`
- Showing focus rings on mouse click — use `:focus-visible`, not `:focus`