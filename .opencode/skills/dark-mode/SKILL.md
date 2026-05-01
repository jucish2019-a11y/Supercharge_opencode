---
name: dark-mode
description: Design beautiful dual-theme UIs with proper contrast, elevation, and semantic color mapping
---

## What I do

I design and implement dual-theme (light/dark) UIs that look beautiful in both modes:

- Create semantic color systems that map to both themes
- Handle elevation and surface hierarchy in dark mode
- Ensure proper contrast ratios in both themes
- Implement smooth theme switching

## When to use me

Use this skill when:
- Adding dark mode to an existing light-only UI
- Building a new app that must support both themes
- Fixing contrast or readability issues in dark mode
- Implementing theme toggle and persistence

## How I work

1. **Map semantic colors** — Define every color as a semantic token with light and dark values:

   | Token | Light | Dark |
   |---|---|---|
   | `--bg-primary` | #FFFFFF | #0A0A0A |
   | `--bg-secondary` | #F5F5F5 | #141414 |
   | `--bg-tertiary` | #EBEBEB | #1E1E1E |
   | `--text-primary` | #171717 | #F5F5F5 |
   | `--text-secondary` | #525252 | #A3A3A3 |
   | `--text-tertiary` | #A3A3A3 | #666666 |
   | `--border` | rgba(0,0,0,0.06) | rgba(255,255,255,0.06) |
   | `--border-strong` | rgba(0,0,0,0.15) | rgba(255,255,255,0.15) |

2. **Handle elevation in dark mode** — In dark mode, elevation is expressed through lighter surfaces, not darker shadows:
   - Level 0 (base): `#0A0A0A`
   - Level 1 (card): `#141414`
   - Level 2 (raised card): `#1A1A1A`
   - Level 3 (modal/popover): `#222222`
   - Level 4 (dialog/alert): `#2A2A2A`

   Shadow in dark mode should be very subtle: `0 2px 8px rgba(0,0,0,0.4)`.

3. **Desaturate accent colors** — Bright, saturated colors cause eye strain on dark backgrounds. For dark mode:
   - Reduce saturation of primary colors by 10-20%
   - Use slightly softer variants of brand colors
   - Increase lightness of text on dark backgrounds less than you'd think — the contrast is already there

4. **Adjust images and media** — Reduce brightness and increase warmth on images in dark mode using CSS: `filter: brightness(0.9)`. Consider dimming or adding a slight overlay.

5. **Implement the toggle** — Use `data-theme` attribute on `<html>`, persist to `localStorage`, respect `prefers-color-scheme`.

## Tailwind dark mode configuration

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class', '[data-theme="dark"]'], // or 'media' for OS preference only
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
      },
    },
  },
};

export default config;
```

## Theme switching logic

### React context with next-themes

```tsx
// providers/theme-provider.tsx
'use client';

import { ThemeProvider as NextThemesProvider } from 'next-themes';
import { type ReactNode } from 'react';

export function ThemeProvider({ children }: { children: ReactNode }) {
  return (
    <NextThemesProvider
      attribute="data-theme"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange={false}
    >
      {children}
    </NextThemesProvider>
  );
}

// Theme toggle component
'use client';

import { useTheme } from 'next-themes';
import { useEffect, useState } from 'react';

export function ThemeToggle() {
  const { theme, setTheme, resolvedTheme } = useTheme();
  const [mounted, setMounted] = useState(false);

  // Prevent hydration mismatch
  useEffect(() => setMounted(true), []);
  if (!mounted) return <div className="w-9 h-9" />;

  return (
    <button
      onClick={() => setTheme(resolvedTheme === 'dark' ? 'light' : 'dark')}
      className="p-2 rounded-md hover:bg-accent"
      aria-label="Toggle theme"
    >
      {resolvedTheme === 'dark' ? <SunIcon /> : <MoonIcon />}
    </button>
  );
}
```

### Manual theme implementation (without next-themes)

```tsx
// hooks/use-theme.ts
import { useState, useEffect, useCallback } from 'react';

type Theme = 'light' | 'dark' | 'system';

function getSystemTheme(): 'light' | 'dark' {
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

function getResolvedTheme(theme: Theme): 'light' | 'dark' {
  if (theme === 'system') return getSystemTheme();
  return theme;
}

export function useTheme() {
  const [theme, setThemeState] = useState<Theme>(() => {
    return (localStorage.getItem('theme') as Theme) ?? 'system';
  });

  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>(
    getResolvedTheme(theme)
  );

  useEffect(() => {
    const root = document.documentElement;
    const resolved = getResolvedTheme(theme);

    root.setAttribute('data-theme', resolved);
    root.classList.remove('light', 'dark');
    root.classList.add(resolved);

    setResolvedTheme(resolved);
  }, [theme]);

  useEffect(() => {
    const media = window.matchMedia('(prefers-color-scheme: dark)');
    const handler = () => {
      if (theme === 'system') {
        setResolvedTheme(getSystemTheme());
      }
    };

    media.addEventListener('change', handler);
    return () => media.removeEventListener('change', handler);
  }, [theme]);

  const setTheme = useCallback((newTheme: Theme) => {
    localStorage.setItem('theme', newTheme);
    setThemeState(newTheme);
  }, []);

  return { theme, setTheme, resolvedTheme };
}
```

## FOUC prevention

```html
<!-- Add to <head> to prevent flash of unstyled content -->
<script>
  (function() {
    const theme = localStorage.getItem('theme') ?? 'system';
    const resolved = theme === 'system'
      ? (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
      : theme;
    document.documentElement.setAttribute('data-theme', resolved);
    document.documentElement.classList.add(resolved);
  })();
</script>
```

## Chart and table adaptations

```tsx
// Chart colors for dark mode
const chartColors = {
  light: {
    grid: '#e5e5e5',
    text: '#171717',
    series: ['#2563eb', '#16a34a', '#dc2626', '#ca8a04'],
  },
  dark: {
    grid: '#262626',
    text: '#e5e5e5',
    series: ['#3b82f6', '#22c55e', '#ef4444', '#eab308'],
  },
};

// Table adaptations
const tableStyles = {
  light: 'bg-white border-gray-200',
  dark: 'bg-neutral-900 border-neutral-800',
};
```

## CSS implementation

```css
:root, [data-theme="light"] {
  --background: 0 0% 100%;
  --foreground: 0 0% 9%;
  --card: 0 0% 100%;
  --card-foreground: 0 0% 9%;
  --popover: 0 0% 100%;
  --popover-foreground: 0 0% 9%;
  --primary: 221 83% 53%;
  --primary-foreground: 0 0% 100%;
  --secondary: 210 40% 96%;
  --secondary-foreground: 222 47% 11%;
  --muted: 210 40% 96%;
  --muted-foreground: 215 16% 47%;
  --accent: 210 40% 96%;
  --accent-foreground: 222 47% 11%;
  --destructive: 0 84% 60%;
  --destructive-foreground: 0 0% 100%;
  --border: 214 32% 91%;
  --input: 214 32% 91%;
  --ring: 221 83% 53%;
}

[data-theme="dark"] {
  --background: 0 0% 4%;
  --foreground: 0 0% 96%;
  --card: 0 0% 4%;
  --card-foreground: 0 0% 96%;
  --popover: 0 0% 4%;
  --popover-foreground: 0 0% 96%;
  --primary: 217 91% 60%;
  --primary-foreground: 0 0% 100%;
  --secondary: 217 19% 27%;
  --secondary-foreground: 0 0% 96%;
  --muted: 217 19% 27%;
  --muted-foreground: 215 20% 65%;
  --accent: 217 19% 27%;
  --accent-foreground: 0 0% 96%;
  --destructive: 0 62% 30%;
  --destructive-foreground: 0 0% 96%;
  --border: 217 19% 27%;
  --input: 217 19% 27%;
  --ring: 224 76% 48%;
}

/* Smooth transitions */
* {
  transition: background-color 0.2s ease, border-color 0.2s ease;
}

/* Respect reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    transition: none !important;
  }
}
```

## Quality checklist

- [ ] Semantic color tokens defined for all UI elements
- [ ] Dark mode values tested for contrast (4.5:1 minimum)
- [ ] Elevation hierarchy works in dark mode (lighter = higher)
- [ ] Accent colors desaturated for dark backgrounds
- [ ] Images and media adapted for dark mode
- [ ] Theme toggle persists to localStorage
- [ ] System preference respected on first visit
- [ ] FOUC prevented with inline script
- [ ] Charts and data visualizations have dark variants
- [ ] Reduced motion respected for theme transitions

## Anti-patterns I avoid

- Never use absolute black (#000000) for backgrounds — use near-black (#0A0A0A to #141414)
- Never use absolute white (#FFFFFF) for text on dark — use off-white (#F5F5F5)
- Semantic tokens only in component CSS — never raw colors
- Test contrast ratios: 4.5:1 for normal text, 3:1 for large text (WCAG AA)
- Adjust images and borders for dark mode — they shouldn't look washed out
- Respect the user's system preference on first visit, then persist their manual choice
- Avoid large areas of pure saturated color in dark mode