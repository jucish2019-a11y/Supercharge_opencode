---
name: ux-patterns
description: Apply industry-standard UX patterns for navigation, forms, data display, feedback, and complex interactions with production-quality implementation
---

## What I do

I implement proven UX patterns that users already understand, making interfaces intuitive without a learning curve:

- **Navigation patterns** — Sidebar, top nav, breadcrumbs, tabs, command palette
- **Form patterns** — Input validation, error states, multi-step forms, autosave
- **Data display** — Tables, lists, cards, empty states, loading states
- **Feedback patterns** — Toasts, alerts, inline errors, success confirmations
- **Interaction patterns** — Modals, drawers, popovers, drag-and-drop, infinite scroll

## When to use me

Use this skill when:
- Building a common UI pattern (navigation, forms, tables, modals)
- Deciding which UX pattern fits a use case
- Implementing interaction states, loading, empty, or error states
- Making an interface intuitive by using established patterns

## How I work

1. **Identify the user need** — What is the user trying to accomplish? What information do they need? What actions can they take?
2. **Choose the right pattern** — Match the need to the most established, expected pattern. Surprising users with novel patterns is a cost, not a feature.
3. **Implement all states** — Default, loading, empty, error, success. Every pattern must handle every state.
4. **Add progressive disclosure** — Show essential info first, reveal details on interaction.
5. **Validate with heuristics** — Visibility of system status, user control, consistency, error prevention.

## Navigation patterns

### Sidebar navigation
- Use for: Apps with 5+ top-level sections, dashboards, tools
- Pattern: Fixed left sidebar (240-280px), collapsible to icons (64px) on smaller screens
- Active state: Background fill + left border accent
- Hover: Subtle background tint, no border
- Group items with labels ("Main", "Settings", "Account")
- Show 4-7 items per group. More = overflow menu.

### Top navigation
- Use for: Marketing sites, apps with 3-7 sections, simple products
- Pattern: Horizontal bar with logo left, nav center, actions right
- Active state: Underline or bold text, not background fill
- Mobile: Hamburger menu with full-height drawer
- Max 7 items — more needs a different pattern

### Command palette
- Use for: Power users, complex apps, keyboard-first workflows
- Pattern: `Cmd+K` / `Ctrl+K` opens a floating search/command bar
- Shows recent commands, navigation, and search results
- Fuzzy search across all commands and pages
- Keyboard navigable (arrows + enter)
- Pattern: Spotlight/VS Code style

### Breadcrumbs
- Use for: Deep hierarchies, file systems, multi-level settings
- Pattern: `Home > Projects > Project Name > Settings`
- Separator: `/` or `>` with subtle color
- Last item is not a link (current page)
- Hidden on mobile, replaced with back button

## Form patterns

### Input validation
- Validate on blur (not on every keystroke)
- Show errors below the field, aligned left
- Error text is specific: "Email must include @" not "Invalid input"
- Success: green checkmark icon after validated field
- Disable submit until all required fields are valid (disable with tooltip showing what's missing)

### Multi-step forms
- Show progress indicator (step 1 of 3)
- Allow back navigation — always
- Save progress at each step (localStorage or API)
- Step labels: "1. Account → 2. Profile → 3. Review"
- Submit on the last step only

### Autosave
- Show "Saving..." indicator (inline or in header)
- Show "Saved" with checkmark on success
- Debounce input by 500ms before saving
- Handle offline: queue saves, sync when reconnected
- Never lose user data — persistent draft state

## Data display patterns

### Tables
- Sticky header for long tables
- Sortable columns with visual indicator (↑↓→)
- Column widths proportional to content, not equal
- Row hover: subtle background highlight
- Selection: checkbox column (left side)
- Actions: kebab menu (⋯) per row, or toolbar for bulk actions
- Responsive: card layout on mobile, horizontal scroll on tablet
- Empty state: illustration + message + CTA

### Lists
- Consistent item height (64-80px with two lines, 48px single line)
- Leading element: avatar, icon, or thumbnail (40-48px)
- Primary text: bold/semibold, single line truncated
- Secondary text: regular weight, secondary color
- Trailing element: metadata, badge, or action
- Dividers between items: 1px subtle border
- Swipe actions on mobile (iOS pattern)

### Cards
- Consistent padding: 16-24px
- Border radius: 8-12px for cards, 6px for nested elements
- Hover: slight shadow lift or background shift
- Content hierarchy: title (semibold) → description (secondary) → metadata (tertiary)
- Maximum 3 actions per card
- Image cards: 16:9 or 4:3 ratio, object-fit: cover

### Empty states
- Friendly illustration or icon (not just text)
- Clear heading: "No projects yet"
- Descriptive subtext: "Create your first project to get started"
- Primary CTA button: "Create Project"
- Secondary link for alternative actions
- Never show a blank page

### Loading states
- Skeleton screens for initial page load (not spinners)
- Inline spinners for button actions (in button, replacing text)
- Progress bars for determinate operations (file upload)
- Optimistic updates for instant-feel interactions (like, star, check)
- Never leave the user wondering if something is happening

## Feedback patterns

### Toasts
- Position: bottom-right (desktop), bottom-center (mobile)
- Auto-dismiss: 4-5 seconds, 8 seconds for long messages
- Action: one inline action ("Undo") max
- Types: success (green), error (red), warning (amber), info (blue)
- Stacks: show 3 max, queue additional, dismiss oldest first
- Dismiss: swipe on mobile, X button on desktop

### Alerts
- Use for: Important information the user must see before proceeding
- Inline alerts: within content flow (above a form, inside a card)
- Full-width alerts: at the top of the page for system-wide issues
- Dismissible: always provide an X unless the alert prevents action
- Persistent: alerts that block interaction must have a clear next step

### Inline errors
- Show immediately below the relevant field
- Red border on the input, red error text below
- Error text is specific and actionable: "Password must be at least 8 characters"
- Clear error when the user starts typing in the field
- Never clear error on blur — only on valid input

## Interaction patterns

### Modals / Dialogs
- Use for: focused tasks, confirmations, critical decisions
- Overlay: 60% opacity backdrop
- Size: max 520px wide (small), 680px (medium), 840px (large)
- Close: X button, Escape key, clicking backdrop
- Focus trap: Tab cycle stays inside modal
- Return focus on close
- Never stack modals

### Drawers / Side panels
- Use for: detail views, forms, settings, filters
- Width: 380-420px (mobile: full screen)
- Slide in from right (convention for detail), left for navigation
- Overlay backdrop (desktop) or push content (optional)
- Close: swipe, Escape, X button, clicking outside

### Popovers / Dropdowns
- Use for: menus, tooltips, quick actions, overflow options
- Appear on click (menus) or hover (tooltips)
- Position: prefer below-left, flip if near edge
- Size: fit content, max 320px wide
- Dismiss: click outside, Escape
- Items: 8px padding, hover background, icon + text pattern

### Drag and drop
- Visual affordance: drag handle (⋮⋮ or ⠿) on the left side
- Pickup: element lifts (shadow increases, slight scale up 1.02)
- Dragging: 50% opacity, drop zone highlights
- Drop: smooth animation to final position (200ms spring)
- Cancel: animate back to origin (200ms ease-out)
- Touch: 200ms hold before drag starts (to not conflict with scroll)

## Quality checklist

- [ ] Every interactive element has default, hover, focus, active, disabled states
- [ ] Every data display has loading, empty, error, and populated states
- [ ] Forms show validation on blur and clear errors on input
- [ ] Modals trap focus and restore it on close
- [ ] Toasts auto-dismiss and stack correctly
- [ ] Navigation shows current location clearly
- [ ] Empty states guide the user to take action
- [ ] Loading states communicate what's happening
- [ ] Error states explain what went wrong and how to fix it

## Anti-patterns I avoid

- Custom patterns when established ones exist — user expectations are powerful
- Hiding primary actions in menus — important actions should be visible
- Infinite scroll without a footer — users need a sense of completion
- Forms without validation feedback — users need to know what's expected
- Modals for content that fits inline — modals are disruptive
- Disabling buttons without explaining why — show what's needed
- Using alert() for confirmations — always use custom dialogs
- Ghost/placeholder text that disappears — labels should persist