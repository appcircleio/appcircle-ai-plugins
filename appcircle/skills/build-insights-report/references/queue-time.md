# Section 5: Queue Time

Reads from: `data.sections.queue_time`. No computation. Section header:
`QUEUE TIME · [DATE RANGE]`. This section aggregates how long builds waited in
queue before starting (always tree-wide, regardless of `include_sub_orgs`).

---

## Data Source Map

| Rendered element | Response path |
|---|---|
| Stat cells | `queue_time.{total_wait_minutes, avg_wait_minutes, p50_wait_minutes, p95_wait_minutes, build_count}` |
| Daily trend chart | `queue_time.daily_trend[]` → `{date, avg_wait_minutes}` |

Reading notes:
- All wait values are minutes. Render `Xm` if < 60, else `X.Xh`.
- `build_count` is the number of builds that contributed wait data — show it as
  context, not as a primary metric.
- `daily_trend` contains only days that had queued builds (it is sparse, not a full
  calendar). Plot the points as given; do not back-fill missing days.
- The section is omitted entirely (listed in `meta.omitted_sections`) when no queue
  waiting data is available for the period — in that case render nothing.

---

## Render: Queue Time Widget

Section header: `QUEUE TIME · [DATE RANGE]`. Uses the Global Design System.

### Stat cells

A row of four cells, same card style as the Health Snapshot summary cards (muted
uppercase label, large bold value), top accent `#1A3352` (navy) 3px:
- **Avg wait** → `avg_wait_minutes`
- **P50 wait** → `p50_wait_minutes`
- **P95 wait** → `p95_wait_minutes`
- **Total wait** → `total_wait_minutes`

Below the row, a muted 12px caption: "Across N builds with queue data" (from
`build_count`).

Colour the P95 value by band so a long tail is visible at a glance: green if
`p95_wait_minutes` ≤ 5, amber if ≤ 15, red if > 15 (minutes).

### Daily trend chart

Only render if `daily_trend` has at least two points. Type: line, single dataset of
`avg_wait_minutes` against `date`, navy `#1A3352`, point radius 3px, tension 0.3,
`spanGaps: true` (the series is sparse by design). Y-axis label minutes (e.g. `2m`).
Subtitle: "Average queue wait per day." Canvas height 180px. Same Chart.js setup and
technical requirements as the Trends section (single `DOMContentLoaded`, grid
`rgba(0,0,0,0.06)`, ticks `#9CA3AF` 10px, `maintainAspectRatio: false`).

If `daily_trend` has fewer than two points, skip the chart and keep just the stat
cells with the caption.

---

## Recommendation card

When queue wait is long or trending up, render a recommendation box below the
chart, styled to match Appcircle's in-product upgrade nudge: a light orange card
(`.rec`) with an uppercase "RECOMMENDATION" label, a bold navy headline, body
copy explaining the situation, and one or two pill-outline buttons.

The headline and body are authored by you before calling `render.py`. Only the
gate (whether the card appears at all) and the buttons are computed by
`render.py` itself from `_queue_recommendation_gate(qt)` — those are deterministic
display rules, not narrative, so they stay in the script:

- **Trigger.** The gate shows when `p95_wait_minutes >= 3`, or when the trend is
  climbing and P95 is at least 1.5 minutes. Otherwise queue health is fine and no
  card renders — this is not a permanent fixture of the section. Compute this
  same condition yourself before deciding whether to author `ai_recommendation`;
  if the gate would be true but you leave the key out, the card silently does not
  render (no fallback wording is generated).
- **What to author.** When the trigger condition is met, set
  `queue_time.ai_recommendation` to `{"headline": "...", "body": "..."}` in the
  envelope. Ground both in the actual numbers:
  - **Trend.** Derive from `daily_trend`: compare the first and last point. A
    >25% increase is "climbing" (most urgent framing — capacity may not be
    keeping up); >25% decrease is "improving" (elevated but easing); otherwise
    "stable". Fewer than two points means no trend claim can be made.
  - **Plan-aware framing.** Appcircle's machine plans, lowest to highest, are
    Standard → Velocity → Ultra. The tool attributes queue wait to each tier via
    `queue_time.plan_breakdown`, e.g. `{"standard_wait_minutes": 6.0,
    "velocity_wait_minutes": 0.0, "ultra_wait_minutes": 0.0}` (a flat
    `queue_time.plan` string, if present, names the tier directly and takes
    precedence over the breakdown). Name whichever tier absorbed the most wait
    this period — that's the one an upgrade would help most — and, if more than
    one tier saw real activity, say "most of your queue wait" rather than
    implying it's the org's only tier. Reference the P95 wait value and the next
    plan tier up in the body text. With no plan information at all, write
    plan-agnostic copy about concurrency and machine plan upgrades generally.
- **Breakdown caption.** Whenever `plan_breakdown` is present, `render.py`
  already shows a small caption under the build-count line with the raw split,
  e.g. "By plan: Standard 6m · Velocity 0m · Ultra 0m" — so the inference behind
  your recommendation is visible, not just asserted. You don't need to restate
  this in the authored body.
- **Buttons.** Fixed, not authored: `render.py` always shows "Increase
  concurrency" (links to `https://appcircle.io/contact`), plus "Upgrade to
  {next plan}" or "View plans" (same URL) unless the dominant tier is already
  Ultra, in which case only the first button shows.

If the tool's `plan_breakdown` keys or shape change, update `_BREAKDOWN_KEYS` and
`_plan_from_breakdown()` in `render.py` (and this table) to match.
