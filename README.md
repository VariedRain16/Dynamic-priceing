[README (1).md](https://github.com/user-attachments/files/29776529/README.1.md)[Uploading # Lisbon Airbnb Pricing Dashboard

A data-driven pricing tool for Airbnb hosts who currently price by instinct. It turns booking
history and occupancy data into clear, explainable pricing recommendations — built to work
with *any* host's data, not just one specific dataset.

## The problem

A Lisbon-based host manages multiple apartments and sets prices mostly by gut feel. They have
booking history and a sense of the market, but no systematic way to turn that into a pricing
strategy — when to charge more, when to discount, and how they stack up against comparable
listings.

## What this app does

- Analyzes booking and occupancy data to find real patterns (seasonality, day-of-week demand,
  lead-time effects)
- Produces a 60-day pricing calendar with plain-language reasoning behind each recommendation
- Presents everything in a dashboard a non-technical host can actually read and act on

## Architecture

The core design principle: **analysis and dashboard code never touch raw data formats
directly.** Everything flows through one canonical schema, so the app works with synthetic
data, this host's real export, or a completely different host's export without any changes
to the logic.

```
Raw data (Airbnb export / PMS export / synthetic)
        │
        ▼
   Adapter layer  ──►  Canonical schema (bookings.csv, availability.csv)
        │
        ▼
  Analysis engine  ──►  JSON output (seasonality, day-of-week, occupancy, pricing calendar)
        │
        ▼
    Dashboard  (reads JSON only, no hardcoded data)
```

### Canonical schema

**Bookings** (one row per reservation):

| Field | Type | Notes |
|---|---|---|
| `listing_id` | string | |
| `listing_name` | string | |
| `check_in` | date | |
| `check_out` | date | |
| `booking_date` | date | date the reservation was made, for lead-time analysis |
| `nights` | int | |
| `nightly_price` | float (EUR) | |
| `total_payout` | float (EUR) | |
| `num_guests` | int | |
| `status` | enum | `completed` \| `cancelled` |

**Availability** (one row per listing/date):

| Field | Type | Notes |
|---|---|---|
| `listing_id` | string | |
| `date` | date | |
| `is_available` | bool | needed for true occupancy rate, not just booked nights |

## Project structure

```
/data
  /raw            # untouched exports as received (gitignored — may contain real host data)
  /processed       # canonical-schema CSVs (bookings.csv, availability.csv)
/adapters          # raw export -> canonical schema converters
/analysis          # seasonality, day-of-week, lead-time, pricing engine (Python)
/dashboard         # frontend, reads analysis output as JSON
```

## Pipeline stages

1. **Synthetic data generator** — produces realistic fake booking/availability data (weekend
   premium, summer high season, holiday spikes, some cancellations) so the pipeline can be
   built and tested before real host data arrives.
2. **Adapter layer** — converts a raw export (Airbnb earnings CSV, PMS export, etc.) into the
   canonical schema. This is the *only* piece that changes per data source.
3. **Analysis engine** — computes occupancy by day-of-week, seasonal trends (with
   decomposition), effective ADR, and lead-time effects. Outputs clean JSON.
4. **Pricing engine** — rule-based multiplier model (not a black box), explainable to a
   non-technical host:
   ```
   recommended_price = base_price
     × seasonal_index(month)
     × day_of_week_multiplier(weekday)
     × lead_time_adjustment(days_until_checkin)
   ```
   Outputs a 60-day price calendar with a short reason tag per date (e.g. "Weekend + summer
   peak").
5. **Dashboard** — occupancy heatmap, seasonality chart, day-of-week chart, and the 60-day
   pricing calendar, filterable by listing.

## Status

- [ ] Synthetic data generator
- [ ] Canonical schema finalized
- [ ] Airbnb export adapter (pending real data from host)
- [ ] Analysis engine
- [ ] Pricing engine
- [ ] Dashboard

## Data privacy note

Real host data (once received) goes in `/data/raw` and `/data/processed`, both gitignored.
Nothing derived from a real host's actual bookings should be committed to this repo.
README (1).md…]()
