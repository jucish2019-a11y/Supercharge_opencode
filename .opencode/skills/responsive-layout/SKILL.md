---
name: responsive-layout
description: Design and implement beautiful adaptive layouts across all screen sizes and breakpoints
---

## What I do

I design and implement responsive layouts:

- **CSS Grid and Flexbox** — Modern layout patterns for all screen sizes
- **Container queries** — Component-level responsive design
- **Responsive images** — srcset, sizes, picture element, optimization
- **Fluid typography** — Clamp-based scaling, viewport-relative sizing
- **Touch considerations** — Tap targets, hover state fallbacks, gesture support
- **Breakpoint strategy** — When and where to break, avoiding device-specific breakpoints

## When to use me

Use this skill when:
- Building layouts that work on mobile, tablet, and desktop
- Implementing responsive navigation patterns
- Optimizing images for different screen densities
- Creating fluid, scalable typography systems
- Designing component-level responsive behavior
- Fixing layout issues on specific device sizes

## Breakpoint strategy

```
Standard breakpoints (mobile-first):
├── sm: 640px   (large phones)
├── md: 768px   (tablets)
├── lg: 1024px  (small laptops)
├── xl: 1280px  (desktops)
└── 2xl: 1536px (large desktops)

Approach:
- Design mobile layout first (default styles)
- Add complexity as viewport grows (min-width media queries)
- Avoid device-specific breakpoints (iPhone, iPad)
- Use content-based breakpoints (when layout breaks, not device width)
```

## CSS Grid patterns

### Holy grail layout

```css
.layout {
  display: grid;
  grid-template:
    'header'  auto
    'main'    1fr
    'footer'  auto
    / 1fr;
  min-height: 100vh;
  gap: 1rem;
}

.layout > header { grid-area: header; }
.layout > main   { grid-area: main; }
.layout > footer { grid-area: footer; }

@media (min-width: 768px) {
  .layout {
    grid-template:
      'header header' auto
      'nav    main'   1fr
      'footer footer' auto
      / 240px  1fr;
  }
  .layout > nav { grid-area: nav; }
}

@media (min-width: 1024px) {
  .layout {
    grid-template:
      'header header header' auto
      'nav    main   aside'  1fr
      'footer footer footer' auto
      / 240px  1fr    240px;
  }
  .layout > aside { grid-area: aside; }
}
```

### Auto-fit card grid

```css
.card-grid {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 280px), 1fr));
}
```

## Container queries

```css
/* Component-level responsive design */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
    gap: 1rem;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
  .card-image {
    aspect-ratio: 16 / 9;
  }
}
```

```tsx
// Tailwind container queries (with plugin)
function CardContainer({ children }: { children: ReactNode }) {
  return <div className="@container">{children}</div>;
}

function Card() {
  return (
    <div className="@sm:grid-cols-2 @lg:grid-cols-3 flex flex-col">
      {/* Responsive within container, not viewport */}
    </div>
  );
}
```

## Responsive images

```tsx
function ResponsiveImage({ src, alt }: { src: string; alt: string }) {
  return (
    <picture>
      <source
        media="(min-width: 1024px)"
        srcSet={`${src}?w=1200 1x, ${src}?w=2400 2x`}
      />
      <source
        media="(min-width: 768px)"
        srcSet={`${src}?w=800 1x, ${src}?w=1600 2x`}
      />
      <img
        src={`${src}?w=400`}
        srcSet={`${src}?w=400 1x, ${src}?w=800 2x`}
        alt={alt}
        loading="lazy"
        decoding="async"
        className="w-full h-auto"
      />
    </picture>
  );
}
```

## Fluid typography

```css
/* Using clamp() for fluid type */
h1 {
  font-size: clamp(1.75rem, 1.25rem + 2.5vw, 3rem);
  line-height: 1.1;
}

h2 {
  font-size: clamp(1.5rem, 1.1rem + 2vw, 2.25rem);
  line-height: 1.2;
}

body {
  font-size: clamp(0.875rem, 0.8rem + 0.375vw, 1rem);
}

/* Or with Tailwind */
/* text-[clamp(1.75rem,1.25rem+2.5vw,3rem)] */
```

## Touch considerations

```css
/* Minimum tap target size */
button, a, [role="button"] {
  min-height: 44px;
  min-width: 44px;
}

/* Remove hover effects on touch devices */
@media (hover: hover) {
  .button:hover {
    background-color: var(--primary-hover);
  }
}

@media (hover: none) {
  .button:active {
    background-color: var(--primary-hover);
  }
}

/* Prevent text selection on interactive elements */
.no-select {
  user-select: none;
  -webkit-user-select: none;
}

/* Smooth scrolling */
html {
  scroll-behavior: smooth;
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}
```

## Responsive navigation

```tsx
function Navigation() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav className="bg-background border-b">
      {/* Desktop nav */}
      <div className="hidden md:flex items-center gap-6">
        <NavLinks />
      </div>

      {/* Mobile menu button */}
      <button
        className="md:hidden p-2"
        onClick={() => setIsOpen(!isOpen)}
        aria-expanded={isOpen}
        aria-label="Toggle navigation"
      >
        {isOpen ? <CloseIcon /> : <MenuIcon />}
      </button>

      {/* Mobile nav */}
      {isOpen && (
        <div className="md:hidden py-4">
          <NavLinks />
        </div>
      )}
    </nav>
  );
}
```

## Quality checklist

- [ ] Mobile-first approach (base styles for mobile, enhance for larger screens)
- [ ] Content-based breakpoints, not device-specific
- [ ] Container queries used for component-level responsiveness
- [ ] Images optimized with srcset for different screen densities
- [ ] Fluid typography scales smoothly between breakpoints
- [ ] Touch targets minimum 44x44px
- [ ] Hover states have touch fallbacks
- [ ] Layout shift prevented (CLS < 0.1)
- [ ] Tested on actual devices, not just browser resizing
- [ ] Reduced motion preferences respected

## Anti-patterns I avoid

- Device-specific breakpoints (iPhone, iPad sizes)
- Hiding content with `display: none` instead of reordering
- Fixed-width layouts that don't adapt
- Not testing on real devices
- Using pixels for font sizes instead of rem
- Horizontal scroll on mobile
- Touch targets smaller than 44x44px
- Layout shifts caused by late-loading content
- Ignoring landscape orientation on mobile
- Using `!important` to override responsive styles