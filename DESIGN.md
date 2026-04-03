# Design Impact Dashboard — Design Document

**Date:** 2026-03-28
**Status:** Design Complete — Ready for Implementation

---

## Understanding Summary

- **What:** A static web dashboard mapping design releases to before/after Contentsquare metrics, hosted at a permanent GitHub Pages URL
- **Why:** Make design impact visible and credible to cross-functional stakeholders — proving ROI without manual data wrangling after every release
- **Who:** Cross-functional stakeholders (product, engineering, marketing) who consult it independently and reference it in meetings; one person generates/updates it periodically
- **Input source:** Azure DevOps release log (structured, clear descriptions and dates)
- **Data source:** Contentsquare via MCP server (connected to VS Code + GitHub Copilot agent mode)
- **Metrics tracked:** Conversion (add-to-cart, checkout, CTR), engagement (bounce rate, time on page, scroll depth), revenue (AOV, revenue per visitor)
- **Constraints:** No budget, Copilot generates all code, low maintenance, must be shareable and print/export ready

## Explicit Non-Goals

- Not real-time / live-connected to Contentsquare
- Not a code-heavy custom application
- Not built for daily use — periodic reporting tool (~monthly)

---

## Assumptions

- Contentsquare MCP access will be approved (admin request pending)
- In the meantime, metrics can be exported manually from Contentsquare UI as a bridge
- GitHub account is available (work account confirmed)
- GitHub Copilot agent mode supports Contentsquare MCP in VS Code
- A JSON data file updated each cycle is an acceptable data layer
- Browser print-to-PDF is sufficient for slide exports
- 2–4 weeks post-launch data window is enough to measure impact
- Azure DevOps release entries include clear descriptions and dates

---

## Architecture

Three layers:

### Layer 1 — Data Collection (per reporting cycle)
VS Code + GitHub Copilot (agent mode) + Contentsquare MCP. After a release settles (~2–4 weeks), query Contentsquare via natural language. Copilot outputs a structured JSON snippet appended to `releases.json`.

### Layer 2 — The Dashboard (built once)
Static webpage — HTML, CSS, and lightweight JavaScript. No framework, no build pipeline. Reads `releases.json` at load time and renders the UI. Copilot generates and maintains the codebase.

### Layer 3 — Hosting (GitHub Pages)
Files live in a GitHub repo. GitHub Pages auto-deploys on push. Stakeholders bookmark one permanent URL.

---

## UI Design

**1. Header**
Ace Hardware branding, dashboard title ("Design Impact"), last-updated date from JSON.

**2. Release Timeline**
Chronological timeline of all tracked releases. Each node is clickable — scrolls to or expands its detail card.

**3. Before/After Impact Cards** (one per release)
- Release name, date, and description
- Metric rows: before value, after value, delta
- Color signal: green = improvement, red = regression (respects `lower_is_better` flag)
- Inline Chart.js grouped bar charts per metric (before vs. after)

**4. Print / Export Bar**
Sticky print-to-PDF button, pre-styled for clean slide-ready output.

---

## Data Structure

`releases.json` — single file, array of release objects:

```json
[
  {
    "id": "pdp-redesign-march-2025",
    "title": "PDP Redesign",
    "date": "2025-03-01",
    "description": "Redesigned the product detail page layout to reduce friction and improve add-to-cart visibility.",
    "metrics": [
      {
        "name": "Add-to-Cart Rate",
        "category": "conversion",
        "before": 8.2,
        "after": 10.4,
        "unit": "%"
      },
      {
        "name": "Bounce Rate",
        "category": "engagement",
        "before": 42.1,
        "after": 36.8,
        "unit": "%",
        "lower_is_better": true
      },
      {
        "name": "Revenue per Visitor",
        "category": "revenue",
        "before": 3.14,
        "after": 3.67,
        "unit": "$"
      }
    ]
  }
]
```

All metric fields are optional — missing metrics simply don't render.

---

## Update Workflow (per reporting cycle, ~20–30 min)

1. **Wait** 2–4 weeks post-launch for data to settle
2. **Query** Contentsquare in VS Code Copilot agent mode — ask for before/after metrics for the release window, request output as a `releases.json` entry
3. **Paste** the new JSON entry into `releases.json`
4. **Push** to GitHub — Pages deploys automatically within ~60 seconds
5. **Share** the URL or print to PDF for a deck

---

## Edge Cases

| Scenario | Resolution |
|---|---|
| Metric not available for a release | Field is optional — card skips missing metrics, no broken UI |
| Negative or inconclusive results | Displayed honestly with red delta — builds long-term credibility |
| MCP access not yet approved | Export manually from Contentsquare UI, enter data by hand — same JSON shape |
| Multiple releases in same period | Note confounding factors in the `description` field |

---

## Decision Log

| Decision | Alternatives Considered | Why |
|---|---|---|
| Static webpage over Excel/Notion | Excel on SharePoint, Notion database | More presentable, consultable in meetings, shareable URL |
| GitHub Pages for hosting | Azure Static Web Apps, SharePoint | Simpler setup, already have GitHub account, equally free |
| JSON file as data layer | Live API connection, database | No server needed, Copilot can generate updates, zero maintenance |
| Chart.js for charts | D3.js, Recharts, no charts | Lightweight, no install, Copilot fluent, prints cleanly |
| Manual deploy (push to GitHub) | CI/CD pipeline | Sufficient for periodic reporting cadence, no overhead |
| MCP as data collection layer | REST API, manual CSV export | No code required, natural language queries, already in VS Code |
| `lower_is_better` flag on metrics | Hardcoded metric list | Flexible — handles bounce rate, load time, and future metrics |

---

## Feature: Revenue Impact Ranking

**Status:** Design complete — ready for implementation

### Summary
Three interconnected UI components that surface which design releases drove the most revenue impact, helping stakeholders orient quickly without reading every card.

### Components

**1. Revenue Impact Ranking leaderboard** (new section between timeline and release cards)
- Labeled "REVENUE IMPACT RANKING" in the standard section header treatment
- Single white card containing a horizontal strip of all releases, ordered highest → lowest score
- Each entry: rank number pill, release title, revenue score (green/red), vertical divider between entries
- Releases with no revenue metrics show N/A in slate-400
- Clicking an entry scrolls to its release card

**2. Ranked badges on cards** (#1, #2, #3 only)
- Placed top-right of the card header alongside the existing date label
- `#1` — ace-red background, white text
- `#2` — slate-700 background, white text
- `#3` — slate-400 background, white text
- Label format: `#1 Revenue`, `#2 Revenue`, `#3 Revenue`

**3. Sort toggle** (in the "RELEASE IMPACT" section header)
- Two-option button group: "Chronological" (default) | "Revenue Impact"
- Positioned left of the existing "Collapse All" button
- Active option uses ace-red treatment; inactive is slate-400
- Toggling re-renders the card grid with a smooth fade transition

### Revenue Impact Score Formula
Average of signed percentage changes across all `category: "revenue"` metrics per release.
The `lower_is_better` flag flips the sign so decreases count as positive impact.
Releases with no revenue metrics score null and sort to the bottom.

### Decision Log

| Decision | Alternatives Considered | Why |
|---|---|---|
| Revenue-only scoring | All metrics, weighted score, best single metric | Revenue is most convincing signal for cross-functional stakeholders |
| Average signed % change | Sum of changes, single best metric | Simple to explain, equal weighting, already respects `lower_is_better` |
| Leaderboard between timeline and cards | Top of page, inside stats row, sidebar | Creates natural "what shipped → what mattered → full detail" narrative |
| Compact horizontal strip | Podium treatment, data table | Matches timeline's horizontal rhythm, stays compact, reads quickly |
| Top 3 badges only | Badge all releases, tiered labels | Clean signal without over-labeling; instantly legible |
| Chronological as default sort | Revenue ranking as default | Preserves existing narrative; ranking is an on-demand view |
| Sort toggle in section header | Separate filter bar, dropdown | Consistent with existing "Collapse All" control placement |

---

## Feature: Leak Finder (Pre-Design Planning)

**Date:** 2026-04-02  
**Status:** Brainstorming complete, design validated, implementation not started

### Understanding Summary

- Build a self-serve way for Product and Design leads to answer: "Where is this page leaking users before we start design work?"
- Keep the first version focused on page-level diagnostics, not full journey analysis.
- Default page set is PLP, PDP, Cart, and Checkout.
- Natural-language querying is a future direction, but V1 should avoid AI/runtime integration.
- V1 interaction is a guided form that returns a standardized leak report.
- Data refresh cadence is weekly, with one owner handling updates and QA.
- The system stays internal-only, aggregated only, low-scale, and best-effort for reliability.

### Explicit Non-Goals

- No free-form natural-language query interface in V1.
- No customer-level or personally identifiable data.
- No requirement for real-time analytics ingestion.
- No attempt to replace full analytics platforms.

### Assumptions

- Internal dashboard users are approximately 5-15 weekly active users.
- Response times up to 30 seconds are acceptable if results are richer and actionable.
- Users prefer consistent, explainable outputs over flexible but complex query tooling.
- Weekly snapshot data is sufficient to support pre-design planning decisions.

### Design Approaches Evaluated

1. Guided Leak Finder playbooks (recommended)
2. Flexible question builder
3. Request-to-insight queue (manual human-in-the-loop)

Recommended path: start with playbooks for speed, clarity, and low maintenance; then add selective flexibility only if demand proves it.

### V1 Architecture

Three layers:

1. Data Prep Layer (weekly, owner-operated)
- Curated snapshots for PLP/PDP/Cart/Checkout with current values, comparison values, and deltas.

2. Insight Engine Layer (in-browser deterministic rules)
- Computes leakage severity and ranks contributing signals.
- No model inference required.

3. Guided Interaction Layer (self-serve)
- Inputs: page, date range, optional segment, comparison mode.
- Output: standardized leak report with status, top drivers, and next investigation prompts.

### Leak Report Output Contract

Each result should always include:

1. Status: Healthy, Watch, or At Risk
2. Leak score and key metric deltas
3. Top 3 contributing signals
4. Suggested investigation prompts for design discovery
5. Confidence indicator with caveats

### Scoring and Thresholds

Initial rule-based score:

LeakScore = wb * DeltaBounce + we * DeltaExit + wc * (-DeltaConversion)

Initial weights:

- wb = 0.35
- we = 0.35
- wc = 0.30

Initial severity bands:

- Healthy: score < 0.20
- Watch: score 0.20-0.49
- At Risk: score >= 0.50

### Reliability and Error Handling

- Validate all inputs before execution.
- If data volume is too low, return "insufficient evidence" instead of forcing a decision.
- Flag known instrumentation changes and reduce confidence accordingly.
- If selected segment data is unavailable, degrade to aggregate and disclose the fallback.
- Every failure state must explain cause and next step.

### Non-Functional Requirements

- Performance: rich answer acceptable up to 30 seconds.
- Scale: 5-15 weekly internal users.
- Privacy/security: internal-only, aggregated data only.
- Availability: best-effort internal tool.
- Maintainability: weekly refresh workflow with one owner.

### Testing Strategy

1. Rule correctness tests
- Validate sign handling for lower-is-better metrics.
- Validate threshold band classification at boundaries.

2. UX trust tests
- Confirm all report states render with explanation and confidence details.
- Confirm caveat banners appear for low-data and instrumentation-change scenarios.

3. Workflow reliability tests
- Simulate stale or partial weekly snapshots and verify graceful degradation.
- Verify deterministic repeatability for identical inputs.

4. User acceptance checks (Product + Design leads)
- Can users independently answer leak-location questions for default pages?
- Do suggested prompts produce clear next design-investigation steps?

### Decision Log

| Decision | Alternatives Considered | Why |
|---|---|---|
| Primary expansion use case is pre-design planning | Post-release evaluation, experiment governance, portfolio steering | Matches immediate team need to decide what to design next |
| Audience is Product + Design leads | Product-only, broader org access | Balances usefulness and governance for V1 |
| Focus question is leak location | Trend-only, segment-only, hypothesis-only | Most actionable first question before design work |
| Page-level scope for V1 | Flow-level first, hybrid start | Lowest complexity with fastest path to self-serve value |
| Default pages PLP/PDP/Cart/Checkout | Custom per-quarter list | Covers key ecommerce decision surfaces |
| Guided form for V1 | Natural language, templates-only, manual queue | Best fit without AI integration and still self-serve |
| Weekly refresh cadence | Daily, near-real-time | Sustainable with single-owner operations |
| Deterministic scoring engine | Heuristic-only labels, ML scoring | Transparent, explainable, maintainable |
| Internal aggregated data only | Deeper user-level detail | Aligns with privacy constraints and lower risk |

### Risks and Mitigations

- Risk: score becomes over-trusted despite data quality gaps.
  Mitigation: confidence labeling and explicit caveats are mandatory.

- Risk: single-owner refresh process becomes a bottleneck.
  Mitigation: keep data schema constrained and process checklist-driven.

- Risk: users request open-ended questions quickly.
  Mitigation: plan a V2 path to controlled flexibility after usage evidence.

### Future Expansion Use Cases

Potential next use cases after Leak Finder stabilizes:

1. Trend regression guardrails by page over rolling windows.
2. Segment-aware leak diagnosis for device and traffic cohorts.
3. Hypothesis queue generation from repeated leak patterns.
4. Pre-design brief generator linking leaks to design investigation prompts.
5. Portfolio prioritization that combines leak severity with opportunity size.

### Exit Criteria Status for Brainstorming

- Understanding lock confirmed: yes
- At least one approach explicitly accepted: yes
- Major assumptions documented: yes
- Key risks acknowledged: yes
- Decision log complete: yes
