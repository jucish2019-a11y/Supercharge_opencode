---
name: animation-choreography
description: Orchestrate multi-element animation sequences — page transitions, shared element morphing, staggered reveals, coordinated exits, and state-driven animation graphs for premium UI motion
---

## What I do

I design orchestrated animation sequences where multiple elements move in concert, creating the perception of continuity, spatial relationships, and narrative flow — the hallmark of premium apps:

- **Page transitions** — Smooth navigation between views with shared elements
- **Shared element transitions** — Elements morph from one context to another (hero animations)
- **Staggered orchestrations** — Grouped entry/exit sequences with timing coordination
- **State-driven animation graphs** — Declarative state machines for complex multi-state animations
- **Layout animations** — Smooth reflow when elements are added, removed, or reordered
- **Scroll-driven animation** — Parallax, reveal-on-scroll, sticky element transitions

## When to use me

Use this skill when:
- Building page transitions that feel like physical navigation (not just route swaps)
- An element appears in a new context and should visually connect to its origin
- Multiple elements need to enter or exit in a coordinated sequence
- Layout changes (filter, sort, add/remove) need smooth reflow instead of jarring repositioning
- Scroll position should drive animation (parallax, progress, reveals)
- The UI needs to feel spatial — like elements exist in a physical environment

## How I work

1. **Map the transition narrative** — What's the spatial story? "The user clicked a card, so the card expands to become the detail page" is a narrative. "Route A unmounts, Route B mounts" is not.
2. **Identify shared elements** — What visual elements persist across the transition? Those become shared element transitions.
3. **Choreograph the sequence** — Define the order and timing of all animated elements during entrance, persistence, and exit.
4. **Define the animation graph** — For complex multi-state components, model transitions as a state machine.
5. **Implement with the platform's animation primitives** — View Transitions API, Framer Motion, React Spring, FLIP technique.
6. **Optimize for 60fps** — Only animate `transform` and `opacity`. Use `will-change` sparingly. Batch reads and writes.

## Page transition patterns

### Slide (directional navigation)

The new page enters from the direction of the user's action:

```
Forward navigation:  New page slides in from right, old page slides out left
Back navigation:     New page slides in from left, old page slides out right
Down navigation:     New page slides up from bottom, old page slides down (drill-down)
Up navigation:       New page slides down from top, old page slides up (going back)
```

Timing:
```
New page entry: 300ms ease-decelerate (cubic-bezier(0, 0, 0.2, 1))
Old page exit:  250ms ease-accelerate (cubic-bezier(0.4, 0, 1, 1))
Overlap: 50ms — the new page starts entering slightly before the old finishes exiting
```

Implementation:
```css
/* View Transitions API (modern browsers) */
::view-transition-old(root) {
  animation: slide-out-left 250ms ease-accelerate;
}
::view-transition-new(root) {
  animation: slide-in-right 300ms ease-decelerate;
}

@keyframes slide-in-right {
  from { transform: translateX(100%); opacity: 0.8; }
  to { transform: translateX(0); opacity: 1; }
}
@keyframes slide-out-left {
  from { transform: translateX(0); opacity: 1; }
  to { transform: translateX(-30%); opacity: 0.6; }
}
```

Rules:
- Never slide the old page 100% off-screen — it creates a blank gap. Slide it 20-30% and fade to 60% opacity.
- Forward = right, back = left. Always. This matches the spatial model (like pages in a book).
- On mobile: the new page covers 100% height. On desktop: the transition can be contained to a content area (not the sidebar).

### Cross-fade (context switch)

For non-directional navigation (tab switching, modal overlay, search results):

```
Old content: opacity 1 → 0, 150ms ease-accelerate
New content: opacity 0 → 1, 200ms ease-decelerate
Gap: none — 50ms overlap for seamless blend
```

Variations:
- **Pure cross-fade**: Simple opacity swap. Subtle, professional.
- **Scale-fade**: Old content scales to 0.98 and fades, new content scales from 1.02 and fades in. Adds depth.
- **Slide-fade**: Old content slides 8px in the direction of the new content and fades. Adds direction.

### Container morph (shared layout)

The container of the old view morphs into the container of the new view:

```
Old: Card at (x:20, y:100, w:300, h:200, r:12)
New: Full-width panel at (x:0, y:0, w:100%, h:100%, r:0)

Animation: 300ms ease-decelerate
All properties animate simultaneously:
- Position: (20,100) → (0,0)
- Size: (300×200) → (100%×100%)
- Border-radius: 12px → 0px
- Shadow: card-shadow → none
- Content: cross-fade at 150ms
```

## Shared element transitions

A shared element persists visually across a navigation boundary — it's the same element, just in a new context.

### Card → Detail (hero expand)

```
Card state:
  Position: (20, 140)
  Size: (340, 200)
  Border-radius: 12px
  Shadow: 0 1px 3px rgba(0,0,0,0.08)
  Image: thumbnail (object-fit: cover)
  Title: 16px, 600

Detail state:
  Position: (0, 0) — full width
  Size: (100%, 400)
  Border-radius: 0px
  Shadow: none
  Image: hero (object-fit: cover)
  Title: 32px, 700

Transition:
  Duration: 400ms
  Easing: cubic-bezier(0.4, 0, 0.2, 1) — standard
  Image: cross-fade at 200ms (thumbnail → full)
  Title: position + size + weight animate simultaneously
  Border-radius: 12 → 0 linear

Content below the hero:
  Entry: stagger from 0ms offset, each 50ms apart
  Fade in + translateY(16px) → 0
  Duration: 200ms each, ease-decelerate
  
  (The content enters AFTER the hero has mostly expanded,
   so the content appears to emerge from within the hero)
```

### List item → Detail (row expansion)

```
List item state:
  Position: (0, rowY)
  Size: (100%, 64px)
  Background: transparent

Detail state:
  Position: (0, 0) — full screen or detail panel
  Size: (100%, 100%)
  Background: bg-primary

Transition:
  Duration: 350ms
  Height: 64px → 100%
  Other list items: fade out 150ms, then slide to close the gap
  Detail content: enter after 200ms delay (stagger, 40ms each)
```

### Avatar → Profile header (shared identity)

```
Avatar in list state:
  Position: (20, rowY + 12)
  Size: (40, 40)
  Border-radius: 50%

Profile header state:
  Position: (24, headerY)
  Size: (64, 64)
  Border-radius: 50%

Transition:
  Duration: 300ms
  Position: animate x and y
  Size: animate scale (1 → 1.6)
  Shadow: none → 0 4px 12px rgba(0,0,0,0.1)
  
Both the source and destination avatars must be:
- The same image (same URL, same aspect ratio)
- Positioned with `position: absolute` for the transition
- Hidden/shown before/after the transition
```

### Implementation: View Transitions API

```js
// Modern shared element transitions
document.startViewTransition(async () => {
  // Update the DOM
  updateContent();
  
  // The browser captures the old state and new state
  // and animates between them
});

// Name shared elements for cross-fade
// CSS: view-transition-name: hero-image;
// The element with this name in old and new states will morph
```

### Implementation: Framer Motion (React)

```tsx
import { motion, AnimatePresence, LayoutGroup } from 'framer-motion';

// Shared layout animation — element morphs when layout changes
<motion.div layout layoutId="card-123">
  <img src="hero.jpg" />
  <h2>Title</h2>
</motion.div>

// When navigated to detail, the same layoutId element morphs
<motion.div layout layoutId="card-123">
  <img src="hero.jpg" />
  <h2>Title</h2>
</motion.div>
```

## Staggered orchestrations

### List reveal (sequential entry)

Elements enter one after another, creating a wave effect:

```
Timing:
  Each element: 200ms fade + slide (opacity 0→1, translateX(20px)→0)
  Stagger delay: 40ms between elements
  Easing: ease-decelerate (cubic-bezier(0, 0, 0.2, 1))
  
Element 1: 0ms start → 200ms end
Element 2: 40ms start → 240ms end
Element 3: 80ms start → 280ms end
...
Element N: (N-1)×40ms start → (N-1)×40 + 200ms end

TOTAL: Never exceed 600ms for any stagger sequence
For >10 items: cap the stagger at 400ms, then all remaining items enter together
```

Implementation:
```css
.list-item {
  animation: fadeSlideIn 200ms var(--ease-decelerate) both;
  animation-delay: calc(var(--index) * 40ms);
}

/* Cap total stagger duration */
.list-item:nth-child(n+11) {
  animation-delay: 400ms;
}
```

### Grouped cascade (hierarchical entry)

Groups enter in sequence, elements within groups stagger:

```
Page load sequence:
  0ms:   Header fades in (200ms)
  100ms: Navigation bar slides down (200ms)
  200ms: KPI cards enter (4 cards, 40ms stagger each)
  400ms: Primary chart fades + scales in (300ms)
  500ms: Secondary content enters (200ms)
  600ms: Footer slides up (200ms)

Total page reveal: ~800ms — feels instant but orchestrated
```

Rules:
- The most important content enters first
- Supporting content enters second
- Decorative/minimal content enters last
- Every group uses the same entry pattern (fade + slight movement)
- Maximum total page transition: 800ms
- Maximum element transition duration: 300ms (except hero morphs at 400ms)

### Orchestrated exit (simultaneous departure)

Exits are NOT staggered — they happen simultaneously and faster than entries:

```
Exit sequence:
  ALL elements: begin exit at 0ms
  Header: fade out (150ms ease-accelerate)
  Content: fade + slide down (150ms ease-accelerate)
  Footer: fade + slide down (150ms ease-accelerate)

Total exit: 150ms — fast, because leaving is urgent
```

Rules:
- Users want to leave quickly — exit is always faster than entry (150ms vs 300ms)
- All elements exit simultaneously — don't stagger exits
- Use ease-accelerate for exits (slow start, fast end)
- If the new page has shared elements, morph them while everything else exits

## State-driven animation graphs

For components with multiple states and complex transitions:

### State machine model

```
States: collapsed, expanding, expanded, collapsing, dragging

Transitions:
  collapsed → expanding:   trigger = click
  expanding → expanded:    trigger = animationEnd (300ms)
  expanded → collapsing:  trigger = click or dragEnd (below threshold)
  collapsing → collapsed: trigger = animationEnd (200ms)
  expanded → dragging:    trigger = dragStart
  dragging → expanded:    trigger = dragEnd (above threshold) — spring animation
  dragging → collapsing:  trigger = dragEnd (below threshold) — spring animation
```

### Spring physics for state transitions

Interactive state changes should feel physical:

```js
// Spring configuration
const springConfig = {
  stiffness: 300,   // Higher = faster movement
  damping: 30,      // Higher = less bounciness
  mass: 1,          // Default
};

// For snappy (tool-like): stiffness: 400, damping: 30
// For playful (consumer): stiffness: 200, damping: 20
// For heavy (editorial):  stiffness: 150, damping: 25
// For gentle (ambient):   stiffness: 100, damping: 20
```

Spring rules:
- Use springs for all touch-driven, drag-driven, and gesture-driven animations
- Use timed easing for all programmatic, state-driven animations
- Springs should settle within 500ms — if they take longer, increase damping
- Never combine spring and timed animations on the same element — pick one

### Interruptible animations

Animations must be interruptible — when a user acts, respond immediately:

```js
// When the user clicks during an ongoing animation:
// 1. Stop the current animation at its current value
// 2. Start the new animation from that value to the new target
// 3. Never jump to the end of the current animation, then start the new one

// Framer Motion handles this automatically
// With CSS: use animation-fill-mode: both and update the from/to values
```

## Layout animation (FLIP technique)

When elements are added, removed, or reordered, animate the reflow:

### FLIP (First, Last, Invert, Play)

```
1. FIRST: Record the element's current position and size
2. LAST:  Apply the layout change instantly (remove element, reorder, filter)
3. INVERT: Calculate the difference and apply a transform to make the element
          appear in its old position
4. PLAY:  Animate from the inverted (old) position to the new position
```

```js
// FLIP implementation
function flipAnimate(element) {
  // FIRST — record starting position
  const first = element.getBoundingClientRect();
  
  // Apply the DOM change
  doLayoutChange();
  
  // LAST — record ending position
  const last = element.getBoundingClientRect();
  
  // INVERT — calculate the delta
  const deltaX = first.left - last.left;
  const deltaY = first.top - last.top;
  const deltaW = first.width / last.width;
  const deltaH = first.height / last.height;
  
  // Apply inverse transform (element appears in old position)
  element.style.transform = `translate(${deltaX}px, ${deltaY}px) scale(${deltaW}, ${deltaH})`;
  element.style.transformOrigin = 'top left';
  
  // PLAY — animate to new position
  requestAnimationFrame(() => {
    element.style.transition = 'transform 300ms ease-decelerate';
    element.style.transform = '';
  });
}
```

### Adding elements

```
New element:
  Entry: scale(0.8) + opacity(0) → scale(1) + opacity(1)
  Duration: 200ms ease-decelerate
  Delay: 0ms (immediate, then reflow other elements)
  
Existing elements:
  Reposition: FLIP animation to new position
  Duration: 300ms ease-decelerate
  
Gap closing:
  After removal, remaining elements slide together
  Duration: 200ms ease-decelerate
```

### Removing elements

```
Removed element:
  Exit: scale(1) + opacity(1) → scale(0.8) + opacity(0)
  Duration: 150ms ease-accelerate
  
After removal:
  Remaining elements FLIP to close the gap
  Duration: 250ms ease-decelerate
  
If the removed element is mid-drag:
  Animate to its "home" position, then fade out
  Spring animation to home, then 150ms fade
```

### Reordering (drag-and-drop)

```
Dragging element:
  On pickup: scale(1.02), shadow increases, opacity stays 1
  During drag: follow pointer with spring (stiffness: 400, damping: 30)
  
Other elements:
  When dragged element crosses a threshold, reorder the list
  Each displaced element: FLIP animation to new position, 250ms ease-decelerate
  
On drop:
  If valid: spring to final position (stiffness: 300, damping: 30)
  If invalid: spring back to original position (stiffness: 200, damping: 25)
  All displaced elements: FLIP to final positions, 200ms
```

### Filtering and sorting

```
When a filter is applied:
  Matching items: fade + scale from 0.9 to 1 (150ms ease-decelerate)
  Non-matching items: fade out + scale to 0.9 (150ms ease-accelerate)
  Remaining items: FLIP to new positions (250ms ease-decelerate)
  
When sort order changes:
  All items: FLIP to new positions (300ms ease-decelerate)
  Stagger: none — all move simultaneously
  Opacity: stays 1 throughout (items don't fade, they just slide)
  
When a filter is removed:
  Previously hidden items: fade in + scale from 0.9 (150ms)
  All items: FLIP to include returned items (250ms)
```

## Scroll-driven animation

### Reveal on scroll

Elements enter as they scroll into the viewport:

```
Entry animation: opacity 0→1 + translateY(24px)→0
Trigger: element is 10% inside the viewport from bottom
Duration: 400ms ease-decelerate
Stagger: 60ms between sibling elements

Implementation with IntersectionObserver:
  Threshold: 0.1 (10% visible)
  Once: true (only animate on first appearance)
```

Rules:
- Don't animate everything on the page — only key content sections
- Hero section: always visible (no scroll animation) — it's the first thing users see
- Use for: feature sections, testimonials, stats, gallery items
- Never reveal critical content only on scroll — users might not scroll

### Parallax

Background elements move at a different rate than foreground:

```
Multi-speed parallax:
  Background: 0.3× scroll speed (moves slower)
  Midground:  0.6× scroll speed
  Foreground: 1.0× scroll speed (normal)

CSS:
  .parallax-bg {
    transform: translateY(calc(var(--scroll-progress) * -0.3));
    will-change: transform;
  }
```

Rules:
- Maximum parallax offset: 100px — more causes disorientation
- Only use on decorative, non-essential elements
- Never parallax text that must be read — it causes reading difficulty
- Use sparingly — subtle parallax (0.1× to 0.3× ratio) is elegant, heavy parallax is gimmicky
- Disable on `prefers-reduced-motion`

### Scroll-linked progress

Animation progress is directly tied to scroll position:

```
Reading progress bar:
  Width: 0% → 100% mapped to scroll(0% → 100%)
  
Step indicator:
  Current step advances as user scrolls through onboarding
  
Image sequence:
  Frame progression mapped to scroll (like Apple product pages)

Implementation:
  Use scroll-progress custom property or scroll-driven animations (CSS)
```

Rules:
- Scroll-linked animations must feel direct — no latency between scroll and response
- Use `will-change: transform` on the animated element
- Calculate scroll progress in `requestAnimationFrame`, never in scroll event
- Provide a non-animated fallback for reduced-motion preference

## Performance optimization

### Golden rule
Only animate `transform` and `opacity`. Everything else causes layout or paint.

| Property | Animation cost | Allowed? |
|----------|---------------|----------|
| transform | Composite layer only — 60fps | Yes |
| opacity | Composite layer only — 60fps | Yes |
| filter | Composite layer (blur is expensive) | Cautiously |
| background-color | Paint — can be 60fps | Yes, simple |
| box-shadow | Paint — expensive | Avoid, use pseudo-element with opacity |
| width/height | Layout — causes reflow | Never |
| margin/padding | Layout — causes reflow | Never |
| top/left/right/bottom | Layout if position: absolute, else reflow | Never |
| border-width | Layout — causes reflow | Never |

### Composite layer management

```css
/* Create a layer only for elements being animated */
.animating {
  will-change: transform, opacity;
}

/* REMOVE will-change after animation completes */
.animating-done {
  will-change: auto;
}
```

Rules:
- `will-change` is not a performance hint to sprinkle everywhere — it's a declaration that you WILL change these properties
- Each `will-change` creates a new compositing layer — memory overhead
- Maximum 3-4 animated layers simultaneously on mobile
- Remove `will-change` after the animation ends
- For persistent animations (scroll-linked): keep the layer, but test memory on mobile

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  /* Option 1: Disable all animation */
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }

  /* Option 2: Reduce to opacity-only (preferred — maintains state change visibility) */
  .reveal { animation: fadeIn 0.01ms; }
  .stagger { animation: fadeIn 0.01ms; }
  .slide-in { animation: fadeIn 0.01ms; }
}
```

Never remove all visual feedback for state changes — users who prefer reduced motion still need to see that something happened. Use instant opacity transitions instead.

## Quality checklist

- [ ] Every page transition uses directional slides (forward/back) or cross-fade
- [ ] Shared elements morph between contexts (card → detail)
- [ ] Staggered entrances total <600ms
- [ ] Exits are faster than entries (150ms vs 300ms)
- [ ] Layout changes animate using FLIP technique (no jarring repositioning)
- [ ] Spring physics used for interactive/gesture animations
- [ ] All animations are interruptible (no "animation lock")
- [ ] Only transform and opacity are animated (no layout properties)
- [ ] will-change is applied only during animations, removed after
- [ ] prefers-reduced-motion reduces to opacity-only, never removes all feedback
- [ ] Scroll animations trigger on IntersectionObserver, not scroll events
- [ ] Parallax offsets are subtle (max 100px)
- [ ] Total page reveal timeline ≤800ms
- [ ] Animation state machine defined for multi-state components
- [ ] No animation jank on mobile (test on mid-range device)

## Anti-patterns I avoid

- Animating width, height, margin, padding — causes layout reflow
- Using CSS transitions for scroll-linked animations — can't be interruptible
- Staggering exits — exits should be simultaneous and fast
- Total page animation >800ms — it feels slow and blocks the user
- Adding will-change to everything — memory overhead, browser creates too many layers
- Blank flash between pages — shared elements or cross-fade prevent this
- Non-interruptible animations — if the user clicks during a 300ms transition, respond immediately
- Animating with JavaScript `scroll` event — use IntersectionObserver or scroll-driven CSS
- Combining spring and timed animations on the same element — pick one paradigm
- Forgetting the reduced-motion fallback — accessibility violation
- Starting all elements at the same time (no stagger) — feels mechanical
- Using setTimeout for animation sequencing — use animation-delay or animation events