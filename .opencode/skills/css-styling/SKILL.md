---
name: css-styling
description: Implement modern CSS and styling — Tailwind CSS, CSS modules, custom properties, cascade layers, container queries, responsive design, and animation patterns
---

## What I do

I implement styling for web applications using modern CSS patterns:

- **Tailwind CSS** — Utility-first styling, configuration, custom components, responsive variants
- **CSS modules** — Scoped styles, composable class names, TypeScript integration
- **Custom properties** — Design tokens, dynamic theming, responsive tokens
- **Layout** — CSS Grid, Flexbox, container queries, responsive patterns
- **Animation** — Transitions, keyframes, reduced motion, performant animations
- **Architecture** — BEM, utility-first, CSS-in-JS tradeoffs, style organization

## When to use me

Use this skill when:
- Setting up Tailwind CSS or a styling system for a new project
- Building responsive layouts that work across devices
- Creating theming systems (light/dark mode, brand variants)
- Optimizing CSS performance (unused styles, specificity conflicts)
- Implementing complex animations and transitions
- Organizing styles in a scalable way for a growing codebase

## Styling approach decision tree

```
Need to style an application?
├── Want fast iteration, utility-first?
│   └── Tailwind CSS (with custom theme)
├── Want scoped styles, zero runtime, simple DX?
│   └── CSS Modules
├── Need runtime theming, JS integration?
│   └── CSS-in-JS (Styled Components, Emotion, Vanilla Extract)
├── Small project, simple needs?
│   └── Plain CSS or PostCSS
├── Design system, shared across projects?
│   └── Tailwind + CSS custom properties (tokens)
└── Performance-critical, zero-JS CSS?
    └── CSS Modules with custom properties
```

## Tailwind CSS

### Configuration (tailwind.config.ts)

```ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
      },
      fontFamily: {
        sans: ['var(--font-sans)', 'system-ui', 'sans-serif'],
        mono: ['var(--font-mono)', 'monospace'],
      },
      fontSize: {
        '2xs': ['0.625rem', { lineHeight: '0.875rem' }],
      },
      borderRadius: {
        '2xl': '1rem',
        '3xl': '1.5rem',
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
        '128': '32rem',
      },
      keyframes: {
        'fade-in': {
          from: { opacity: '0' },
          to: { opacity: '1' },
        },
        'slide-in': {
          from: { transform: 'translateY(10px)', opacity: '0' },
          to: { transform: 'translateY(0)', opacity: '1' },
        },
      },
      animation: {
        'fade-in': 'fade-in 0.2s ease-out',
        'slide-in': 'slide-in 0.3s ease-out',
      },
    },
  },
};

export default config;
```

### Custom components with Tailwind

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

const Button = ({ className, variant, size, ...props }: ButtonProps) => (
  <button className={cn(buttonVariants({ variant, size, className }))} {...props} />
);
```

### Tailwind utility patterns

```tsx
// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">

// Responsive visibility
<div className="block md:hidden">Mobile only</div>
<div className="hidden md:block">Desktop only</div>
<div className="hidden lg:hidden">Not on large screens</div>

// Common card pattern
<div className="rounded-lg border bg-card text-card-foreground shadow-sm p-6">

// Flex centering
<div className="flex items-center justify-center">

// Text truncation
<p className="truncate max-w-xs">

// Multi-line truncation
<p className="line-clamp-3">

// Aspect ratio
<div className="aspect-video rounded-lg overflow-hidden">
  <img className="w-full h-full object-cover" />
</div>

// Sticky positioning
<div className="sticky top-0 z-40 bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60">

// Focus ring
<button className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2">
```

## CSS custom properties (design tokens)

### Token system

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --card: 0 0% 100%;
  --card-foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --secondary: 210 40% 96.1%;
  --secondary-foreground: 222.2 47.4% 11.2%;
  --muted: 210 40% 96.1%;
  --muted-foreground: 215.4 16.3% 46.9%;
  --accent: 210 40% 96.1%;
  --accent-foreground: 222.2 47.4% 11.2%;
  --destructive: 0 84.2% 60.2%;
  --destructive-foreground: 210 40% 98%;
  --border: 214.3 31.8% 91.4%;
  --input: 214.3 31.8% 91.4%;
  --ring: 222.2 84% 4.9%;
  --radius: 0.5rem;

  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  --spacing-2xl: 3rem;

  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;

  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);

  --duration-fast: 150ms;
  --duration-normal: 250ms;
  --duration-slow: 350ms;
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  --card: 222.2 84% 4.9%;
  --card-foreground: 210 40% 98%;
  --primary: 210 40% 98%;
  --primary-foreground: 222.2 47.4% 11.2%;
  --secondary: 217.2 32.6% 17.5%;
  --secondary-foreground: 210 40% 98%;
  --muted: 217.2 32.6% 17.5%;
  --muted-foreground: 215 20.2% 65.1%;
  --accent: 217.2 32.6% 17.5%;
  --accent-foreground: 210 40% 98%;
  --destructive: 0 62.8% 30.6%;
  --destructive-foreground: 210 40% 98%;
  --border: 217.2 32.6% 17.5%;
  --input: 217.2 32.6% 17.5%;
  --ring: 212.7 26.8% 83.9%;
}
```

## CSS Modules

### Scoped component styles

```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-sm);
  padding: 0.5rem 1rem;
  border-radius: var(--radius);
  font-weight: 500;
  transition: background-color var(--duration-fast) var(--ease-default);
}

.primary {
  background-color: hsl(var(--primary));
  color: hsl(var(--primary-foreground));
}

.primary:hover {
  background-color: hsl(var(--primary) / 0.9);
}

.size-sm {
  padding: 0.25rem 0.75rem;
  font-size: var(--text-sm);
}

.size-lg {
  padding: 0.75rem 2rem;
  font-size: var(--text-lg);
}
```

```tsx
import styles from './Button.module.css';
import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ variant = 'primary', size = 'md', className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(styles.button, styles[variant], styles[`size-${size}`], className)}
      {...props}
    />
  );
}
```

## CSS Grid patterns

### Page layout

```css
.page {
  display: grid;
  grid-template-columns: 280px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    'sidebar header'
    'sidebar main'
    'sidebar footer';
  min-height: 100vh;
}

.sidebar { grid-area: sidebar; }
.header  { grid-area: header; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }

@media (max-width: 768px) {
  .page {
    grid-template-columns: 1fr;
    grid-template-areas:
      'header'
      'main'
      'footer';
  }
  .sidebar { display: none; }
}
```

### Responsive card grid

```css
.card-grid {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 320px), 1fr));
}
```

## Container queries

### Component-scoped responsive design

```css
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

## Cascade layers

```css
@layer reset, base, components, utilities;

@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

@layer base {
  body {
    font-family: var(--font-sans);
    color: hsl(var(--foreground));
    background-color: hsl(var(--background));
    line-height: 1.5;
  }
}

@layer components {
  .btn { /* ... */ }
  .card { /* ... */ }
  .input { /* ... */ }
}

@layer utilities {
  .text-balance { text-wrap: balance; }
  .text-pretty { text-wrap: pretty; }
}
```

## Animation patterns

### Performant transitions

```css
.transform-transition {
  transition: transform var(--duration-normal) var(--ease-default),
              opacity var(--duration-normal) var(--ease-default);
  will-change: transform, opacity;
}

.hover-lift:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.focus-ring:focus-visible {
  outline: 2px solid hsl(var(--ring));
  outline-offset: 2px;
}
```

### Keyframe animations

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(8px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.animate-fade-in { animation: fadeIn var(--duration-normal) var(--ease-out); }
.animate-slide-up { animation: slideUp var(--duration-normal) var(--ease-out); }
```

### Reduced motion support

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

## Responsive images

```tsx
function ResponsiveImage({ src, alt }: { src: string; alt: string }) {
  return (
    <picture>
      <source media="(min-width: 1024px)" srcSet={`${src}?w=1200`} />
      <source media="(min-width: 768px)" srcSet={`${src}?w=800`} />
      <img
        src={`${src}?w=400`}
        alt={alt}
        loading="lazy"
        decoding="async"
        className="w-full h-auto"
      />
    </picture>
  );
}
```

## Quality checklist

- [ ] Custom properties defined for all design tokens
- [ ] Dark mode theme values defined and tested for contrast
- [ ] Responsive breakpoints consistent and tested across devices
- [ ] Container queries used for component-level responsiveness
- [ ] Animations respect `prefers-reduced-motion`
- [ ] No `!important` used except in utility overrides
- [ ] Images are responsive with `srcset` or `width: 100%`
- [ ] Focus states are visible and accessible
- [ ] Font loading optimized (`font-display: swap`)
- [ ] No layout shift from unloaded fonts or images
- [ ] CSS is scoped (modules or Tailwind)

## Anti-patterns I avoid

- Using raw hex/rgb colors in components instead of semantic tokens
- Using `z-index` values above 50 without a defined stacking context
- Using `!important` to fix specificity issues
- Inline styles for things that should be design tokens
- Large CSS bundles from unused utility classes
- Animating layout properties (width, height, top, left) — use transform instead
- Not setting `font-display: swap` on custom fonts
- Using `px` for font sizes instead of `rem`
- Ignoring `prefers-reduced-motion` for animations
- Using JavaScript for layout that CSS handles better