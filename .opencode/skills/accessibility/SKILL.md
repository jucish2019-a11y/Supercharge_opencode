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

## When to use me

Use this skill when:
- Auditing a page or component for accessibility issues
- Building new UI components that must be accessible
- Fixing keyboard navigation or focus issues
- Adding ARIA labels and roles
- Ensuring form inputs have proper labels and error associations
- Making dynamic content (modals, menus, toasts) accessible

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

## Guidelines

- Prefer semantic HTML over ARIA — first law of ARIA: don't use ARIA
- All interactive elements must be keyboard reachable and operable
- Every form input needs an associated label
- Focus must always be visible — never remove focus indicators
- Modals must trap focus and restore it on close
- Test with Tab, Shift+Tab, Enter, Space, Escape as minimum keyboard testing
- Don't use `tabindex` above 0 — use 0 for custom interactive elements, -1 for programmatic focus