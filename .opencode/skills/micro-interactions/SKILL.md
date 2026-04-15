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

## How I work

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

## Interaction patterns

### Buttons
```css
.button {
  transition: background-color 150ms ease, transform 100ms ease, box-shadow 150ms ease;
}
.button:hover {
  background-color: var(--button-bg-hover);
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
.button:active {
  transform: scale(0.97);
}
.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

### Cards
```css
.card {
  transition: box-shadow 200ms ease, transform 200ms ease;
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
- **Skeleton**: Pulse animation on placeholder shapes (preferred for initial load)
- **Spinner**: For short waits (<1s), small inline spinner
- **Progress bar**: For determinate operations with known duration
- **Spinner + text**: For waits >2s, show progress message

### Error/Success feedback
- **Error**: Shake horizontally (translateX oscillation, 300ms)
- **Success**: Scale up slightly then settle (scale 1 → 1.05 → 1, 200ms)

## Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Guidelines

- Animations should be fast: 150-300ms for most states
- Only animate `transform` and `opacity` for performance (avoid animating width, height, margin, padding)
- Every interactive element needs hover, focus, and active states
- Skeleton screens > spinners for content loading
- Never animate auto-play — motion should be user-triggered or state-triggered
- Test on low-end devices — disable heavy animations if they cause jank
- Always provide a `prefers-reduced-motion` fallback