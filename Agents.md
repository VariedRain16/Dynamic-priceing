[AGENTS (1).md](https://github.com/user-attachments/files/29776536/AGENTS.1.md)
# AGENTS.md

Guidance for AI coding agents (Antigravity, Claude Code, Cursor, etc.) working in this repo.

## Project summary

A pricing dashboard for Airbnb hosts. Raw booking data comes in through an adapter, gets
converted into a canonical schema, gets analyzed for patterns, and produces a 60-day pricing
recommendation calendar in a dashboard.

## Non-negotiable architecture rule

**Analysis and dashboard code must never assume a specific data source.** They only ever read
from the canonical schema (see `/data/processed/bookings.csv` and `availability.csv`).

If you're adding a feature that touches raw export formats, listing-specific values, currency
assumptions, or a specific city — stop. That logic belongs in `/adapters`, not in
`/analysis` or `/dashboard`. This app needs to work for any host's data, not just the one
sample dataset it launched with.

## Canonical schema (do not change without updating all adapters + analysis code)

**bookings.csv**
```
listing_id, listing_name, check_in, check_out, booking_date, nights,
nightly_price, total_payout, num_guests, status
```
`status` is one of `completed` or `cancelled`. Dates are ISO 8601 (`YYYY-MM-DD`).

**availability.csv**
```
listing_id, date, is_available
```

Occupancy rate must always be computed as `booked_nights / available_nights`, using
`availability.csv` — never approximate it from bookings alone.

## Directory conventions

- `/data/raw` — untouched exports exactly as received. Never edit files here. Gitignored if
  they contain real host data.
- `/data/processed` — canonical-schema CSVs only. This is the sole input to `/analysis`.
- `/adapters` — one file per data source (e.g. `airbnb_earnings_adapter.py`,
  `synthetic_generator.py`, `pms_export_adapter.py`). Each adapter's only job: raw format in,
  canonical schema out. Keep these isolated and independently testable.
- `/analysis` — pandas/statsmodels code. Reads only from `/data/processed`. Outputs JSON to a
  fixed location the dashboard reads from (don't let the dashboard read CSVs directly).
- `/dashboard` — frontend. Reads only the analysis engine's JSON output. No hardcoded listing
  names, prices, or dates anywhere in dashboard code.

## Pricing engine rules

- Keep it rule-based and explainable (`base_price × seasonal_index × day_of_week_multiplier ×
  lead_time_adjustment`), not a black-box ML model. The host needs to trust and understand the
  output, not just receive a number.
- Every multiplier must be derived from the actual historical data passed in — never hardcode
  multiplier values as constants. If the underlying data changes (new host, more history), the
  multipliers should recompute accordingly.
- Every recommended price in the output calendar needs a short plain-language reason tag (e.g.
  `"Weekend + summer peak"`), not just a number.

## Working with real host data

Real data may be added to `/data/raw` and `/data/processed` later. Treat anything in those
folders as sensitive:
- Never commit real host data to the repo, log it verbatim, or include raw values in commit
  messages/PR descriptions.
- Synthetic data (from `/adapters/synthetic_generator.py`) is safe to use for development,
  tests, and demos — prefer it unless a task explicitly requires real data.

## Before submitting changes

- If you touched `/analysis` or `/dashboard`, confirm the change works against the synthetic
  dataset, not just against one specific data source.
- If you touched the canonical schema, update every file in `/adapters` and note the change in
  README.md.
- Don't introduce a frontend framework or dependency not already in use without flagging it —
  keep the stack minimal and consistent.
