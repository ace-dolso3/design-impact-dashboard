# Design System — Design Impact Dashboard

This file is read automatically by Claude Code at the start of every session.
Apply these conventions to all UI changes without needing to be asked.

---

## Brand Colors

| Token | Hex | Usage |
|---|---|---|
| `ace-red` | `#D40029` | Primary — CTAs, badges, timeline dots, left card border, FAB, icons |
| `ace-red-dk` | `#B00022` | Hover state for ace-red elements |
| `ace-red-lt` | `#FEF0F0` | Subtle red tint backgrounds |

Never introduce new brand colors. Ace red is the only accent. All other UI uses the Tailwind slate scale.

## Slate Scale Usage

- `slate-900` — primary text, large numbers
- `slate-700` — secondary text, labels
- `slate-500` — muted text, before-values
- `slate-400` — timestamps, placeholders, icon tints
- `slate-200` — borders, dividers, progress bar tracks
- `slate-100` — card borders, light backgrounds
- `slate-50 / #F8FAFC` — page background

## Semantic Colors (metrics only)

- **Improvement:** `#16A34A` (green-600) fill, `#4ADE80` (green-400) tooltip tint
- **Regression:** `#DC2626` (red-600) fill, `#FCA5A5` (red-400) tooltip tint
- **Flat / no change:** `#94A3B8` (slate-400)

---

## Typography

**Font:** Inter (Google Fonts CDN), weights 300–900
**Scale conventions:**
- Section labels: `text-[10px] font-semibold uppercase tracking-[.12em] text-ace-red`
- Card titles: `text-xl font-bold text-slate-900`
- Body/description: `text-sm text-slate-500 leading-relaxed`
- Metric names: `text-sm font-semibold text-slate-800`
- Stat values: `text-4xl font-black text-slate-900 leading-none`
- Timeline labels: `text-[10px] font-semibold text-slate-700`
- Timestamps: `text-[9px] font-medium tracking-wide text-slate-400`

---

## Icons

**Library:** Material Symbols Outlined (Google Fonts CDN)
**Usage:** `<span class="material-symbols-outlined">icon_name</span>`
**Do not** use other icon libraries (Heroicons, FontAwesome, etc.).

Icons in use: `deployed_code`, `monitoring`, `trending_up`, `analytics`, `print`, `arrow_forward`, `expand_less`, `expand_more`, `unfold_less`, `unfold_more`, `keyboard_arrow_up`

---

## Layout & Spacing

- Max content width: `max-w-7xl mx-auto px-6`
- Release grid: `grid grid-cols-1 xl:grid-cols-2 gap-6`
- Card padding: `px-6 py-5` (header), `p-5` (metrics body)
- Metric rows: `rounded-xl p-4` with `space-y-3` between rows
- Section spacing: `mb-10` between major sections

---

## Component Conventions

### Cards
- `bg-white rounded-2xl border border-slate-100 shadow-sm`
- Full-height left red border: `absolute left-0 top-0 bottom-0 w-1 bg-ace-red rounded-l-2xl` on a `relative` parent
- Hover lift: `transform: translateY(-4px)` + elevated `box-shadow` (`.release-card:hover`)
- Collapsible metrics via `max-height` toggle + `.card-collapsed` class

### Progress Bars
- Track: `h-[6px] bg-slate-200 rounded-full`
- Fill: `.prog-fill` — starts `width: 0%`, animates to target when card scrolls into view via `IntersectionObserver`
- Before-value marker: `w-[2px] h-[13px] bg-slate-400` absolutely positioned at `left: {beforePct}%`
- Tooltip on hover via shared `#chart-tooltip` div appended to `<body>` (escapes overflow clipping)

### Badges
- `text-[11px] font-bold px-2.5 py-1 rounded-full`
- `.badge-pos` (green bg/text) for improvements, `.badge-neg` (red) for regressions

### Timeline
- Dots: `w-3 h-3 rounded-full bg-ace-red`, white ring via `box-shadow: 0 0 0 2px white`
- Hover: dot scales to `1.9×`, `::before` pseudo-element animates `tlPulse` sonar ring (scale + fade, infinite)
- Labels: 2-line clamp (`-webkit-line-clamp: 2`), hover-expand only if text actually overflows (`tl-expandable` class set by JS)
- Connector line trimmed to span first dot → last dot only (set by JS after render)
- Date appears **above** the title in each timeline entry

### Header
- Sticky, `bg-white/90 backdrop-blur border-b border-slate-100 z-50`
- Height: `h-16`
- Ace SVG logo (166×74, fill `#D40029`) + "Design Impact" wordmark

### FAB (scroll-to-top)
- Fixed `bottom-6 right-6`, visible only when `scrollY > 300`
- `bg-ace-red`, rounded-2xl, 48×48px

---

## Animation Principles

- **Easing:** `cubic-bezier(.22,.68,0,1.2)` for springy/poppy motion; `ease-out` for fades
- **Scroll-triggered:** Cards use `IntersectionObserver` (`threshold: 0.08`) — `animationPlayState` starts paused, runs on enter. Progress bars animate at the same moment.
- **Stagger delays:** `.d-1` through `.d-8` (0.05s → 0.60s) for sequential load-in
- **Reduced motion:** `@media (prefers-reduced-motion: reduce)` collapses all durations to `.01ms`
- Keep animations **subtle** — they should feel responsive, not showy. No looping animations except the timeline dot pulse (which only runs on hover).

---

## Print / Export

- `@media print`: hide `.no-print` elements, force 2-column grid, remove sticky header, white background
- "Export PDF" button triggers `window.print()`
- Cards use `break-inside: avoid` for clean page breaks

---

## What NOT to Do

- Do not introduce new CSS frameworks, JS libraries, or npm packages
- Do not split into multiple files — everything stays in `index.html` (+ `releases.json` for data)
- Do not add Chart.js or any canvas-based charting — progress bars are intentional
- Do not use inline `onclick` for new interactions — follow the existing pattern of named JS functions called from `onclick` attributes
- Do not add features that require a server, database, or build step
- Do not change the data schema in `releases.json` without also updating the `RELEASES_DATA` fallback in `index.html`
