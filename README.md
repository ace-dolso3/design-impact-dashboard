# Design Impact Dashboard

A static webpage that maps design releases to before/after Contentsquare metrics. Built for periodic reporting — share a URL, print to PDF, or walk through it in a meeting.

**Status:** Built and working. Data is fictitious. Real data entry and GitHub Pages deployment are pending.

---

## Project Files

```
Design Impact Dashboard/
├── index.html        ← The entire dashboard (HTML + CSS + JS, single file)
├── releases.json     ← Data source — one entry per design release
├── ace-logo.svg      ← Ace Hardware logo (SVG, #D40029)
├── DESIGN.md         ← Original design brainstorm and decision log
└── README.md         ← This file
```

There is no build step. No package.json. No node_modules. It's a single HTML file.

---

## Running Locally

You need a local HTTP server because `fetch('releases.json')` won't work over `file://`.

**Option 1 — Python (built-in, no install needed):**
```bash
cd "path/to/Design Impact Dashboard"
python3 -m http.server 3456
```
Then open: `http://localhost:3456`

**Option 2 — VS Code Live Server extension:**
Install the "Live Server" extension, right-click `index.html` → "Open with Live Server".

**Fallback:** The dashboard also has `releases.json` data embedded inline in `index.html` as the `RELEASES_DATA` constant. If the `fetch` fails (e.g. opening the file directly in a browser), it falls back to this embedded copy automatically. Keep both in sync when you add new releases.

---

## Adding a New Release

This is the only ongoing maintenance task. It takes about 5–10 minutes.

### 1. Add an entry to `releases.json`

Append a new object to the array. Order doesn't matter — the dashboard sorts by date automatically.

```json
{
  "id": "my-release-slug",
  "title": "Short Release Name",
  "date": "2025-09-15",
  "description": "One or two sentences describing what changed and why. This appears below the title in the card.",
  "metrics": [
    {
      "name": "Add-to-Cart Rate",
      "category": "conversion",
      "before": 7.8,
      "after": 10.2,
      "unit": "%"
    },
    {
      "name": "Bounce Rate",
      "category": "engagement",
      "before": 44.3,
      "after": 38.1,
      "unit": "%",
      "lower_is_better": true
    },
    {
      "name": "Revenue per Visitor",
      "category": "revenue",
      "before": 3.22,
      "after": 4.01,
      "unit": "$"
    }
  ]
}
```

### 2. Update the embedded fallback in `index.html`

Search for `const RELEASES_DATA =` (near the bottom of the `<script>` block) and replace its value with the full updated array from `releases.json`. This keeps the dashboard working when opened directly as a file without a server.

### 3. Push to GitHub

```bash
git add releases.json index.html
git commit -m "Add [release name] metrics"
git push
```

GitHub Pages deploys within ~60 seconds.

---

## Data Schema Reference

### Release object

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | string | ✅ | URL-safe slug. Used as HTML element IDs — must be unique. |
| `title` | string | ✅ | Shown in timeline and card header. |
| `date` | string | ✅ | ISO format: `YYYY-MM-DD`. Used for sorting and display. |
| `description` | string | ✅ | Appears below the title in the card. |
| `metrics` | array | ✅ | One or more metric objects (see below). |

### Metric object

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | ✅ | Display label. |
| `category` | string | ✅ | `"conversion"`, `"engagement"`, or `"revenue"`. Currently cosmetic — doesn't affect rendering. |
| `before` | number | ✅ | Baseline value (pre-release). |
| `after` | number | ✅ | Post-release value. |
| `unit` | string | ✅ | `"%"`, `"$"`, `"min"`, etc. Appended to values in the UI. |
| `lower_is_better` | boolean | ❌ | Default `false`. Set to `true` for metrics like Bounce Rate, Cart Abandonment, Load Time — flips the green/red color signal so a decrease shows as an improvement. |

---

## Code Architecture

Everything lives in `index.html`. The file is structured top-to-bottom:

### `<style>` block (lines ~31–225)
All CSS. Key sections:
- **Animations** — `fadeUp`, `fadeIn`, `dotPop` keyframes; `.anim-fade-up`, `.d-1`...`.d-8` delay utilities
- **Timeline** — `.tl-dot`, `.tl-entry`, `.tl-label` (2-line clamp + hover expand), `tlPulse` sonar ring
- **Cards** — `.release-card` hover lift
- **Progress bars** — `.prog-fill` (starts at `width: 0%`, transitions to target on scroll)
- **FAB** — `#scroll-top-fab` fixed position, visible only after scrolling 300px
- **Collapse** — `.card-collapsed` toggles `max-height: 0` for smooth expand/collapse
- **Print** — `@media print` hides interactive elements, forces 2-column grid
- **Reduced motion** — `@media (prefers-reduced-motion)` collapses all animation durations

### `<body>` HTML (lines ~227–330)
Static markup for:
- **Header** — sticky, Ace logo, "Design Impact" title, last-updated date, Export PDF button
- **Summary stats** — 4 stat cards (`#summary-stats`), populated by JS
- **Timeline** — `#timeline` container, populated by JS
- **Release grid** — `#releases-grid`, populated by JS
- **FAB** — `#scroll-top-fab` fixed button
- **Tooltip** — `#chart-tooltip` floating div appended to body (positioned via JS to escape card overflow clipping)

### `<script>` block — JS functions

| Function | What it does |
|---|---|
| `fmtDate(dateStr)` | Formats `"2025-01-14"` → `"January 14, 2025"` |
| `fmtShort(dateStr)` | Formats `"2025-01-14"` → `"Jan 2025"` |
| `delta(metric)` | Computes `{ raw, pct, good, flat }` from a metric object. Respects `lower_is_better`. |
| `renderStats(releases)` | Populates the 4 summary stat cards (release count, metric count, success rate, avg change). |
| `renderTimeline(releases)` | Renders the dot timeline. After insert: marks `tl-expandable` on entries with >2 lines of text; trims connector line to span first→last dot only. |
| `progressBarHtml(m, ri, mi)` | Returns HTML for one metric row's progress bar. Encodes tooltip data in `data-tip`. |
| `renderCards(releases)` | Renders all release cards into `#releases-grid`. Calls `initTooltips()` after DOM settles. |
| `animateBars()` | *(Kept for reference but not called directly.)* Animates all bars at once. |
| `initTooltips()` | Wires `mouseenter`/`mouseleave` on all `[data-tip]` elements to position and show `#chart-tooltip`. |
| `observeCards()` | `IntersectionObserver` on `.release-card` elements. When a card enters the viewport: starts its `fadeUp` animation and animates its progress bars. |
| `toggleCard(id)` | Collapses or expands a single card's metrics section. |
| `toggleAllCards()` | Collapses or expands all cards at once. |
| `updateToggleAllBtn()` | Syncs the "Collapse All / Expand All" button label and icon to current state. |
| `init()` | Entry point. Fetches `releases.json`; falls back to `RELEASES_DATA` if fetch fails. Calls all render functions. |

---

## Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Markup | HTML5 | Single file |
| Styles | Tailwind CSS (CDN) | No install. Config is inline in a `<script>` block. |
| Icons | Material Symbols Outlined (Google Fonts CDN) | Icon names used: `deployed_code`, `monitoring`, `trending_up`, `analytics`, `print`, `arrow_forward`, `expand_less`, `expand_more`, `unfold_less`, `unfold_more`, `keyboard_arrow_up` |
| Font | Inter (Google Fonts CDN) | Weights 300–900 |
| Charts | None (removed) | Replaced with CSS progress bars for print compatibility and simplicity |
| Hosting | GitHub Pages (pending setup) | Free, deploys on push |
| Data collection | Contentsquare MCP (pending admin approval) | Until approved: export manually from Contentsquare UI |

---

## GitHub Pages Setup (not yet done)

1. Create a new GitHub repo named `design-impact-dashboard` (or similar)
2. Push the project files to `main`
3. Go to repo **Settings → Pages**
4. Set source to `main` branch, `/ (root)` folder
5. GitHub will publish the site at `https://<your-username>.github.io/design-impact-dashboard/`

The `releases.json` file will be served from the same origin as `index.html`, so `fetch('releases.json')` will work without the inline fallback. Keep the fallback anyway — it's harmless and useful for local preview.

---

## Known Issues / Pending Items

- **Badge arrow direction bug:** On `lower_is_better` metrics where a decrease is *good* (e.g. Cart Abandonment Rate going from 46.8% → 39%), the badge shows `▲ -7.8%` — the upward arrow contradicts the negative number. It should show `▼ -7.8%` (a decrease is the improvement). The `delta()` function logic is correct; the badge arrow rendering just needs to use the sign of `d.raw` rather than `d.good` to pick the arrow character.

- **Contentsquare MCP access** — admin approval pending. In the meantime, export CSVs manually from the Contentsquare UI and enter values by hand into `releases.json`.

- **All data is fictitious** — the 6 releases in `releases.json` (Jan–Aug 2025) are placeholder data. Replace with real release data before sharing externally.

- **GitHub repo not yet created** — the project lives only on OneDrive. See GitHub Pages Setup above.

---

## Updating with Copilot in VS Code (recommended workflow)

Once you have the Contentsquare MCP connected in VS Code (agent mode), the update cycle is:

1. Open the project folder in VS Code
2. In Copilot Chat (agent mode), ask: *"Pull before/after metrics for [release name] from Contentsquare for the date range [X to Y] and format them as a `releases.json` entry."*
3. Paste the output into `releases.json` and update the `RELEASES_DATA` fallback in `index.html`
4. Preview locally, then `git push`

For code changes, Copilot can read and edit `index.html` directly. The single-file architecture means there's no build step to worry about — edit and refresh.
