---
name: internationalization
description: Design and implement UIs that work globally — RTL layouts, text expansion, cultural adaptation, number/date/currency formatting, and multi-language architecture
---

## What I do

I design and implement interfaces that work for users in every language, region, and culture:

- **RTL layouts** — Mirror layouts for Arabic, Hebrew, Farsi, Urdu
- **Text expansion** — Design for 40-200% text length increase in translations
- **Cultural adaptation** — Color meanings, icon semantics, name formats, reading direction
- **Number, date, and currency formatting** — Locale-aware display and input
- **Multi-language architecture** — String externalization, ICU message format, pluralization
- **Layout resilience** — Flexible containers that don't break with long text

## When to use me

Use this skill when:
- Building a product that will be translated into 2+ languages
- Adding RTL (right-to-left) support to an existing LTR layout
- Designing forms that handle international names, addresses, and phone numbers
- Formatting dates, times, numbers, and currencies for multiple locales
- Choosing internationalization libraries and architecture
- Fixing layout breakage caused by long translated text

## How I work

1. **Assess the target markets** — Which languages? Which regions? What's the priority order?
2. **Design for the worst case** — Use the longest translation as the design constraint, not English.
3. **Externalize all strings** — Never hardcode text in components. Use an i18n library.
4. **Handle text direction** — Build CSS that works in both LTR and RTL without `direction:` overrides everywhere.
5. **Format data by locale** — Dates, numbers, currencies, names must respect local conventions.
6. **Test with pseudo-localization** — Fake translations that expose layout issues before real translation.

## Text expansion planning

Every language has different text length characteristics. Design for the longest, not the shortest:

| Language | Expansion vs English | Common issues |
|----------|---------------------|---------------|
| English | 0% (baseline) | — |
| French | +15-20% | Wordy, many articles |
| German | +20-35% | Compound words, longer terms |
| Finnish | +30-60% | Agglutinative, very long words |
| Russian | +20-30% | Cyrillic, longer words |
| Japanese | -10-20% | Kanji is compact, but needs more vertical space |
| Chinese | -20-30% | Most compact, but needs specific font sizes |
| Arabic | +15-25% | RTL, ligatures, diacritics |

### Designing for expansion

**Buttons:**
- Never fix button width in pixels. Use `min-width` + `auto` width.
- English: "Save" → German: "Speichern" → Finnish: "Tallentaa"
- Use `padding: 8px 16px` not `width: 80px`

**Labels and headings:**
- Allow text to wrap to 2 lines. Never `white-space: nowrap` on translatable text.
- Test with pseudo-localization strings that are 40% longer.
- Truncation with tooltip as last resort — never first choice.

**Navigation:**
- Sidebar nav: allow items to wrap to 2 lines or use icon + collapsible text
- Top nav: use "More" overflow menu at 6-7 items (not 10+)
- Breadcrumbs: truncate middle segments, keep first and last

**Tables:**
- Column widths: percentage-based, not fixed pixels
- Header text: allow 2-line wrapping or use tooltips for full text
- Consider hiding less important columns on smaller screens

**Forms:**
- Labels above inputs (not side-by-side) — more resilient to length changes
- Error messages below inputs, full width — no width constraints
- Dropdown/select: allow option text to wrap

### CSS patterns for text resilience

```css
/* Flexible button — grows with text */
.button {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-4);
  white-space: nowrap; /* OK for buttons — one line */
  min-width: 64px;
  /* NO fixed width */
}

/* Flexible nav item — wraps if needed */
.nav-item {
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  /* For tooltips on hover showing full text */
}

/* Flexible card title — 2 lines max */
.card-title {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Flexible form label — stacks if needed */
.form-label {
  /* Label above input, always full width */
  display: block;
  margin-bottom: var(--space-1);
  word-break: break-word;
}
```

## RTL (Right-to-Left) layout

### Languages that need RTL
- Arabic (ar)
- Hebrew (he, iw)
- Farsi/Persian (fa)
- Urdu (ur)
- Pashto (ps)
- Sindhi (sd)

### CSS logical properties

Replace every physical CSS property with a logical equivalent:

| Physical (LTR-only) | Logical (direction-aware) | What it does |
|---------------------|--------------------------|--------------|
| `margin-left` | `margin-inline-start` | Start-side margin |
| `margin-right` | `margin-inline-end` | End-side margin |
| `padding-left` | `padding-inline-start` | Start-side padding |
| `padding-right` | `padding-inline-end` | End-side padding |
| `border-left` | `border-inline-start` | Start-side border |
| `border-right` | `border-inline-end` | End-side border |
| `left` | `inset-inline-start` | Start position |
| `right` | `inset-inline-end` | End position |
| `text-align: left` | `text-align: start` | Start alignment |
| `text-align: right` | `text-align: end` | End alignment |
| `float: left` | `float: inline-start` | Start float |
| `float: right` | `float: inline-end` | End float |

### Flexbox and Grid

```css
/* Use row direction that respects dir attribute */
.container {
  display: flex;
  flex-direction: row; /* Respects dir — RTL automatically reverses */
}

/* If you used row-reverse for LTR-specific reasons, switch to logical approach */
.sidebar-right {
  order: 1; /* LTR: right side; RTL: reverses automatically with dir */
}
```

For CSS Grid:
```css
.grid {
  display: grid;
  grid-template-columns: auto 1fr auto;
  /* Columns respect writing direction automatically */
}
```

### Setting direction

```html
<html dir="rtl" lang="ar">
```

Or dynamically:
```js
document.documentElement.dir = isRTL(locale) ? 'rtl' : 'ltr';
```

### What flips in RTL

These elements should flip to the opposite side:
- Text alignment (left → right)
- Icons with directional meaning (arrows, back/forward, breadcrumbs "→" → "←")
- Navigation sidebars (left → right)
- Toast/notification positions (bottom-right → bottom-left)
- Avatar + name layout (avatar left → avatar right)
- Checkbox/radio position (left of label → right of label)
- Progress bar fill direction (left-to-right → right-to-left)
- Table row actions (right side → left side)

### What does NOT flip in RTL

These elements stay the same direction:
- Numbers (always LTR within RTL text)
- Code snippets (always LTR)
- LTR brand logos
- Math and scientific notation
- Clock faces and time
- Media playback controls (play is always ▶)
- Physical direction icons (North, East, West)

### RTL-specific icons

| Icon | LTR meaning | RTL handling |
|------|------------|--------------|
| Arrow right (→) | Forward / Next | Flip to ← |
| Arrow left (←) | Back / Previous | Flip to → |
| Chevron right (⟩) | Expand / Navigate in | Flip to ⟨ |
| Chevron left (⟨) | Collapse / Navigate out | Flip to ⟩ |
| Indent | Indent right | Flip to indent left |
| List bullet | Left-aligned | Keep, but text wraps to left |
| Refresh (↻) | Refresh | Don't flip (circular, no direction) |

```css
/* Flip directional icons in RTL */
[dir="rtl"] .icon-directional {
  transform: scaleX(-1);
}
```

## Number, date, and currency formatting

### Numbers

| Locale | Format | Example |
|--------|--------|---------|
| en-US | Comma separator, period decimal | 1,234,567.89 |
| de-DE | Period separator, comma decimal | 1.234.567,89 |
| fr-FR | Space separator, comma decimal | 1 234 567,89 |
| hi-IN | Indian number system | 12,34,567.89 |
| ar-SA | Arabic-Indic digits | ١٬٢٣٤٬٥٦٧٫٨٩ |

```js
// Always use Intl.NumberFormat
new Intl.NumberFormat('de-DE').format(1234567.89);
// "1.234.567,89"

new Intl.NumberFormat('en-IN').format(1234567.89);
// "12,34,567.89"
```

### Dates

| Locale | Format | Example |
|--------|--------|---------|
| en-US | Month/Day/Year | 04/15/2026 |
| en-GB | Day/Month/Year | 15/04/2026 |
| de-DE | Day.Month.Year | 15.04.2026 |
| ja-JP | Year年Month日Day | 2026年4月15日 |
| zh-CN | Year年Month月Day日 | 2026年4月15日 |
| ar-SA | Day/Month/Year (Arabic) | ١٥/٠٤/٢٠٢٦ |

```js
// Always use Intl.DateTimeFormat
new Intl.DateTimeFormat('de-DE', { dateStyle: 'long' }).format(date);
// "15. April 2026"
```

**Never parse dates from strings without knowing the format.** "04/05/2026" is April 5 in the US and May 4 in the UK. Use ISO 8601 (YYYY-MM-DD) for all stored/transported dates.

### Currency

| Locale | Currency | Format |
|--------|----------|--------|
| en-US | USD | $1,234.56 |
| de-DE | EUR | 1.234,56 € |
| ja-JP | JPY | ￥1,234 |
| fr-FR | EUR | 1 234,56 € |

```js
// Always use Intl.NumberFormat with currency
new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(1234);
// "￥1,234"
```

### Relative time

| English | German | Japanese |
|---------|--------|----------|
| "3 days ago" | "vor 3 Tagen" | "3日前" |
| "in 2 hours" | "in 2 Stunden" | "2時間後" |

```js
new Intl.RelativeTimeFormat('de-DE', { numeric: 'auto' }).format(-3, 'day');
// "vor 3 Tagen"
```

### Pluralization

Pluralization rules vary wildly by language:

| Language | Plural categories | Example |
|----------|------------------|---------|
| English | one, other | 1 item / 2 items |
| French | one, other | 1 élément / 2 éléments (0 uses "other") |
| Russian | one, few, many, other | 1 элемент / 2 элемента / 5 элементов |
| Arabic | zero, one, two, few, many, other | ٠ عناصر / ١ عنصر / ٢ عنصرين / ٣ عناصر |
| Japanese | other only | 1アイテム / 2アイテム (no plural form) |
| Polish | one, few, many, other | 1 element / 2 elementy / 5 elementów |

**Never hardcode English plural rules.** Use ICU message format:

```json
{
  "itemCount": "{count, plural, one {# item} other {# items}}"
}
```

```js
// ICU format handles all languages
new Intl.PluralRules('ar').select(0);    // "zero"
new Intl.PluralRules('ar').select(1);    // "one"
new Intl.PluralRules('ar').select(2);    // "two"
new Intl.PluralRules('ar').select(3);    // "few"
new Intl.PluralRules('ar').select(11);   // "many"
new Intl.PluralRules('ar').select(100);  // "other"
```

## International names and addresses

### Name fields

Names are NOT "First Name + Last Name" everywhere:

| Culture | Name structure | Example |
|---------|--------------|---------|
| Western | Given + Family | John Smith |
| Chinese | Family + Given | 张三 (Zhang San) |
| Hindi | Given + Patronymic + Family | Rahul Kumar Sharma |
| Arabic | Given + Father + Grandfather + Family | محمد بن عبدالله بن خالد القحطاني |
| Spanish | Given + Paternal + Maternal | Juan García López |
| Icelandic | Given + Patronymic | Björk Guðmundsdóttir |

**Design for:**
- A single "Full name" field whenever possible — fewer assumptions, fewer errors
- If you need separate fields: "Given name" and "Family name" (not "First" and "Last")
- Allow 50+ characters for name fields
- Allow letters, spaces, hyphens, apostrophes — no "letters only" validation
- Don't assume the family name is the last name (it's first in many cultures)
- Don't require a family name (Icelandic, some Indonesian names have none)

### Address fields

The US address format (Street, City, State, ZIP) doesn't work globally:

| Country | Format |
|---------|--------|
| US | Street, City, State ZIP |
| UK | House number + Street, Town, County, Postcode |
| Japan | 〒Postal, Prefecture, City, District, Block, Building (largest to smallest) |
| China | Province, City, District, Street, Building (largest to smallest) |
| Germany | Street + Number, PLZ City |

**Design for:**
- Use the country's standard address format, shown after country selection
- Postal codes: allow variable length (US: 5, UK: 6-8, Canada: 6, Japan: 7)
- State/province is not universal — show/hide based on country
- Some countries don't use postal codes at all
- House number position varies: "123 Main St" (US) vs "Main St 123" (Germany)
- Use Google's libaddressin if you need structured international addresses

### Phone numbers

```js
// Always use libphonenumber or equivalent
// Format: +[country code] [national number]
// Display: locale-aware formatting

// US: +1 (555) 123-4567
// UK: +44 20 7946 0958
// Japan: +81 3-1234-5678
```

- Store as E.164 format: `+15551234567`
- Display in the user's local format
- Always include the country code selector
- Phone number lengths vary: 7-15 digits internationally
- Some countries use extensions

## Pseudo-localization

Test your layout before real translations arrive using pseudo-localization:

### Pseudolocale strings

```
// English: "Save changes"
// Pseudo (40% longer): "[Šâvéééé chángésés ẋẋẋ]"
// Pseudo (RTL simulation): "Save changes ẋ]"
// Pseudo (CJK width): "Ｓａｖｅ ｃｈａｎｇｅｓ"
```

### What pseudo-localization exposes

| Issue | What you see |
|-------|-------------|
| Text overflow | Pseudo-text spills out of containers |
| Layout breaks | Fixed-width elements can't contain long text |
| Hardcoded strings | Some text stays in English (wasn't externalized) |
| Concatenated strings | Grammatically broken sentences in pseudo-locale |
| RTL issues | Pseudo-RTL reveals direction bugs (if using pseudo-RTL) |
| Font fallbacks | CJK/Arabic pseudo shows if fonts support the glyphs |

### Implementation

```json
// pseudolocalize package or custom implementation
{
  "en": "Save changes",
  "en-XA": "[Šâvé chángés ẋẋẋẋ]",  // 40% longer, accented
  "en-XB": "]ségnahc evaS[",         // RTL simulation
  "en-XC": "Ｓａｖｅ ｃｈａｎｇｅｓ"   // CJK width
}
```

Build your app with `en-XA` as the locale and every layout issue becomes immediately visible.

## Cultural considerations

### Color meanings vary by culture

| Color | Western | Chinese | Indian | Middle Eastern |
|-------|---------|---------|--------|---------------|
| Red | Danger, stop | Luck, prosperity | Purity, fertility | Danger |
| Green | Success, go | Infidelity | Islam, fertility | Islam, luck |
| White | Purity, peace | Death, mourning | Mourning | Purity |
| Yellow | Warning, caution | Royalty, sacred | Sacred | Mourning |
| Blue | Trust, calm | Healing | Krishna | Trust |

- Don't assume green = positive everywhere
- Use shape and icon alongside color to convey meaning
- For global products, use universal symbols: ✓ for success, ✕ for error, ⚠ for warning

### Icon semantics vary

| Icon | Western meaning | May not work in |
|------|----------------|-----------------|
| 🦉 Owl | Wisdom | Some cultures: bad omen |
| 👍 Thumbs up | Approval | Middle East: offensive |
| 🤙 Call me | Surf culture | Most places: unclear |
| 🐦 Bird | Freedom | Some cultures: bad luck |
| ✋ Stop | Stop | Some cultures: greeting |

- Prefer abstract/geometric icons over cultural symbols
- Avoid hand gestures
- Avoid animals with cultural significance
- Avoid religious symbols

### Name and calendar systems

- Fiscal year: starts April in Japan, July in Australia, January in US
- First day of week: Sunday (US), Monday (Europe), Saturday (Middle East)
- Weekend: Saturday+Sunday (most), Friday+Saturday (Middle East)
- Era: 2026 CE (Western), 令和8年 (Japanese), 1447 AH (Islamic), 4723 (Chinese)

## i18n architecture

### String externalization

```js
// BAD — hardcoded
<button>Delete project</button>

// GOOD — externalized
<button>{t('project.delete.button')}</button>

// Translation file (en.json)
{
  "project": {
    "delete": {
      "button": "Delete project",
      "confirmation": "Delete project?",
      "description": "This will permanently delete \"{name}\" and all its tasks. This can't be undone.",
      "success": "Project deleted."
    }
  }
}
```

### Message format (ICU)

```json
{
  "greeting": "Hello, {name}!",
  "itemCount": "{count, plural, one {# item} other {# items}}",
  "taskAssigned": "{gender, select, male {He} female {She} other {They}} assigned you a task.",
  "lastLogin": "Last login: {time, relative}"
}
```

### Library selection

| Library | Framework | Features |
|---------|-----------|----------|
| react-intl | React | ICU messages, number/date formatting, pluralization |
| next-intl | Next.js | Server components, App Router, ICU |
| i18next | Any | Flexible, plugin-based, large ecosystem |
| FormatJS | Any | ICU message format standard |
| lingui | React | Compile-time extraction, smaller bundles |

## Quality checklist

- [ ] All user-facing text is externalized (zero hardcoded strings)
- [ ] CSS uses logical properties (margin-inline-start, not margin-left)
- [ ] Buttons and containers flex with text length (no fixed widths)
- [ ] Long text wraps or truncates gracefully (2-line max for headings)
- [ ] RTL layout works with `dir="rtl"` on the html element
- [ ] Directional icons flip in RTL (arrows, chevrons)
- [ ] Numbers formatted with Intl.NumberFormat for user's locale
- [ ] Dates formatted with Intl.DateTimeFormat for user's locale
- [ ] Currencies formatted with Intl.NumberFormat currency option
- [ ] Pluralization uses Intl.PluralRules or ICU format (not English rules)
- [ ] Name fields don't assume Western naming conventions
- [ ] Address format adapts based on selected country
- [ ] Phone numbers stored in E.164, displayed in local format
- [ ] Pseudo-localization (en-XA) test run shows no layout breaks
- [ ] First day of week adapts to locale
- [ ] Text expansion of 40% doesn't break any layout
- [ ] No cultural assumptions in icons, colors, or symbolism

## Anti-patterns I avoid

- Hardcoded text in components — always externalize
- Physical CSS properties (left, right) — use logical (inline-start, inline-end)
- Fixed-width containers for text — use min-width + auto or max-width
- English-only plural rules — use ICU format with all plural categories
- "First Name" + "Last Name" labels — use "Given name" + "Family name" or "Full name"
- US address format for all countries — adapt per country
- MM/DD/YYYY date display — use locale-aware formatting
- Color-only indicators for status — add icons/shapes for colorblind and cultural safety
- Cultural symbols (animals, hand gestures) as icons — use abstract shapes
- Concatenating translated string fragments — grammar doesn't work across languages
- Assuming all scripts are LTR — always support RTL
- Translating the product without translating the design — text length changes require layout changes