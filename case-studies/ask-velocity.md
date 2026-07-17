# Ask Velocity — AI Demand Platform for Private Aviation (FBOs)

> A multi-tenant SaaS that helps Fixed-Base Operators (FBOs) grow hangar leasing
> and fuel sales — automatically ingesting private-jet flight activity, surfacing
> and enriching high-value prospects, running personalized email/SMS outreach,
> and giving operators a live view of demand and pipeline.

<table>
<tr><td><b>Live</b></td><td><a href="https://askvelocity.ai">askvelocity.ai</a></td></tr>
<tr><td><b>Role</b></td><td>Architect / Full-Stack &amp; AI Engineer</td></tr>
<tr><td><b>Domain</b></td><td>Private aviation · FBO demand generation</td></tr>
<tr><td><b>Core stack</b></td><td>Python · FastAPI · asyncpg · React · Material&nbsp;UI · Leaflet · PostgreSQL/PostGIS · Google Cloud Run</td></tr>
</table>

---

## Overview

FBOs make money two ways — leasing hangar space and selling jet fuel — and both
depend on knowing **who is flying into their airports**, who owns and operates
those aircraft, and how to reach them. That intelligence is normally assembled by
hand, per airport, every day.

Ask Velocity turns that manual grind into an automated demand engine, and runs it
for **many FBO clients at once** from one shared platform with strict per-client
data isolation.

---

## Architecture

A **hub-and-spoke, multi-tenant** design: one shared data/ingestion backbone,
isolated per-client services and schemas.

```
   Per-client React UIs (branded build each · maps · dashboards)
                          │
   Per-client FastAPI services  ──  one Postgres SCHEMA per client
                          │              (structural isolation)
                          ▼
   ┌─────────────────────────────────────────────────────────┐
   │  Aviation Shared Service (single codebase)               │
   │  • flight-data ingestion  • cross-client daily digest    │
   │  • authenticated flight-read API (per-airport API keys)  │
   └───────────────┬───────────────────────┬─────────────────┘
        aviation flight-data API      HubSpot · OpenSky · SendGrid · Twilio
```

- **Structural tenant isolation** — each client runs as its own Cloud Run service
  backed by a dedicated Postgres **schema**; every query is schema-qualified, so
  cross-tenant leakage is impossible by construction rather than by careful
  `WHERE` clauses.
- **Canonical + ports** — one reference service holds the feature logic; each
  client service is a port that differs only by schema and branding, so a fix
  ships everywhere.
- **Shared backbone** — the compute-heavy, cross-cutting work (flight ingestion,
  digests, the authenticated flight-read API) lives once in the shared service,
  keyed by a tenant registry.

---

## Key Features

| Area | What it does |
| :-- | :-- |
| 🛬 **Flight-data ingestion** | Daily sync of private-jet arrivals/departures from a third-party aviation flight-data API into a shared `flight_history` store; airports are added via a config table — no redeploy |
| 🔎 **Prospect discovery** | Resolves a tail number to reachable contacts by aggregating five sources (owner/operator/company emails + office phones + user-added contacts), deduped and flagged for email/SMS reachability |
| 📊 **Recurrence & demand analytics** | Surfaces recurring ("transient") aircraft ranked by visit frequency and top home bases per airport — who to prioritize |
| ✉️ **AI email/SMS outreach** | Personalized campaigns dispatched through a shared email/SMS backbone (SendGrid + Twilio), batched to scale to thousands of recipients |
| 🗺️ **Dashboards & live radar** | Leaflet maps of partner-airport flows, plus a live air-traffic radar and traffic heatmaps via the OpenSky Network |
| 🏢 **Hangar-operator analytics** | For real-estate-focused clients, a live HubSpot deal sync powers occupancy, lease-health, and revenue dashboards |

---

## Engineering Deep-Dives

### 1. Resilient third-party flight ingestion
The ingestion service caches the upstream API token with a short TTL and fetches
credentials from **Secret Manager at runtime**, so the upstream password can be
rotated with **zero downtime** (a 401 simply triggers re-auth). Each airport
syncs on a **7-day rolling window** guarded by a **14-day safety gate**: if a
sync falls too far behind (e.g., an outage), it halts loudly instead of silently
importing stale data. A **resumable backfill** recovers month-by-month, tracking
how far each airport has been fetched. New airports are onboarded by inserting a
row into a `tracked_airports` config table — the cron picks it up with no deploy.

### 2. Multi-source contact aggregation
Turning a tail number into someone you can actually email is a five-source
`UNION` (user-added contacts + owner/operator/company emails + office phones),
normalized and **deduped by `(email, phone)`**, US-filtered, and tagged with
`has_email` / `has_sms` reachability flags. Lookups chunk tail numbers (500 per
query) to stay within Postgres parameter limits and avoid N+1 queries across
thousands of aircraft.

### 3. Campaign dispatch at scale
Outreach is resolved and dispatched as a **fire-and-forget background task** so
the HTTP request returns immediately while the UI polls status. Recipients are
resolved in 500-tail chunks and shipped to the shared email backbone in
**300-recipient batches**, deduped along the way — designed to handle
thousand-plus-recipient sends without blocking or timing out.

### 4. CRM → analytics sync (hangar-operator clients)
HubSpot is a transactional CRM, not an analytics store, so closed-won deals are
synced into a fast local table: a list of deal IDs is fanned into **batched
association reads** and then **batched object reads**, and materialized in a
single transaction. Two `asyncio` locks — one for the sync, one for a freshness
check — prevent a **thundering herd** when several dashboards load at once (a
stale cache triggers exactly one refresh, not one per viewer).

### 5. Live air-traffic
A live-radar service wraps the OpenSky Network for current aircraft states,
historical flight tracks, and hourly traffic **heatmaps**, each with tuned cache
TTLs (seconds for live state, an hour for heatmaps) to stay responsive without
hammering the upstream.

### 6. Correct time everywhere
All timestamps are stored as `timestamptz` (true UTC instants) and converted to
local (New York) time only at read/display time — eliminating a whole class of
off-by-one and DST bugs in digests, analytics, and lease-expiration windows.

---

## Tech Stack

| Layer | Technologies |
| :-- | :-- |
| **Backend** | Python 3.11 · FastAPI · asyncpg (pooled, 5–20 conns) · Pydantic v2 · JWT (HS256) · orjson |
| **Frontend** | React 18 · Material UI 7 · React Router 7 · Axios (JWT interceptors) · Leaflet · ECharts |
| **Data** | PostgreSQL (schema-per-tenant) · PostGIS · shared `flight_history` store |
| **Integrations** | Aviation flight-data API · HubSpot · OpenSky Network · SendGrid · Twilio |
| **Cloud & CI/CD** | Cloud Run · Cloud SQL · Cloud Scheduler · Secret Manager · GitHub Actions + Workload Identity Federation |

---

## Outcome

A production, multi-tenant demand platform where onboarding a new FBO is mostly
configuration (a schema, a registry entry, a branded build, a backfill), daily
prospecting is automated end-to-end, and operators get a live, data-grounded
view of demand — from raw flight activity all the way through enrichment,
personalized outreach, and pipeline analytics.

<sub>Individual FBO client identities and the flight-data vendor are generalized.
Proprietary data, infrastructure identifiers, and credentials are omitted.</sub>
