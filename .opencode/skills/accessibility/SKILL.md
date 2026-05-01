---
name: accessibility
description: Audit and fix accessibility issues: WCAG compliance, keyboard navigation, screen readers, and ARIA
---

## What I do

I audit and fix accessibility issues in web applications:

- Check WCAG 2.1 AA compliance
- Fix keyboard navigation and focus management
- Add proper ARIA attributes and semantic HTML
- Ensure screen reader compatibility
- Test color contrast and text legibility
- Implement accessible dynamic content (modals, menus, toasts)
- Automate accessibility testing

## When to use me

Use this skill when:
- Auditing a page or component for accessibility issues
- Building new UI components that must be accessible
- Fixing keyboard navigation or focus issues
- Adding ARIA labels and roles
- Ensuring form inputs have proper labels and error associations
- Making dynamic content (modals, menus, toasts) accessible
- Setting up automated accessibility testing

## How I work

1. **Audit the current state** — Review the HTML structure, ARIA usage, and keyboard behavior. Check for common violations.
2. **Check against WCAG principles**:
   - **Perceivable**: Text alternatives, captions, contrast, adaptable content
   - **Operable**: Keyboard accessible, enough time, seizure safety, navigable
   - **Understandable**: Readable, predictable, input assistance
   - **Robust**: Compatible with assistive technologies
3. **Fix by priority** — Critical (blocks access) > High (major difficulty) > Medium (frustrating but usable) > Low (nice to have)
4. **Verify** — Test with keyboard only. Test with a screen reader if possible. Re-check contrast and ARIA validity.

## Common fixes

| Issue | Fix |
|---|---|
| Missing alt text | Add descriptive `alt` to images, `alt=""` for decorative |
| No heading structure | Use `h1-h6` in logical order, no level skipping |
| Non-semantic divs | Use `<button>`, `<nav>`, `<main>`, `<article>` etc. |
| Missing form labels | Add `<label>` with `for`, or `aria-label`/`aria-labelledby` |
| No focus indicator | Add `:focus-visible` styles, never `outline: none` |
| Modal focus trap | Focus first interactive element on open, trap Tab, restore on close |
| Dynamic content updates | Use `aria-live` regions for toasts, errors, status |
| Color-only information | Add text/icon indicators alongside color |
| Insufficient contrast | Ensure 4.5:1 for normal text, 3:1 for large text |
| Skip navigation | Add skip link as first focusable element |

## ARIA patterns

### Accessible modal/dialog

```tsx
import { useRef, useEffect } from 'react';

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const dialogRef = useRef<HTMLDivElement>(null);
  const previousFocus = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement as HTMLElement;
      const firstFocusable = dialogRef.current?.querySelector(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      ) as HTMLElement;
      firstFocusable?.focus();
    } else {
      previousFocus.current?.focus();
    }
  }, [isOpen]);

  useEffect(() => {
    function handleKeyDown(e: KeyboardEvent) {
      if (e.key === 'Escape') onClose();
      if (e.key === 'Tab' && dialogRef.current) {
        const focusable = dialogRef.current.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        const first = focusable[0] as HTMLElement;
        const last = focusable[focusable.length - 1] as HTMLElement;

        if (e.shiftKey && document.activeElement === first) {
          e.preventDefault();
          last.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      }
    }

    if (isOpen) {
      document.addEventListener('keydown', handleKeyDown);
      document.body.style.overflow = 'hidden';
    }

    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      document.body.style.overflow = '';
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      ref={dialogRef}
      onClick={(e) => e.target === e.currentTarget && onClose()}
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose} aria-label="Close dialog">Close</button>
    </div>
  );
}
```

### Accessible tabs

```tsx
function Tabs({ tabs }: { tabs: { id: string; label: string; content: ReactNode }[] }) {
  const [activeTab, setActiveTab] = useState(tabs[0].id);

  return (
    <div>
      <div role="tablist" aria-label="Settings tabs">
        {tabs.map(tab => (
          <button
            key={tab.id}
            role="tab"
            aria-selected={activeTab === tab.id}
            aria-controls={`panel-${tab.id}`}
            id={`tab-${tab.id}`}
            tabIndex={activeTab === tab.id ? 0 : -1}
            onClick={() => setActiveTab(tab.id)}
            onKeyDown={(e) => {
              if (e.key === 'ArrowRight') {
                const next = tabs[(tabs.findIndex(t => t.id === activeTab) + 1) % tabs.length];
                setActiveTab(next.id);
              }
              if (e.key === 'ArrowLeft') {
                const prev = tabs[(tabs.findIndex(t => t.id === activeTab) - 1 + tabs.length) % tabs.length];
                setActiveTab(prev.id);
              }
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>
      {tabs.map(tab => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== tab.id}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

### Accessible form with errors

```tsx
function AccessibleForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-required="true"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <span id="email-error" role="alert" className="text-red-600">
            {errors.email}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          aria-required="true"
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <span id="password-error" role="alert" className="text-red-600">
            {errors.password}
          </span>
        )}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

### ARIA live regions

```tsx
function ToastContainer() {
  const [toasts, setToasts] = useState<Toast[]>([]);

  return (
    <div
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    >
      {toasts.length > 0 && toasts[toasts.length - 1].message}
    </div>
  );
}

// For important announcements:
<div aria-live="assertive" aria-atomic="true">
  {errorMessage}
</div>
```

## Automated testing

### axe-core integration

```ts
import { run } from 'axe-core';

// In tests
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('component has no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on: [push]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm install && npm run build
      - run: npm install -g @lhci/cli
      - run: lhci autorun
```

### Playwright accessibility tests

```ts
import { test, expect } from '@playwright/test';
import { injectAxe, checkA11y } from 'axe-playwright';

test('homepage accessibility', async ({ page }) => {
  await page.goto('/');
  await injectAxe(page);
  await checkA11y(page, null, {
    axeOptions: {
      runOnly: ['wcag2a', 'wcag2aa'],
    },
  });
});
```

## Keyboard navigation testing

```
Manual keyboard test checklist:
- [ ] Tab moves focus forward through interactive elements
- [ ] Shift+Tab moves focus backward
- [ ] Enter activates buttons and links
- [ ] Space toggles checkboxes and buttons
- [ ] Arrow keys navigate within widgets (tabs, menus, radios)
- [ ] Escape closes modals and menus
- [ ] Focus is visible at all times (focus indicators)
- [ ] Focus trap works in modals
- [ ] Focus returns to trigger element when modal/menu closes
```

## Color contrast

```css
/* Ensure these ratios pass WCAG AA:
   Normal text: 4.5:1
   Large text (18px+ or 14px+ bold): 3:1
   UI components: 3:1
*/

/* Use online tools or browser devtools to check:
   - Chrome DevTools > Elements > Accessibility pane
   - axe DevTools extension
   - WebAIM Contrast Checker
*/
```

## Guidelines

- Prefer semantic HTML over ARIA — first law of ARIA: don't use ARIA
- All interactive elements must be keyboard reachable and operable
- Every form input needs an associated label
- Focus must always be visible — never remove focus indicators
- Modals must trap focus and restore it on close
- Test with Tab, Shift+Tab, Enter, Space, Escape as minimum keyboard testing
- Don't use `tabindex` above 0 — use 0 for custom interactive elements, -1 for programmatic focus
- Use `aria-live` for dynamic content updates
- Provide text alternatives for all non-text content
- Ensure sufficient color contrast (4.5:1 for normal text)

## Anti-patterns I avoid

- Using `div` or `span` for buttons instead of `<button>`
- Removing focus indicators with `outline: none`
- Using `tabindex` > 0
- Missing `alt` attributes on images
- Using color alone to convey information
- Not trapping focus in modals
- Missing form labels or using placeholders as labels
- Using `aria-hidden` on focusable elements
- Auto-playing audio/video without user control
- Setting font sizes in pixels that prevent user zoom
- Using tables for layout instead of data
- Missing skip navigation links
- Using `title` attributes as the only label method