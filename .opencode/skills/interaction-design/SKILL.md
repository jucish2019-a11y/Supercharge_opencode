---
name: interaction-design
description: Design drag-and-drop, gesture handling, command palette, keyboard shortcuts, touch interactions, and power-user patterns that make interfaces feel premium and efficient
---

## What I do

I design interaction patterns that make interfaces feel responsive, efficient, and physically coherent — the difference between a "good" UI and a "professional" one:

- **Drag and drop** — Reorderable lists, kanban boards, file uploads, resize handles
- **Gesture handling** — Swipe, pinch, pull-to-refresh, long-press, multi-touch
- **Command palette** — Keyboard-first navigation and search (Cmd+K)
- **Keyboard shortcuts** — Mnemonic keybindings, shortcut discoverability, conflict resolution
- **Touch interactions** — Haptic feedback zones, gesture recognizers, palm rejection
- **Focus management** — Tab order, focus trapping, roving tabindex, skip links
- **Undo/redo** — History management, optimistic updates, reversible actions

## When to use me

Use this skill when:
- Adding drag-and-drop to lists, boards, or file managers
- Building a command palette or keyboard shortcut system
- Designing touch gesture interactions for mobile
- Implementing complex form interactions (multi-select, inline editing, drag selection)
- Making a desktop app feel native (keyboard shortcuts, focus management)
- Designing power-user features (bulk actions, batch editing, quick actions)

## How I work

1. **Identify the interaction model** — What input methods does this support? Mouse, keyboard, touch, stylus?
2. **Define the gesture vocabulary** — Map user intentions to interactions.
3. **Design all feedback states** — Idle, hover, active, dragging, selected, hovered-over-target, invalid, success, cancel.
4. **Handle edge cases** — Touch vs. mouse conflicts, interrupted gestures, boundary conditions, concurrent interactions.
5. **Implement with platform APIs** — Pointer Events, Keyboard Events, Intersection Observer, Drag and Drop API.
6. **Test on all input devices** — Mouse, trackpad, touch, keyboard. Each has different expectations.

## Drag and drop

### Drag initiation

**Mouse:**
- mousedown on drag handle → 150ms hold → drag begins
- The 150ms delay prevents accidental drags during normal clicking
- Visual: cursor changes to `grabbing`, element lifts (shadow increase + scale 1.02)

**Touch:**
- touchstart on drag handle → 200ms hold or 8px movement → drag begins
- The 200ms prevents conflict with scroll gestures
- Visual: haptic feedback on initiation (if available), element lifts

**Keyboard:**
- Focus on item → Space/Enter to pick up → Arrow keys to move → Space/Enter to drop → Escape to cancel
- Visual: picked-up item has a visible border, arrow keys move the insertion indicator

### Drag states

```
Idle:        Default appearance
Hover:       Pointer cursor on handle, handle highlights
Pending:     150ms hold started, hasn't moved yet
Dragging:    Element follows pointer, elevated (shadow + scale), 50% opacity at origin
Over target: Drop zone highlights (background color, border, gap opens)
Invalid:     Drop zone shows "not allowed" state (red border, icon)
Drop:        Element settles into position with spring animation
Cancel:      Element springs back to origin (200ms ease-out)
```

### Drop zone behavior

```
List reordering:
  Gap opens at the insertion point (8px → space equal to dragged item height)
  Gap follows the pointer — moves smoothly, not jumping
  
Kanban columns:
  Column highlights on hover (border color change)
  Cards in column shift down to make room (smooth, 200ms)
  Invalid column: red border, "not allowed" cursor
  
File upload:
  Full-page overlay appears on dragenter
  Drop zone animates border-dash (marching ants)
  File type validation: show icons for accepted types
  
Cross-container:
  Drag from container A to container B
  Container A: item stays as ghost (30% opacity) 
  Container B: gap opens for insertion
```

### Drag implementation

```js
// Pointer Events (unified mouse + touch)
let isDragging = false;
let startX, startY;
const DRAG_THRESHOLD = 8; // pixels before drag activates

element.addEventListener('pointerdown', (e) => {
  if (!isOnDragHandle(e)) return;
  startX = e.clientX;
  startY = e.clientY;
  element.setPointerCapture(e.pointerId);
  
  // Start hold timer
  holdTimer = setTimeout(() => {
    prepareDragFeedback();
  }, 150);
});

element.addEventListener('pointermove', (e) => {
  const dx = e.clientX - startX;
  const dy = e.clientY - startY;
  
  if (!isDragging && Math.hypot(dx, dy) > DRAG_THRESHOLD) {
    isDragging = true;
    clearTimeout(holdTimer);
    startDrag(e);
  }
  
  if (isDragging) {
    updateDragPosition(e);
    updateDropZone(e);
  }
});

element.addEventListener('pointerup', (e) => {
  if (isDragging) {
    completeDrop(e);
  } else {
    cancelDrag();
  }
  isDragging = false;
});
```

### Accessibility for drag and drop

Drag and drop is inherently difficult for keyboard and screen reader users. Always provide an alternative:

- **Keyboard**: Arrow keys to move items up/down in a list (roving tabindex)
- **Screen reader**: Announce position ("Item 3 of 10. Press Space to pick up. Use arrow keys to reorder. Press Space to drop.")
- **Buttons**: "Move up" / "Move down" buttons visible on focus or always visible
- **Select + move**: Select multiple items, then use toolbar buttons to move them

```html
<li role="listitem" aria-grabbed="false" aria-label="Project Alpha, position 3 of 10" tabindex="0">
  <button aria-label="Move up" onclick="moveUp(3)">↑</button>
  <button aria-label="Move down" onclick="moveDown(3)">↓</button>
  Project Alpha
</li>
```

## Gesture handling

### Swipe gestures

**Swipe to dismiss/delete:**
```
Threshold: 60px horizontal movement OR 40% of item width
Velocity threshold: 0.5px/ms (fast flick = dismiss even if <60px)
Direction: horizontal only (prevent conflict with vertical scroll)
Haptic: trigger on threshold crossing

States:
  0-20px:   No visual action (dead zone for scroll)
  20-60px:  Item follows finger, reveal action underneath (red delete)
  >60px:    Snap to revealed state, show full action
  >60px + release: execute action with confirmation
  <60px + release: spring back to origin (250ms)
  
Visual:
  Item opacity decreases as it slides (1.0 → 0.6)
  Action button revealed underneath (red background, white icon)
  If dismissed: item shrinks to 0 height, adjacent items close gap (FLIP)
```

**Swipe to refresh (pull-to-refresh):**
```
Threshold: 60px downward pull
Activation: spinner appears at 40px, rotates at 60px
Rubber-band effect: over-pull decelerates (overscroll-behavior)

States:
  0-40px:   Pulling indicator (arrow rotates from down to up)
  40-60px:  Indicator changes to "release to refresh"
  >60px:    Release triggers refresh
  Refreshing: Spinner animates, content has opacity overlay
  
Reset: content slides back down, refresh indicator fades
Duration: 300ms ease-decelerate for reset
```

### Long-press

```
Activation: 500ms continuous press (no movement >10px)
Feedback: haptic at 500ms, slight scale-down (0.98) on element
Context menu: appears at press location after activation

Use for:
  Mobile: right-click equivalent (context menu)
  Multi-select: first long-press enters selection mode, subsequent taps toggle selection
  Drag initiation: 200ms hold starts drag (not context menu)
  
Conflict resolution:
  Long-press < 200ms: normal tap
  Long-press 200-500ms: drag begins (if moved >8px)
  Long-press > 500ms: context menu or selection mode
```

### Multi-touch

```
Pinch-to-zoom (maps, images):
  Min scale: 1.0 (can't zoom out below content fit)
  Max scale: 5.0
  Double-tap: toggle between 1.0 and 2.5 (animated, 300ms)
  Pinch: scale maps linearly to finger distance
  
Two-finger rotation (image editing):
  Rotation snaps to 15° increments with haptic
  Visual: rotation indicator shows current angle
  
Split-tap (accessibility):
  One finger holds, second tap = different action
  Screen reader users: explore by touch + second tap to activate
```

## Command palette

The command palette is the most powerful interaction pattern for complex apps:

### Structure

```
┌─────────────────────────────────────────┐
│ 🔍 Search commands, pages, actions...    │  ← Search input
├─────────────────────────────────────────┤
│ Recent                                   │  ← Category header
│  → Open Project Alpha                   │  ← Result item
│  → Create new task                      │
│ Actions                                 │
│  → Invite team member                   │
│  → Export CSV                           │
│ Pages                                   │
│  → Settings                             │
│  → Billing                              │
├─────────────────────────────────────────┤
│ ↑↓ navigate  ↵ select  esc close       │  ← Keyboard hints
└─────────────────────────────────────────┘
```

### Trigger
- `Cmd+K` / `Ctrl+K` — industry standard (Notion, Vercel, Linear, Raycast)
- Or `Cmd+J` / `Ctrl+J` if K conflicts
- Overlay: 60% opacity backdrop, centered modal (max 560px wide)

### Search behavior
- Fuzzy search across all commands, pages, and recent items
- Search after 1 character (no minimum)
- Results categorized: Recent → Actions → Pages → Settings
- Maximum 8 results visible (scroll for more)
- No results: "No results for 'xyz'. Try a different search."

### Keyboard navigation
```
↑/↓:        Move selection up/down
Enter:      Execute selected item
Tab:        Switch category (if nested)
Escape:     Close palette (one level at a time for nested)
Backspace:  Navigate up one level (if in a submenu)
```

### Grouping commands

```
Prefix-based navigation (like Raycast):
  "project" → Shows project-related commands
  ">export" → Quick action mode
  "/settings" → Page navigation mode
  
Nested menus:
  "Create" → Submenu shows: Project, Task, Document, Team
  Selected → Enter opens submenu, shows children
  Backspace → Returns to parent menu
```

### Implementation

```tsx
function CommandPalette() {
  const [open, setOpen] = useState(false);
  const [query, setQuery] = useState('');
  
  // Listen for Cmd+K
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        setOpen(prev => !prev);
      }
      if (e.key === 'Escape') setOpen(false);
    };
    document.addEventListener('keydown', handler);
    return () => document.removeEventListener('keydown', handler);
  }, []);
  
  return (
    <AnimatePresence>
      {open && (
        <Modal>
          <SearchInput value={query} onChange={setQuery} />
          <Results items={filterCommands(query)} />
        </Modal>
      )}
    </AnimatePresence>
  );
}
```

## Keyboard shortcuts

### Designing shortcut systems

**Principles:**
1. Mnemonic: the key should relate to the action (Ctrl+S for Save, not Ctrl+Q)
2. Consistent: same shortcut across the app for the same action
3. Non-destructive: destructive shortcuts require confirmation or undo
4. Discoverable: shortcuts shown in menus and tooltips
5. No browser conflicts: never override browser shortcuts (Ctrl+T, Ctrl+W, Ctrl+L)

**Modifier convention:**

| Platform | Primary modifier | Secondary modifier |
|----------|------------------|-------------------|
| macOS | ⌘ Command | ⌥ Option |
| Windows/Linux | Ctrl | Alt |

**Common shortcuts (use these — don't invent new ones):**

```
Ctrl/Cmd + S      Save
Ctrl/Cmd + Z      Undo
Ctrl/Cmd + Shift + Z  Redo
Ctrl/Cmd + C      Copy
Ctrl/Cmd + V      Paste
Ctrl/Cmd + X      Cut
Ctrl/Cmd + A      Select all
Ctrl/Cmd + F      Find/search
Ctrl/Cmd + K      Command palette
Ctrl/Cmd + P      Print / Preview
Ctrl/Cmd + N      New (file, project, etc.)
Ctrl/Cmd + W      Close current tab
Ctrl/Cmd + /      Toggle comment (code editors)
Ctrl/Cmd + Shift + P  Command palette (VS Code style)
```

### Custom shortcut rules

```
Single letter shortcuts (no modifier):
  Only in focused contexts (text editors, canvas)
  Examples: j/k for up/down (Gmail/Vim), x for select, e for edit
  
Modifier + letter:
  Global shortcuts, always active
  Examples: Cmd+Shift+N (new private window), Cmd+E (export)
  
Modifier + modifier + letter:
  Advanced/rare actions
  Examples: Cmd+Ctrl+Shift+V (paste without formatting)
  
Number keys:
  Tab switching: Ctrl+1 through Ctrl+9
  Zoom levels: Ctrl+0 (reset), Ctrl+- (zoom out), Ctrl+= (zoom in)
```

### Shortcut discovery

```
In menus:        Show shortcut next to the menu item
                  Save          ⌘S
                
In tooltips:     Show shortcut on hover
                  Create task   Ctrl+N

In onboarding:   Show a "Keyboard shortcuts" card
                  Press ? to see all shortcuts

Help modal:      Global ? or Ctrl+/ opens shortcut reference
                  Searchable list of all shortcuts
                  Grouped by category
```

### Shortcut conflict resolution

```
Priority order (higher wins):
1. Focused input/editor shortcuts (e.g., Ctrl+B for bold in text editor)
2. Component-level shortcuts (e.g., arrow keys in a list)
3. Page-level shortcuts (e.g., 1-9 for tab switching)
4. App-level shortcuts (e.g., Ctrl+K for command palette)
5. Browser shortcuts (never override these)

When two features need the same shortcut:
- The more frequently used action gets the shortcut
- The less frequent action gets Shift + shortcut
- Document both in the shortcut reference
- Never silently shadow a shortcut
```

## Focus management

### Tab order
```
Logical order matches visual order:
  Left-to-right, top-to-bottom (LTR)
  Right-to-left, top-to-bottom (RTL)
  
Tab should follow the user's expected reading order,
NOT the DOM order unless DOM matches visual layout.
  
If DOM order ≠ visual order:
  Use tabindex to fix, OR
  Reorder the DOM to match visual layout (preferred)
```

### Roving tabindex

For lists and toolbars where only one item should be in the tab sequence:

```js
// Only the currently "active" item has tabindex="0"
// All others have tabindex="-1"
// Arrow keys move the active item
// Tab moves to the next widget group

function handleKeyDown(e, items, currentIndex) {
  let nextIndex = currentIndex;
  
  switch (e.key) {
    case 'ArrowDown':
    case 'ArrowRight':
      e.preventDefault();
      nextIndex = (currentIndex + 1) % items.length;
      break;
    case 'ArrowUp':
    case 'ArrowLeft':
      e.preventDefault();
      nextIndex = (currentIndex - 1 + items.length) % items.length;
      break;
    case 'Home':
      nextIndex = 0;
      break;
    case 'End':
      nextIndex = items.length - 1;
      break;
  }
  
  items[currentIndex].tabIndex = -1;
  items[nextIndex].tabIndex = 0;
  items[nextIndex].focus();
}
```

### Focus trap (modals)

```js
function trapFocus(modalElement) {
  const focusable = modalElement.querySelectorAll(
    'a[href], button:not([disabled]), input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last = focusable[focusable.length - 1];
  
  modalElement.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;
    
    if (e.shiftKey) {
      if (document.activeElement === first) {
        e.preventDefault();
        last.focus();
      }
    } else {
      if (document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  });
}
```

### Focus restoration

When a modal closes, return focus to the element that opened it:

```js
let previousFocus = null;

function openModal(trigger) {
  previousFocus = trigger; // Store the trigger
  modal.style.display = 'block';
  firstFocusable.focus(); // Move focus into modal
}

function closeModal() {
  modal.style.display = 'none';
  previousFocus.focus(); // Restore focus
}
```

### Skip links

```html
<a href="#main-content" class="skip-link">Skip to main content</a>
<!-- Skip link is visually hidden, visible on focus -->
```

```css
.skip-link {
  position: absolute;
  left: -9999px;
  top: 0;
  z-index: 9999;
  padding: 8px 16px;
  background: var(--color-primary);
  color: white;
}
.skip-link:focus {
  left: 0; /* Becomes visible on Tab focus */
}
```

## Undo/redo

### History stack

```js
class UndoManager {
  constructor(maxHistory = 50) {
    this.undoStack = [];
    this.redoStack = [];
    this.maxHistory = maxHistory;
  }
  
  execute(action) {
    action.execute();
    this.undoStack.push(action);
    this.redoStack = []; // Clear redo stack on new action
    if (this.undoStack.length > this.maxHistory) {
      this.undoStack.shift();
    }
  }
  
  undo() {
    const action = this.undoStack.pop();
    if (action) {
      action.undo();
      this.redoStack.push(action);
    }
  }
  
  redo() {
    const action = this.redoStack.pop();
    if (action) {
      action.execute();
      this.undoStack.push(action);
    }
  }
}
```

### Optimistic updates

Update the UI immediately, then confirm with the server:

```
1. Apply the change to local state immediately
2. Show a subtle "saving" indicator (inline, not a spinner)
3. Send the API request in the background
4. On success: clear "saving" indicator, mark as "saved"
5. On failure: revert the local change, show error toast with "Retry"
```

```
Undo pattern for destructive actions:
  Instead of: "Are you sure you want to delete?" [Yes] [No]
  Use: Delete immediately + show toast "Project deleted. [Undo]"
  Undo in toast: revert the deletion (5 second window)
  After 5 seconds: deletion is permanent
```

### Undo in different contexts

| Context | Undo strategy | Time window |
|---------|-------------|-------------|
| Text editing | Full character-by-character history | Unlimited (within session) |
| Destructive deletion | Toast with undo button | 5-10 seconds |
| Form edits | Unsaved changes indicator + discard | Until navigation or save |
| File operations | Trash/recycle bin | 30 days |
| Batch operations | Undo all as single action | One undo step |
| Collaborative edits | Per-user undo (not undoing others' changes) | Full session |

## Touch targets and hit areas

### Minimum sizes

| Platform | Minimum touch target | Recommended |
|----------|---------------------|-------------|
| iOS | 44×44px | 48×48px |
| Android | 48×48dp | 48×48dp |
| Web (WCAG) | 44×44px | 48×48px |

### Hit area expansion

The visible element can be smaller than the touch target:

```css
/* Small icon button (24px visible, 44px touch target) */
.icon-button {
  width: 24px;
  height: 24px;
  /* Expand hit area without expanding visual size */
  position: relative;
}

.icon-button::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 44px;
  height: 44px;
}
```

### Spacing between targets

```
Minimum gap between adjacent touch targets: 8px
Recommended gap: 12px
For icon-only buttons in a toolbar: 4px gap (touch targets overlap slightly is OK)
For list items with swipe actions: full-separation, no overlap
```

## Quality checklist

- [ ] Drag-and-drop supported on mouse, touch, and keyboard
- [ ] Drag handles are visually distinct (dots or grip icon)
- [ ] Drop zones provide clear visual feedback (highlight, gap, invalid state)
- [ ] All drag operations have keyboard-accessible alternatives
- [ ] Swipe gestures have dead zones that don't conflict with scroll
- [ ] Long-press activates at 500ms with haptic feedback
- [ ] Command palette opens with Cmd+K, supports fuzzy search
- [ ] All keyboard shortcuts are discoverable (? shortcut help)
- [ ] Shortcuts don't conflict with browser or OS shortcuts
- [ ] Focus management: tab order matches visual order
- [ ] Modals trap focus and restore it on close
- [ ] Skip links provided for main content
- [ ] Destructive actions support undo (toast with undo button)
- [ ] Touch targets minimum 44×44px (recommended 48×48px)
- [ ] Hit areas can be larger than visible element size
- [ ] Optimistic updates with server confirmation and revert on failure
- [ ] Undo/redo stack limited to 50 items, redo clears on new action

## Anti-patterns I avoid

- Drag-and-drop without keyboard alternatives — inaccessible
- Swipe gestures without a dead zone — conflicts with scrolling
- Hidden shortcuts with no discoverability — only power users know they exist
- Overriding browser shortcuts (Ctrl+T, Ctrl+W, Ctrl+L) — breaks browser functionality
- Confirmation dialogs for reversible actions — use undo instead
- Confirmation dialogs for irreversible actions without undo — give both confirm AND undo
- Focus moving without user action — never move focus programmatically unless the user triggered a context change
- Tab order that doesn't match visual order — users Tab expecting the next visible element
- Touch targets below 44px — unusable on mobile
- Drag that starts on first mousedown without a hold delay — too easy to accidentally drag