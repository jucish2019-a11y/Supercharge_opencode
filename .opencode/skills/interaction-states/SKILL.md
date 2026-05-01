---
name: interaction-states
description: Design complete state machines for interactive elements — 12+ states, transition timing, frequency-novelty decisions, and gesture-aware patterns that separate premium UI from generic
---

## What I do

I define how every interactive element responds to user input across all possible states. Most interfaces define 2-3 states (default, hover, active). Premium interfaces like Linear, Stripe, and Apple's HIG define 12+ states with specific visual expressions, timing curves, and transition rules for every state change.

- Map the complete state machine for any interactive element (buttons, cards, inputs, toggles, navigation)
- Specify visual expressions per state (color, opacity, scale, shadow, border, background)
- Define transition timing for every state change (entry curve, exit curve, duration)
- Apply the frequency-novelty matrix (which interactions get animation investment)
- Handle gesture-tracked states (interruptible transitions, real-time tracking)
- Ensure selection highlighting, hover intent, and cursor feedback respect the brand palette

## When to use me

Use this skill when:
- Building interactive components that need to feel premium (not just functional)
- Defining a design system's interactive layer (state tokens, transition specs)
- An interface feels "flat" or "mechanical" despite good visual design
- Hover, focus, and active states feel generic or inconsistent
- Animation feels uniform (everything animates equally, or nothing does)

For related but distinct work:
- **micro-interactions** — The 8-state model for common UI patterns (buttons, cards, toggles)
- **motion-system** — Easing curves, duration scales, choreography patterns
- **premium-motion** — Spring physics, scroll-linked animation, container transforms
- **interaction-design** — Drag-and-drop, gesture handling, command palette, keyboard shortcuts

This skill provides the state machine specification; micro-interactions provides the pattern implementations; motion-system provides the timing curves; premium-motion provides the physics engine.

## How I work

### Checker mode (auditing existing interaction design)

1. **State inventory** — For each interactive element, list every defined state. Are any of the 12 standard states missing?
2. **Transition audit** — For each state change, is there a defined curve and duration? Entry and exit should often be asymmetric.
3. **Frequency audit** — Are high-frequency interactions animated heavily? Are low-frequency interactions animated at all?
4. **Consistency check** — Do similar elements (all buttons, all cards) share the same state machine?
5. **Accessibility audit** — Are focus states visible? Do states work without hover? Is state conveyed through more than color?
6. **Interruption audit** — Can transitions be interrupted mid-flight? What happens when a user reverses direction during a transition?

### Applier mode (designing interaction states from scratch)

#### Step 1: Select the state machine for your element type

**Core states** (every interactive element):

| # | State | Description | Visual cue |
|---|-------|-------------|------------|
| 1 | Default | Resting, no interaction | Base background, no ring |
| 2 | Hover | Cursor over, not pressing | Light background shift, no ring |
| 3 | Active/Pressed | Finger or mouse down | Darker background, subtle scale(0.97), no ring |
| 4 | Focused | Keyboard focus | Ring outline (2-3px, theme color) |
| 5 | Focused + Hovered | Keyboard focus AND cursor over | Ring + hover background |
| 6 | Selected/Current | Toggle is on, tab is active | Filled background, accent indicator |

**Extended states** (context-dependent):

| # | State | Description | Visual cue |
|---|-------|-------------|------------|
| 7 | Disabled | Cannot interact, grayed out | 40% opacity, no-pointer cursor, no ring on focus |
| 8 | Loading | Async action in progress | Spinner or pulse animation, no-pointer cursor |
| 9 | Error | Validation failure | Error border, error message, shake animation |
| 10 | Success | Action completed | Brief accent flash, checkmark, or subtle scale pulse |

**Advanced states** (premium interactions):

| # | State | Description | Visual cue |
|---|-------|-------------|------------|
| 11 | Drag | Being dragged by user | Opacity 80%, slight shadow lift, scale(1.02) |
| 12 | Drop target | Valid drop zone during drag | Dashed border, subtle background highlight |

Not every element needs all 12. Core (1-6) is mandatory. Extended (7-10) depends on element type. Advanced (11-12) only for draggable interfaces.

#### Step 2: Define visual expressions per state

For each state, specify these visual properties:

```
Button state machine example:

Default:    bg: var(--surface)        text: var(--on-surface)      border: 1px solid var(--outline)
Hover:      bg: var(--surface-hover)  text: var(--on-surface)      border: unchanged
Active:     bg: var(--surface-active) text: var(--on-surface)      border: unchanged    scale: 0.98
Focused:    bg: unchanged            text: unchanged               ring: 2px var(--primary)
Disabled:   bg: var(--surface-dim)   text: var(--on-surface-dim)  border: var(--outline-dim)  opacity: 0.4
Loading:    bg: var(--surface)        text: unchanged               progress: 16px spinner
Error:      bg: var(--error-container) text: var(--error)           border: var(--error)
```

Design token references (not raw values) ensure the state machine works across themes (light, dark, high-contrast).

#### Step 3: Define transition timing for every state change

Every state change is a two-part transition: **exit from old state, enter new state**. These are often asymmetric.

| From → To | Entry duration | Entry curve | Exit duration | Exit curve |
|-----------|---------------|-------------|--------------|------------|
| Default → Hover | 120ms | ease-out | — | — |
| Hover → Default | — | — | 80ms | ease-in |
| Default → Focused | 0ms | instant | — | — |
| Any → Active | 60ms | ease-out | — | — |
| Active → Hover | — | — | 100ms | ease-in |
| Any → Loading | 200ms | ease-in-out | — | — |
| Loading → Success | 300ms | spring(1, 170, 26) | — | — |
| Any → Error | 50ms | ease-out | — | — |
| Error shake | 400ms | ease-in-out | — | — |

**Asymmetry principle**: Entries into positive states (hover, focus) are slightly slower than exits. Entries into attention states (error, active) are faster than exits. This creates "alive" feeling interfaces.

**Frequency principle**: High-frequency state changes (hover, focus) should be fast (60-120ms). Low-frequency state changes (loading → success, error) can be slower with more personality (200-500ms).

#### Step 4: Apply the frequency-novelty matrix

Not every interaction deserves animation investment. The matrix determines where to invest:

| | Low novelty (common) | High novelty (rare) |
|---|---|---|
| **High frequency** (100+ uses/session) | **Instant or near-instant** (0-50ms). Context menus, tab switches, button presses, command palette. No animation. | **Quick with character** (100-200ms). Opening a project, completing a task, saving. Brief but recognizable. |
| **Low frequency** (1-5 uses/session) | **Standard** (200-400ms). Logging out, changing settings, opening preferences. Purposeful but not dramatic. | **Invested** (400-800ms). First-time onboarding, project creation, major state transitions. Full choreography. |

**Rules**:
- Command palettes (Cmd+K): appear instantly, no animation. Speed is the feature.
- Context menus (right-click): appear instantly (0ms), dismiss with subtle fade (100ms).
- Button hovers: 80-120ms. Fast because they happen constantly.
- View transitions (navigating between pages): 300-500ms. This establishes spatial relationships.
- First-time experiences (onboarding, empty state → populated): 500-800ms. Make it memorable.
- Success confirmations: 200-300ms with spring physics. Brief but satisfying.

#### Step 5: Design hover intent detection

Premium interfaces don't trigger hover states the instant the cursor passes over an element. They detect intent:

```css
.hover-intent {
  transition: background-color 150ms 50ms;
  /* 50ms delay before hover activates — accidental passes don't trigger */
}

.hover-intent:hover {
  background-color: var(--surface-hover);
  transition-delay: 0ms;
  /* Instant exit — no delay when leaving */
}
```

Implementation options:
- **CSS-only**: Use `transition-delay` on enter, `0ms` on exit (as above)
- **JS hover intent**: Track cursor velocity. Only trigger hover if cursor slows near the element (velocity below threshold for 50ms+)
- **Proximity**: Trigger hover when cursor enters a 4px extended hitbox around the element, but animate over 150ms

#### Step 6: Design selection highlighting

When users select text, the highlight color should match the brand palette, not the browser default:

```css
::selection {
  background-color: var(--primary-alpha-20);
  color: var(--on-primary-alpha-20);
}
```

Where `--primary-alpha-20` is the primary brand color at 20% opacity. This works on both light and dark backgrounds.

For different UI contexts:
- **Input selection**: Use primary color at 30% opacity
- **Code selection**: Use a warm/cool neutral (not primary) at 20% opacity for readability
- **Error-state selection**: Use error color at 20% opacity

#### Step 7: Define gesture-tracked states

For touch and drag interactions, states need to be interruptible and spatially aware:

**Touch feedback pattern**:
- Touch start: immediate visual feedback (scale 0.97, background shift) within 1 frame
- Touch hold (300ms+): subtle pulse or secondary feedback indicating "hold detected"
- Touch move: if dragging begins, element lifts (shadow increase, scale 1.02) and follows finger
- Touch end: if short tap, trigger action. If drag, handle drop.

**Interruptible transition pattern**:
- All view transitions must be cancellable mid-animation
- If user reverses direction (e.g., swipes back during a forward transition), the animation reverses from its current position
- Never queue animations — if a new state arrives while transitioning, switch targets immediately
- Spring physics handle this naturally (they converge from any position)

**Keyboard state tracking**:
- Tab into an element: focus ring appears instantly (0ms)
- Tab out: focus ring disappears with 80ms fade (so it doesn't feel like it "vanished")
- Key press on focused element: active state immediately (0ms entry, important for keyboard users who can't see hover)

#### Step 8: Document the state machine

For each component type, document:

```
Component: Primary Button
States: default, hover, active, focused, focused+hovered, selected, disabled, loading, error
Transitions:
  default→hover: 120ms ease-out
  hover→default: 80ms ease-in
  default→focused: instant (0ms)
  any→active: 60ms ease-out
  active→hover: 100ms ease-in
  loading→success: 300ms spring(1, 170, 26)
  error enter: 50ms ease-out + 400ms shake
Accessibility:
  Focus ring: 2px solid var(--primary), 2px offset
  Error: not color alone (icon + text + border)
  Disabled: opacity 0.4, aria-disabled="true", no pointer events
```

## State machine templates

### Button (core + extended)

```
States: default, hover, active, focused, focused+hovered, disabled, loading
Hover delay: 50ms enter, 0ms exit
Active detection: mousedown/touchstart
Loading: replace content with spinner, maintain width
```

### Card (core + advanced)

```
States: default, hover, active, focused, dragging, drop-target
Hover: subtle lift (transform: translateY(-2px), shadow increase)
Focused: focus ring on card, scroll into view if needed
Dragging: opacity 80%, scale 1.02, shadow lift
Drop target: dashed border, background highlight
```

### Toggle/Switch (core + selected)

```
States: default-off, default-on, hover-off, hover-on, active-off, active-on, focused
Transition between states: 200ms spring physics for the thumb movement
Background crossfade: 150ms for track color change
```

### Input (core + extended)

```
States: default, focused, hovered, focused+hovered, error, disabled, loading
Focused: border color shift + subtle ring
Error: border + error message + shake on enter
Loading: spinner in the input right side, input still editable
```

### Navigation item (core + selected)

```
States: default, hover, active, selected, focused
Selected: persistent background indicator, accent left-border or bottom-border
Hover on selected: slightly brighter background (not the same as unselected hover)
```

## Anti-patterns I avoid

- Animating high-frequency interactions — context menus and command palettes should appear instantly
- Using the same duration for enter and exit — exits should be faster than entries for positive states
- Triggering hover on cursor pass-through — use hover intent detection (50ms delay minimum)
- Only defining :hover and :focus — the 6 core states are minimum for any interactive element
- Conveying state through color alone — use shape, icon, text, and position changes too
- Applying the same animation pattern everywhere — the frequency-novelty matrix determines investment
- Queueing animations when a new state arrives — switch targets immediately from current position
- Using browser-default selection highlighting — brand the selection color
- Treating disabled elements as "just grayed out" — reduce opacity, remove pointer events, add aria-disabled
- Forgetting focused+hovered state — keyboard users who also navigate with mouse need this combined state