---
name: premium-motion
description: Physics-based animation, scroll-linked transitions, container transforms, stagger cascades, and interruptible gesture tracking that make interfaces feel alive rather than animated
---

## What I do

I define the motion layer that separates premium interfaces from generic ones. Most sites use CSS transitions with `ease-in-out` and call it done. Stripe, Linear, and Apple use spring physics, scroll-linked animation, container transforms, and interruptible transitions.

This skill covers:
- Spring-based animation (mass, tension, friction, damping) instead of easing curves
- Stagger cascade patterns with calculated delays
- Scroll-linked animation (progress tied to scroll position, not just triggered by it)
- Container transform / shared element transitions (morphing elements between states)
- Interruptible gesture-tracked transitions
- Frequency-based animation investment (animate what matters, not everything)

## When to use me

Use this skill when:
- An interface feels mechanically animated instead of physically responsive
- Transitions use generic easing (`ease-in-out`, `cubic-bezier(0.4, 0, 0.2, 1)`) instead of spring physics
- All elements animate simultaneously instead of cascading
- Animations can't be interrupted or reversed mid-flight
- Scroll animations trigger once instead of linking to scroll progress
- Every interaction has the same animation weight regardless of importance

For related but distinct work:
- **motion-system** — Duration scales, easing library, choreography patterns (the vocabulary)
- **animation-choreography** — Specific animation techniques: FLIP, shared element, scroll-driven (the recipes)
- **micro-interactions** — State models for interactive elements (hover, focus, active, etc.)
- **interaction-states** — Complete state machine specs with transition timing (the state chart)

This skill provides the physics engine and linking strategy; motion-system provides the timing scale; animation-choreography provides the animation techniques; interaction-states defines when each transition fires.

## How I work

### Checker mode (auditing existing motion design)

1. **Spring audit** — Are animations using CSS easing or spring physics? Replace generic curves with springs where elements need natural deceleration.
2. **Stagger audit** — Do groups of elements animate simultaneously or in sequence? Simultaneous = amateur. Cascading = premium.
3. **Scroll audit** — Are scroll animations triggered (fire once) or linked (progress tied to position)? Triggered = acceptable. Linked = premium.
4. **Interruption audit** — Can transitions be reversed mid-flight? If not, the interface feels mechanical.
5. **Investment audit** — Is every interaction animated equally? The frequency-novelty matrix should determine animation weight.
6. **Consistency audit** — Do similar interactions use similar timing? Button presses should feel the same everywhere.

### Applier mode (building a premium motion system)

#### Step 1: Choose your spring physics system

CSS transitions use easing curves. Premium interfaces use spring physics. Springs simulate real-world motion with configurable mass, tension, and friction.

**Spring presets** (use as starting points, customize per project):

| Preset | Mass | Tension | Friction | Feel | Use for |
|--------|------|---------|----------|------|---------|
| Gentle | 1 | 120 | 14 | Soft, floating | Modals, overlays, large content transitions |
| Snappy | 1 | 170 | 26 | Quick, precise | Buttons, toggles, selection changes |
| Bouncy | 1 | 180 | 12 | Playful, overshoots | Notifications, success states, playful UIs |
| Stiff | 1 | 210 | 36 | Rigid, immediate | Drag reordering, layout shifts, density changes |
| Mushy | 1 | 100 | 24 | Slow, dreamy | Onboarding reveals, ambient background motion |

**CSS approximation** of spring presets (when spring libraries aren't available):

```
Gentle:  cubic-bezier(0.22, 1, 0.36, 1)  500ms
Snappy:  cubic-bezier(0.32, 0.72, 0, 1)   250ms
Bouncy:  cubic-bezier(0.34, 1.56, 0.64, 1) 350ms
Stiff:   cubic-bezier(0.25, 0.1, 0.25, 1)  150ms
```

These are approximations. Real springs overshoot and settle, which CSS cubic-bezier can't fully replicate. If your project uses React, Framer Motion provides `spring()` configs. On native, use platform spring APIs (UISpringAnimation on iOS, spring-animation on Android).

**When to use springs vs. curves**:

| Context | Use springs | Use curves |
|---------|------------|------------|
| Elements that decelerate naturally | Yes | No |
| Elements that need overshoot | Yes | No |
| Elements that need interruptibility | Yes | No |
| Simple opacity fades | No | Yes (simpler) |
| Background color changes | No | Yes (no overshoot needed) |
| Scroll-position-linked animation | No | Yes (linear mapping) |

#### Step 2: Define stagger cascade patterns

When multiple elements enter a view, they should not appear simultaneously. Premium interfaces use calculated stagger delays:

**Stagger formula**: `delay = base + (index * gap)`

| Pattern | Gap | Feel | Example |
|---------|-----|------|---------|
| List cascade | 30-50ms | Fast, efficient | Email inbox items appearing |
| Card grid | 40-80ms | Organized, purposeful | Feature cards in a grid |
| Menu items | 20-30ms | Quick, responsive | Dropdown menu items |
| Dashboard widgets | 60-80ms | Considered, weighted | Chart and stat panels |
| Stacked notifications | 80-120ms | Playful, noticeable | Toast stack appearing |

**Implementation** (CSS):

```css
.stagger-item {
  opacity: 0;
  transform: translateY(12px);
  animation: stagger-in 300ms ease-out forwards;
  animation-delay: calc(var(--stagger-index) * 50ms);
}

@keyframes stagger-in {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

**Implementation** (Framer Motion):

```jsx
<motion.div
  initial={{ opacity: 0, y: 12 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{
    delay: index * 0.05,
    type: "spring",
    stiffness: 170,
    damping: 26
  }}
/>
```

**Stagger principles**:
- Stagger gap should be shorter than individual animation duration (typically 15-25% of duration)
- Maximum total stagger should not exceed 600ms (after that, the pattern breaks and late items feel broken)
- First item in a stagger should have 0 delay (it anchors perception)
- Use `prefers-reduced-motion` to collapse staggers into simultaneous appearance

#### Step 3: Implement scroll-linked animation

Most interfaces use scroll-triggered animation (Intersection Observer fires once, animation plays). Premium interfaces use scroll-linked animation (animation progress is tied to scroll position).

**Scroll-triggered** (acceptable for entrance animations):
```js
// Animation fires once when element enters viewport
observer = new IntersectionObserver(callback);
// Then: element.classList.add('animate-in');
// Element plays animation and stays in final state
```

**Scroll-linked** (premium, for progressive reveal):
```js
// Animation progress = scroll progress
const scrollProgress = (scrollY - elementTop) / elementHeight;
element.style.opacity = scrollProgress;
element.style.transform = `translateY(${(1 - scrollProgress) * 20}px)`;
```

**Scroll-linked decision matrix**:

| Content type | Trigger or linked? | Pattern |
|-------------|-------------------|---------|
| Section entrance | Triggered | Fade/slide in once |
| Parallax background | Linked | Y-position mapped to scroll |
| Progress indicators | Linked | Width/height mapped to scroll |
| Image reveal on scroll | Linked | Clip-path or opacity mapped to scroll |
| Sticky element transition | Linked | Transform mapped to scroll within sticky range |
| 3D perspective shift | Linked | RotateX/Y mapped to scroll |
| Counter animation | Triggered | Count-up animation fires once |

**Performance rules for scroll-linked**:
- Only animate `transform` and `opacity` (compositor-only properties)
- Use `requestAnimationFrame` for scroll handlers, never direct scroll event
- Throttle to 60fps maximum (skip frames if scroll fires faster)
- Always provide `prefers-reduced-motion` fallback: skip to final state

#### Step 4: Define container transform patterns

Container transforms morph an element from one state to another while maintaining visual continuity. This is the single most impactful transition technique for spatial awareness.

**Pattern 1: Card to detail (hero expand)**

The card in a grid morphs into the full detail view. The card's position, size, border-radius, and background transition simultaneously. All other content fades/slides out.

```
Entry: card scales from grid position to full-width
       border-radius transitions from card-radius (12px) to detail-radius (0)
       surrounding content fades out (150ms)
       detail content fades in (200ms, 50ms delay)

Exit:  reverse — detail shrinks back to card position
       content fades back in
```

**Pattern 2: List item to detail (slide reveal)**

A list item expands to reveal detail content below it. Other items slide down to make room.

```
Entry: clicked item height expands (300ms spring)
       detail content fades in within expanded area (200ms, 100ms delay)
       other items translate down (250ms, stagger offset per item)

Exit:  detail content fades out (100ms)
       item height collapses (250ms spring)
       other items translate back up (200ms)
```

**Pattern 3: Shared axis transition (tab switch)**

Content exits along an axis (left/right, up/down, forward/back) and new content enters along the same axis.

```
Exit:  outgoing content slides left + fades out (200ms)
Enter: incoming content slides in from right + fades in (200ms, 50ms delay)
       tabs slide indicator to new position (250ms spring)
```

**Pattern 4: Fade through (unrelated content)**

For switching between unrelated content (e.g., navigation sections). Not a crossfade — outgoing fades first, then incoming fades in.

```
Exit:  outgoing content opacity 1 → 0.85 → 0 (150ms)
Gap:  brief moment (50ms) where neither is visible
Enter: incoming content opacity 0 → 1 (200ms)
```

**Pattern 5: Morphing shell (layout stays, content transitions)**

The page shell (nav, sidebar, header) stays fixed. Only the content area transitions. This maintains spatial orientation.

```
Shell: nav, sidebar, header remain fixed (0ms transition)
Content: outgoing fades/slides out (150ms)
         incoming fades/slides in (200ms, 50ms delay)
```

#### Step 5: Define interruptible transitions

Premium interfaces allow transitions to be interrupted mid-flight. If a user changes direction during a transition, the animation reverses from its current position rather than completing first.

**Interruptibility rules by interaction type**:

| Interaction | Interruptible? | Behavior |
|-------------|---------------|----------|
| Hover in/out | Yes | Reverse immediately from current position |
| Modal open/close | Yes | If closing during open animation, shrink from current position |
| Tab switch | No (complete current) | Finish current transition, then start new one |
| Drag gesture | Yes (gesture-tracked) | Animation follows finger/mouse position in real-time |
| Page navigation | Context-dependent | If new nav event within 300ms, cancel and start new |
| Scroll-linked | Always | Animation is tied to scroll position, inherently interruptible |

**Implementation approach**:

For CSS transitions: Use `transition: all 200ms ease-out` and toggle classes. CSS transitions are naturally interruptible — if you remove the class mid-transition, it reverses from the current position.

For JS animations: Never use `setTimeout`-based sequencing. Use animation libraries (Framer Motion, GSAP, Web Animations API) that support cancelling and reversing from current position.

For gesture-driven animations: Use `requestAnimationFrame` with pointer position tracking. Each frame, the animated property converges toward the pointer position using spring physics, not a fixed target.

```js
// Gesture-tracked spring animation
let current = 0;
let velocity = 0;
const tension = 170;
const friction = 26;

function animateSpring(target) {
  const force = (target - current) * (tension / 1000);
  velocity += force;
  velocity *= (1 - friction / 100);
  current += velocity;

  element.style.transform = `translateX(${current}px)`;

  if (Math.abs(velocity) > 0.1 || Math.abs(target - current) > 0.1) {
    requestAnimationFrame(() => animateSpring(target));
  }
}
```

#### Step 6: Apply the frequency-novelty matrix

(See **interaction-states** skill for the detailed matrix. Summary:)

| | Low novelty | High novelty |
|---|---|---|
| **High frequency** | Instant (0-50ms) | Quick with character (100-200ms) |
| **Low frequency** | Standard (200-400ms) | Invested (400-800ms) |

**Investment examples**:

| Interaction | Frequency | Novelty | Investment | Timing |
|-------------|-----------|---------|------------|--------|
| Button hover | High | Low | Instant feel | 80ms ease-out |
| Button press | High | Low | Instant feel | 60ms ease-out |
| Context menu | High | Low | Instant appear | 0ms enter, 100ms exit |
| Command palette | High | Medium | Quick with character | 120ms spring |
| Tab switch | Medium | Low | Standard | 200ms ease-in-out |
| Modal open | Medium | Medium | Standard with character | 250ms spring |
| Page transition | Low | High | Invested | 400ms stagger + spring |
| Onboarding step | Low | High | Invested | 500ms spring |
| First-time reveal | Very low | Very high | Full choreography | 600ms+ stagger cascade |
| Success confirmation | Low | Medium | Quick with character | 300ms spring |

#### Step 7: Define reduced motion alternatives

Every animation must have a reduced-motion fallback. Not all animations disappear — some become instant state changes.

| Animation type | Reduced motion behavior |
|---------------|------------------------|
| Hover/active states | Instant color change (0ms) |
| Stagger cascades | All items appear simultaneously |
| Scroll-linked animation | Elements visible in final state immediately |
| Container transforms | Instant state change (no morph) |
| Spring animations | Instant state change (0ms) |
| Page transitions | Instant content swap |
| Loading spinners | Static "Loading..." text or pulsing opacity |
| Progress indicators | Still visible but static (no animation) |

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

#### Step 8: Document the motion system

Create a motion spec for the project:

```
Project: [Name]
Motion personality: [Gentle / Snappy / Bouncy / Stiff / Custom]

Spring config:
  Standard: mass=1, tension=170, friction=26 (Snappy)
  Entrance: mass=1, tension=120, friction=14 (Gentle)
  Exit:     mass=1, tension=210, friction=36 (Stiff)

Duration scale:
  Instant:    0ms    (context menus, command palettes)
  Quick:      100ms  (hover states, focus rings)
  Standard:   200ms  (button feedback, toggles)
  Emphasized: 400ms  (modal open, page transitions)
  Scenic:     600ms+ (onboarding, first-time reveals)

Stagger patterns:
  List items:    base 0ms + 30ms per item
  Card grid:     base 0ms + 60ms per item
  Menu:          base 0ms + 20ms per item

Container transitions:
  Card → Detail: Hero expand (300ms Gentle spring)
  List → Detail: Slide reveal (250ms Snappy spring)
  Tab switch:    Shared axis X (200ms Snappy)
  Unrelated:     Fade through (150ms out + 50ms gap + 200ms in)

Scroll behavior:
  Entrances:     Scroll-triggered (IntersectionObserver)
  Parallax:      Scroll-linked (requestAnimationFrame)
  Progress:      Scroll-linked (direct mapping)

Reduced motion: All animations → instant state changes
```

## Anti-patterns I avoid

- Using `ease-in-out` for everything — it's the most generic curve and feels mechanical
- Animating all elements simultaneously — stagger cascades create perceived order and intentionality
- Animating high-frequency actions heavily — command palettes and context menus should appear instantly
- CSS transitions that can't be interrupted — if a user reverses direction, the animation must reverse from current position
- Scroll-triggered animation where scroll-linked is needed — if the user scrolls back, the animation should reverse
- Container cross-fades where container transforms are possible — cross-fades destroy spatial awareness
- The same spring configuration for every transition — entrance, exit, and persistent UI need different spring feels
- Ignoring `prefers-reduced-motion` — every animation must have an instant fallback
- Queueing animations that overlap — if a new state arrives during a transition, switch targets immediately
- Applying animation to pure color changes — background color transitions should be simple curves, not springs (no overshoot needed)