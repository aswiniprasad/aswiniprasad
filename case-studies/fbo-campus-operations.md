# FBO Campus Operations & Revenue Analytics

> A real-estate operations dashboard for a private-aviation hangar operator —
> turning a live CRM feed into occupancy, lease-health, and revenue intelligence
> across a multi-campus portfolio, with live air-traffic radar layered on top.

**Role:** Architect & full-stack engineer
**Stack:** Python · FastAPI · PostgreSQL · React · Material UI · Recharts · Leaflet · Google Cloud Run
**Domain:** Private-aviation real estate / FBO campus operations

---

## The Problem

Unlike the aircraft-sales dashboards I'd built for other aviation clients, this
operator's business is **real estate**: leasing hangar space across multiple
airport campuses. Leadership needed to see — across the whole portfolio and
per campus — *how full are we, how much revenue is that, and which leases are
about to lapse?*

All of that data already lived in their **CRM** as closed-won deals. The problem
was making it live, aggregated, and decision-ready without hand-built
spreadsheets.

---

## Architecture

```
   Executive + per-campus dashboards (React + Recharts + Leaflet)
                          │
                    FastAPI backend
        ┌─────────────────┼─────────────────┐
   HubSpot CRM        PostgreSQL          OpenSky Network
 (closed-won deals →  (skyharbour schema, (live aircraft
  live cached table)   cached deal data)   positions)
```

- **Live CRM sync** — closed-won deals (with their associated companies,
  contacts, campus, aircraft, and hangar objects) are pulled from HubSpot via
  batched association + object reads and cached into a live Postgres table with
  a short TTL.
- **Revenue rollups** — per-deal economics (base rent, triple-net, tax, fuel)
  roll up to campus, then to portfolio, using SQL window functions.
- **Live radar** — the OpenSky Network feed powers a real-time map of aircraft,
  filtered to tenants' tail numbers.

---

## Key Features

- **Executive dashboard** — portfolio-wide occupancy %, revenue, and deal counts,
  with per-campus breakdown cards and year-over-year growth-velocity trends.
- **Campus dashboard** — occupancy by square footage, revenue composition, fuel
  margins, and a full resident/tenant table.
- **Lease-health monitoring** — color-coded expiration urgency (expired /
  critical / warning / healthy) and **revenue-at-risk** by year.
- **Live air-traffic radar** — real-time aircraft positions with a tenant-only
  filter.
- **Multi-phase campus modeling** — operational phases of a single campus are
  tracked and gated independently.

---

## Engineering Challenges & Solutions

### 1. Turning a transactional CRM into an analytics source
HubSpot is a CRM, not an analytics store. Reading it live on every dashboard
load would be slow and rate-limited. Solution: a **sync service** that fans a
list of deal IDs into batched association calls, then batched object reads, and
materializes the result into a `hubspot_deals_live` table. Dashboards read the
fast local table; the sync refreshes on a TTL.

### 2. Preventing thundering-herd syncs
When several users open dashboards at once, all of them could trigger a refresh
simultaneously. An **asyncio lock** serializes sync work, and a separate
freshness check short-circuits redundant refreshes — so a stale cache triggers
exactly one sync, not one per viewer.

### 3. Multi-phase campuses
Some campuses operate in distinct phases that leadership wants to track
separately, even though the CRM stores them under one code. The dashboard
layer **expands** that single code into per-phase views on read, and access
control gates each phase independently.

### 4. Timezone-correct financial reporting
Lease expirations, revenue bucketing, and digest dates all convert `timestamptz`
instants to local time at read time — so "leases expiring this year" and daily
digests never suffer fence-post errors.

---

## Infrastructure & Delivery

- **Google Cloud Run** + **Cloud SQL** (isolated `skyharbour` schema)
- **GitHub Actions** deploys via **Workload Identity Federation**
- Shares authentication and daily-digest infrastructure with the broader
  aviation platform, while remaining fully data-isolated

---

## Outcome

Leadership went from spreadsheet-driven guesswork to a live, portfolio-wide view
of occupancy, revenue, and lease risk — plus a real-time picture of who's flying
in. The build shows how to responsibly turn a transactional CRM into a
performant analytics product without a separate data warehouse.

<sub>Client identity generalized. Revenue figures, deal terms, and credentials omitted.</sub>
