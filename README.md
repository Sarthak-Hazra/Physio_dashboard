# Physio_dashboard
Dashboard made by HTML for kinetic age
# Physio Performance Dashboard — Kinetic Age

A single-page dashboard for Relationship Managers (RMs) to review weekly physio
performance: a weighted scorecard (operational / clinical / relationship),
a leaderboard, a full metric table, charts, and a prioritized action list.

## What this is

- One self-contained file: `index.html`. HTML, CSS and JavaScript are all
  inline — there is no build step, no bundler, no `npm install`. Open the
  file in a browser and it runs.
- No backend of its own. It calls existing Kinetic Age admin-panel APIs
  directly from the browser using a bearer token pasted in at runtime.
- No secrets are stored in the file. The bearer token lives only in the
  page's input field and in memory for the session — it is never hardcoded
  or persisted.

## Running it locally

1. Open `index.html` in a browser (double-click, or serve it with any
   static file server — e.g. `python3 -m http.server` from this folder).
2. Paste a valid bearer token into the "Bearer token" field in the top bar.
3. Pick a week from the week selector. The dashboard fetches and renders
   from there.

## Where to plug in for backend/admin-panel embedding

The APIs it calls are defined near the bottom of the `<script>` block:

- `API_BASE` — operations analytics endpoints (session details, timing,
  reschedules, utilization, weekly aggregates).
- `TRIAL_DASHBOARD_API` — trial records (paginated).
- `REVIEWS_API` — client reviews / CSAT (paginated).
- `PANEL_SESSIONS_API` — daily session records, queried once per day of the
  selected week.

All of these currently point at `https://www.kineticages.co.in/api/...`. If
this gets embedded inside the admin panel itself, the token field can likely
be replaced with whatever auth context the panel already has, and these
constants are the places to redirect if the routes move.

## Architecture notes for whoever picks this up

- **Central state**: `APP_STATE`, committed only via `commitAppState()`
  after a week's data has fully loaded. Every view (executive cards, KPI
  row, leaderboard, table, charts, action list) reads from this same state,
  so nothing can show a stale value from a previous week.
- **Scoring**: `computeScores()` implements the weighted model — completion
  20%, conversion 15%, CSAT 15%, reschedule 10%, punctuality 10%, SOP 10%,
  report quality 10%, renewal 10%. Thresholds live in `THRESHOLDS`.
- **Reschedule scoring**: `reschScore()` — only trainer-caused reschedules
  penalize a physio; notice time determines minor/major/same-day severity.
- **Pagination**: trial, review, and daily-session APIs are paginated and
  are fully walked before aggregation (see the `liveGetAll*` functions) —
  important not to "optimize" this into a single-page fetch, it will
  silently undercount.

## Known limitations (carried over from the PS report)

- CSAT is attributed via the assigned-physio field on the reviews API —
  there's no dedicated per-physio CSAT endpoint yet.
- The Sessions API's aggregate count has been observed to differ from its
  paginated item count by one in sample data; the dashboard aggregates what
  it actually receives rather than trusting the reported total. Worth a
  backend reconciliation check before this is fully relied on.
- No live renewal endpoint exists yet — renewal is currently pulled from
  the same source as the RM sheet rather than a dedicated API.
- `PENDING_METRICS` in the code flags which metrics (OTP, renewal) are
  still backed by partial/manual data rather than a fully live source.

## Suggested next steps for the tech team

- Decide whether this stays a standalone static page or gets folded into
  the admin panel's existing frontend stack (React/whatever it runs on).
- Replace the manual bearer-token field with the panel's existing session
  auth once embedded.
- Backend reconciliation for the Sessions API count mismatch noted above.
- A dedicated per-physio renewal endpoint, if relationship-score accuracy
  needs to improve further.
