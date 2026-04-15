---
name: data-visualization
description: Design charts, graphs, dashboards, and data-heavy interfaces that are accurate, readable, and beautiful — from chart selection to axis styling to color-blind-safe palettes
---

## What I do

I design data visualizations that make complex data understandable and actionable:

- **Chart selection** — Choose the right chart type for the data story
- **Axis and scale design** — Tick marks, gridlines, labels, and formatting that don't mislead
- **Color for data** — Color-blind-safe palettes, sequential/diverging schemes, highlight strategies
- **Tooltip and annotation design** — How to reveal detail on demand without cluttering
- **Dashboard layout** — Information density, hierarchy, and scannability for data-heavy screens
- **Responsive charts** — Charts that work on mobile, tablet, and desktop without losing meaning
- **Accessibility** — Making charts understandable for screen readers and keyboard users

## When to use me

Use this skill when:
- Building dashboards, analytics pages, or data-heavy interfaces
- Choosing chart types for specific data stories
- Styling axes, gridlines, tooltips, and legends
- Designing color schemes for categorical, sequential, or diverging data
- Making charts accessible and responsive
- Deciding between table vs. chart vs. sparkline for data display

## How I work

1. **Understand the data story** — What question is the user trying to answer? What decision will they make from this data? The chart type follows from the question.
2. **Choose the right chart** — Match data relationship to visualization type.
3. **Design the chart anatomy** — Axes, gridlines, labels, colors, tooltips, legend.
4. **Handle edge cases** — Negative values, missing data, outliers, long labels, many series.
5. **Make it responsive** — Chart behavior at mobile, tablet, desktop widths.
6. **Make it accessible** — Text alternatives, keyboard interaction, screen reader tables.
7. **Make it beautiful** — Remove chart junk, maximize data-ink ratio, align to the design system.

## Chart selection guide

### By data relationship

| Question | Chart type | When to use |
|----------|-----------|-------------|
| How much? | Bar chart (vertical or horizontal) | Comparing discrete categories |
| How has it changed over time? | Line chart | Trends over continuous time periods |
| What's the composition? | Stacked bar or area chart | Parts of a whole over time or categories |
| What's the proportion? | Donut chart (not pie) | Simple part-to-whole with ≤5 segments |
| How do two variables relate? | Scatter plot | Correlation, clusters, outliers |
| How is this distributed? | Histogram or violin plot | Frequency distribution |
| What's the range and median? | Box plot or range chart | Distribution comparison across categories |
| What's the flow? | Sankey or alluvial diagram | Flow between categories |
| How does geography matter? | Choropleth or bubble map | Spatial data patterns |
| What's the hierarchy? | Treemap or sunburst | Nested proportions |

### When NOT to use certain charts

- **Pie chart**: Almost never. Use donut (max 5 slices) or bar chart instead. Pie charts are perceptually inaccurate for comparison.
- **3D charts**: Never. 3D distorts perception and adds no information.
- **Donut with >5 segments**: Use a bar chart. Too many slices make donuts unreadable.
- **Dual-axis line chart**: Avoid. Confuses which axis maps to which line. Use two separate charts or index to a common baseline.
- **Bubble chart**: Only when you have 3 meaningful dimensions. Otherwise, scatter plot.
- **Radar/spider chart**: Avoid. Area perception is distorted. Use parallel coordinates or small multiples.

## Chart anatomy

### Axes

**X-axis (horizontal, independent variable)**
```
-- Axis line: none (clean) or 1px solid var(--border)
-- Tick marks: none (preferred) or 4px solid var(--border)
-- Tick labels: var(--text-tertiary), var(--text-xs) (12px), var(--weight-regular)
-- Label rotation: never rotate. If labels don't fit, use fewer ticks or horizontal bar chart.
```

**Y-axis (vertical, dependent variable)**
```
-- Axis line: none (clean) or 1px solid var(--border)
-- Tick marks: none or 4px solid var(--border)
-- Range: always start at zero for bar charts. For line charts, start at meaningful minimum.
-- Tick labels: var(--text-tertiary), var(--text-xs) (12px), aligned right, consistent decimal places
-- Format: compact for large numbers (1.2K, 3.4M). Show units in axis label, not every tick.
```

**Zero baseline rule**: Bar charts MUST start at zero. Line charts can start at a meaningful minimum, but clearly label the Y-axis starting value. Never truncate the Y-axis to exaggerate differences.

### Gridlines

```
-- Horizontal gridlines: yes (help read values across)
-- Vertical gridlines: no (rarely needed, adds clutter)
-- Style: 1px dashed, var(--border) at 30% opacity, or 1px solid at 10% opacity
-- Number: 4-6 horizontal gridlines maximum. More is visual noise.
-- On hover: highlight the relevant gridline (subtle opacity increase)
```

### Data points and lines

```
-- Line chart stroke width: 2px for primary series, 1.5px for secondary
-- Line chart dots: 4px radius, only on hover/tooltip (not always visible)
-- Line chart area fill: primary series at 8% opacity, secondary at 4%
-- Bar chart border-radius: 4px on top corners only (for vertical bars)
-- Bar chart gap: 2-4px between bars, 8-12px between groups
-- Data point highlight on hover: 6px radius with 4px white border + 2px accent border
```

### Legend

```
-- Position: below the chart, horizontal row (preferred), or right side (if many series)
-- Format: colored dot (8px) + label (var(--text-sm)) + optional value
-- Interactive: clicking a legend item toggles series visibility
-- Highlight: hovering a legend item highlights the corresponding series, dims others
-- Maximum series: 8 colors. More than 8 = group or filter.
```

### Tooltip

```
-- Background: var(--bg-secondary) or var(--bg-tertiary)
-- Border: 1px solid var(--border)
-- Border-radius: var(--radius-md) (8px)
-- Shadow: 0 4px 16px rgba(0,0,0,0.12)
-- Padding: var(--space-8) var(--space-12) (8px 12px)
-- Font: var(--text-sm), var(--weight-medium) for values, var(--weight-regular) for labels
-- Layout:
│  Series name           │
│  ● Color dot  Value    │
│  ● Color dot  Value    │
│  Date / Category       │
-- Trigger: appear 150ms after hover, disappear instantly on mouse leave
-- Position: prefer above the data point. Flip below if near top edge.
-- Content: always show the category label and all visible series values
```

### Annotations

Annotations highlight key insights directly on the chart:
```
-- Style: thin line from data point to label, var(--text-secondary) for label
-- Callout: 1-2 sentences max, placed to not overlap data
-- Use for: peaks, valleys, milestones, targets, thresholds
-- Interactive: show on hover if not critical, always visible if critical
```

## Color for data

### Categorical color palette (up to 8 series)

Colors must be distinguishable for all types of color vision:

```
--chart-1: #4F46E5  /* Indigo — Primary, most important series */
--chart-2: #06B6D4  /* Cyan — Distinct from indigo */
--chart-3: #F59E0B  /* Amber — Warm contrast */
--chart-4: #10B981  /* Emerald — Green, safe */
--chart-5: #EC4899  /* Pink — Distinct warm */
--chart-6: #8B5CF6  /* Violet — Similar to indigo, use for pairs */
--chart-7: #F97316  /* Orange — Warm, distinct from amber */
--chart-8: #14B8A6  /* Teal — Cool, distinct from cyan */
```

Rules:
- Series 1 always uses the primary brand color
- Never use red and green alone for positive/negative — add shape or pattern
- Never use two sequential hues that are adjacent (blue + indigo) for adjacent series
- For colorblind safety: pair color with shape (circle, square, triangle, diamond)
- In hover state: highlight the hovered series at full opacity, dim all others to 30% opacity

### Sequential palette (ordered data: low → high)

```
--seq-50:  #EEF2FF  /* Lightest */
--seq-100: #E0E7FF
--seq-200: #C7D2FE
--seq-300: #A5B4FC
--seq-400: #818CF8
--seq-500: #6366F1  /* Middle */
--seq-600: #4F46E5
--seq-700: #4338CA
--seq-800: #3730A3
--seq-900: #312E81  /* Darkest */
--seq-950: #1E1B4B  /* Extreme */
```

Use when: heatmaps, choropleths, intensity grids, risk matrices.
Light shades for low values, dark for high. Ensure end-to-end contrast > 4.5:1.

### Diverging palette (centered data: below/above target)

```
--div-negative: #EF4444  /* Red — below target */
--div-neutral:  #F5F5F5  /* Gray — at target */
--div-positive: #10B981  /* Green — above target */
```

Colorblind-safe alternative:
```
--div-negative: #E67E22  /* Orange — below target */
--div-neutral:  #F5F5F5  /* Gray — at target */
--div-positive: #3B82F6  /* Blue — above target */
```

Use when: variance from budget, sentiment analysis, A/B test results, any data with a meaningful center point.

### Semantic colors for data

```
--data-positive:  #10B981  /* Green — growth, success, on-target */
--data-negative:  #EF4444  /* Red — decline, failure, off-target */
--data-neutral:   #6B7280  /* Gray — no change, paused */
--data-warning:   #F59E0B  /* Amber — approaching threshold */
--data-info:      #3B82F6  /* Blue — informational, no judgment */
```

Always pair semantic color with an icon or shape:
- Positive: ↑ arrow or ● circle
- Negative: ↓ arrow or ■ square
- Neutral: ● circle
- Warning: ⚠ triangle

## Chart types — detailed design rules

### Bar chart

```
Bar width: 24-48px for vertical, auto for horizontal
Bar gap: 2-4px between bars in a group
Group gap: 12-16px between groups
Bar radius: 4px top corners only (vertical), left corners (horizontal)
Hover: slight opacity increase + tooltip
Active: full opacity, others dim to 30%
Negative values: bars extend left of zero line, color with --data-negative
Stacked: total bar label on top of stack if meaningful
```

### Line chart

```
Stroke: 2px (primary series), 1.5px (secondary)
Area fill: 8% opacity (primary), 4% (secondary)
Dots: hidden by default, 4px on hover
Missing data: dashed line connecting known points, or gap with annotation
Multiple series: max 4 before using small multiples
Time axis: daily (≤90 days), weekly (≤2 years), monthly (≤5 years), yearly
```

### Donut chart

```
Maximum segments: 5 (more = use a bar chart)
Stroke width: 60% of radius
Gap between segments: 2px (thin white stroke between)
Center content: total value + label (var(--text-2xl), var(--weight-bold))
Center secondary: percentage or change indicator (var(--text-sm), var(--text-secondary))
Hover: segment expands outward by 4px, tooltip shows value + percentage
Colors: categorical palette, starting at 12 o'clock
```

### Sparkline

```
Use for: inline trend in a table cell, KPI card
Stroke: 1.5px, --color-primary
Area fill: linear gradient from primary (10%) to transparent
Width: 80-120px cell width
Height: 32-40px
No axes, no labels, no tooltip needed (unless clicking shows detail)
Context: paired with the current value number
Positive: green stroke + green area gradient
Negative: red stroke + red area gradient
```

### Small multiples

```
Use when: comparing the same metric across many categories
Layout: grid of 3-4 columns
Each chart: same Y-axis scale, minimal axes (labels on first column/row only)
Highlight: current selection highlighted, others dimmed
Size: each chart 200-300px wide
```

## Dashboard layout

### Information hierarchy

```
┌─────────────────────────────────────────────────────┐
│ Header: Page title + date range filter + actions    │
├─────────────┬─────────────┬─────────────┬──────────┤
│  KPI Card   │  KPI Card   │  KPI Card   │ KPI Card │
│  (1-line)   │  (1-line)    │  (1-line)   │ (1-line) │
├─────────────┴─────────────┴─────────────┴──────────┤
│                                                      │
│           Primary Chart (full width)                  │
│           Line or bar chart                          │
│                                                      │
├────────────────────────┬────────────────────────────┤
│                        │                              │
│   Secondary Chart      │   Tertiary Chart / Table     │
│   Stacked bar / Area    │   Donut / List              │
│                        │                              │
└────────────────────────┴────────────────────────────┘
```

### KPI card design

```
┌──────────────────────┐
│  Label (text-xs,     │
│  text-tertiary)      │
│                      │
│  Value               │
│  (text-2xl, 600)    │
│                      │
│  ↑ 12.5% vs prev    │
│  (text-sm, positive)│
│                      │
│  sparkline ─────╱╲  │
└──────────────────────┘
```

- Height: 120-140px
- Padding: var(--space-16) (16px)
- Background: var(--bg-primary) or var(--bg-secondary)
- Border: 1px solid var(--border)
- Border-radius: var(--radius-lg) (12px)
- Trend indicator: ↑ green for positive, ↓ red for negative, → gray for neutral
- Sparkline: 24px height at bottom of card

### Dashboard density

| Density | Target Users | Card Padding | Grid Gap | Axis Labels | Font Size |
|---------|-------------|-------------|----------|-------------|-----------|
| Compact | Analysts, power users | 12px | 12px | All shown | 11-12px |
| Default | Most users | 16px | 16px | Selected | 12-14px |
| Comfortable | Executives, presentations | 24px | 24px | Key only | 14-16px |

## Responsive charts

### Mobile (< 640px)
- Stack all charts vertically, full width
- KPI cards: 2 per row, not 4
- Reduce axis labels: show every other tick
- Horizontal bar chart instead of vertical bar chart (labels fit better)
- Tooltip becomes a bottom sheet with larger tap target
- Donut stays but center text grows proportionally
- Hide secondary charts behind a "Show more" toggle

### Tablet (640–1024px)
- KPI cards: 2 per row
- Charts: full width or 50/50 split
- Axis labels: show most
- Tooltip: same behavior as desktop

### Desktop (> 1024px)
- Full dashboard layout
- All charts visible
- Hover interactions
- Keyboard navigation between chart elements

## Accessibility for charts

Charts are inherently visual. Make them accessible through multiple channels:

### Screen reader fallback
Every chart must have an accessible alternative:

```html
<figure role="img" aria-label="Revenue increased from $1.2M in Q1 to $1.8M in Q4">
  <svg>...</svg>
  <figcaption class="sr-only">
    Bar chart showing quarterly revenue. Q1: $1.2M, Q2: $1.4M, Q3: $1.6M, Q4: $1.8M.
    Overall trend: 50% increase across the year.
  </figcaption>
</figure>
```

### Keyboard navigation
- Tab into the chart: focuses the chart container
- Arrow keys: navigate between data points
- Enter/Space: show tooltip with full data
- Escape: dismiss tooltip
- Focus indicator: visible ring around the focused data point

### Color alternatives
- Always pair color with pattern or shape
- Patterns for colorblind users: solid, diagonal lines, dots, crosshatch, horizontal lines
- In SVG: use `pattern` elements as fallback fills
- In text: use symbols (● ▲ ■ ◆ ★) alongside colors in legends

### Data table fallback
- Provide a "View as table" button below every chart
- The table contains the exact same data as the chart
- Sortable columns, same filters as the chart

## Chart rendering performance

### Large datasets
- ≤ 1000 points: render in SVG (searchable, accessible)
- 1000–10,000 points: render in Canvas (better performance)
- > 10,000 points: use WebGL (D3FC, deck.gl, or similar)
- Consider downsampling: show daily averages instead of minute-level data for long time ranges

### Animation on load
- Animate chart entry: bars grow from baseline, lines draw from left (300ms, ease-decelerate)
- Stagger bar animation: 20ms per bar
- Animate once on first render, not on every data update (data updates should be immediate)
- Respect `prefers-reduced-motion`: show final state immediately

### Update transitions
- When data changes, transition smoothly (200ms, ease-standard)
- Don't animate data updates of >500ms — it feels laggy
- New data points: fade in (150ms)
- Removed data points: fade out (150ms)

## Quality checklist

- [ ] Chart type matches the data question (comparison → bar, trend → line, proportion → donut)
- [ ] Zero baseline for bar charts
- [ ] Color-blind safe palette with shape/pattern alternatives
- [ ] Axis labels are readable, not rotated, consistent decimal places
- [ ] Gridlines are subtle (10-30% opacity), not distracting
- [ ] Tooltip shows category + all series values
- [ ] Legend is interactive (toggle series visibility)
- [ ] Chart has accessible alternative (aria-label + data table)
- [ ] Keyboard navigation works (arrow keys between data points)
- [ ] Responsive: chart works on mobile with alternative layout
- [ ] Animations respect prefers-reduced-motion
- [ ] No more than 8 colors in a categorical palette
- [ ] Semantic colors (positive/negative) paired with shape indicators
- [ ] Long labels handled (truncation, scrolling, horizontal bars)
- [ ] Empty state: message + CTA, not a broken chart
- [ ] Loading state: skeleton with axis lines, not a spinner

## Anti-patterns I avoid

- Pie charts with >5 slices — use a bar chart or donut
- 3D effects on charts — distort perception and add no value
- Dual Y-axes — confuse readers about which axis maps to which data
- Rainbow color scales — not perceptually uniform, misleading
- Starting bar charts above zero — exaggerates differences
- Truncating axis labels — make them fit or use horizontal bars
- Animating every data update — smooth transitions only for structural changes
- Charts without context — always include title, axis labels, and units
- Using red/green alone for positive/negative — 8% of men are colorblind
- Showing raw numbers without formatting — use K, M, B suffixes, % symbols, currency
- Overlapping series without interaction — dim non-focused series on hover