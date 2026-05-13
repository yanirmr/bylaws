# bylaws — Kibbutz Bylaw Simulator

## Project Overview
Single-page Hebrew (RTL) simulator for a kibbutz bylaw ("תקנון"). Calculates individual member financial entitlements under a proposed settlement scheme. No build step, no framework — all CSS, HTML, and JS in `index.html` (~1020 lines). Depends on Google Fonts and Chart.js 4.4.0 (CDN).

---

## Layout Structure

```
<body>
  .container
    <header>          — Title, subtitle, version note
    .mode-bar         — "Single scenario" / "Compare scenarios" toggle buttons
    #results-strip    — Dynamic results (JS-rendered)
    .chart-container  — Chart.js bar+line chart
    #breakdown-text   — Numeric formula breakdown (JS-rendered)
    #panels           — Three input panels (JS-rendered)
    .disclaimer       — Legal disclaimer footer
```

---

## Three Input Panels

| Panel | Topic | Fields |
|-------|-------|--------|
| **01 Personal** | Member's own data | Seniority years, age, deceased parents/spouse, siblings, home value, standard unit value, debts, credits |
| **02 Kibbutz Reality** | Community financials | Net assets, veteran count, seniority pool, annual profit, community deficit, new members/year, growth rate, discount rate |
| **03 Bylaw Rules** | Tunable policy parameters | Seniority cap, seniority/equality split, orphan/widow bonuses, payment horizon, phase thresholds (t1/t2), phase share percentages |

---

## JS Logic (IIFE, ~540 lines)

```
FIELDS{}        — 27 field definitions (label, type, min/max, tooltip, bylaw source)
PANEL_INFO{}    — Panel metadata
state{A, B}     — Two scenario states (B used in compare mode)

compute(s)      — Core calculation:
  A. Seniority (base + orphan bonus + widow bonus, capped)
  B. Gross right (seniority portion + equal share from net assets)
  C. Personal adjustments (credits − debts + housing gap)
  D. Year-by-year payout loop (3-phase distribution model over `horizon` years)
  → returns grossRight, adjustedRight, memberReceived (NPV), yearlyData[]

renderResults() — Updates #results-strip (single or compare layout)
renderBreakdown()— Updates formula text line
renderChart()   — Destroys+recreates Chart.js instance
renderPanels()  — Rebuilds all input HTML, re-wires events
renderField()   — Renders one field (range slider or select + tooltip)
onFieldChange() — Updates state → re-renders results only (not panels)
```

---

## Two Modes

- **Single**: one set of inputs → one result column
- **Compare**: two side-by-side input columns (A/B) → results show both + delta; B initializes as a copy of A on mode switch

---

## Key Design Patterns

- **No re-render of panels on input change** — only `renderResults()` is called; panels are only rebuilt on mode switch, keeping sliders stable
- **Tooltip toggle** — each field has a `?` button that toggles a hidden `.field-tooltip` div with bylaw citations
- **NPV calculation** — the payout loop discounts future payments by `disc` rate; `memberFraction = adjustedRight / netAssets` determines per-member share
