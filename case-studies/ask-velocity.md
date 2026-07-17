# Ask Velocity — AI Demand Platform for Private Aviation (FBOs)

> A multi-tenant SaaS that helps Fixed-Base Operators (FBOs) grow hangar leasing
> and fuel sales — ingesting private-jet flight activity, ranking high-value
> prospects, running AI-personalized outreach, and giving operators a live,
> forward-looking view of demand and pipeline.

**Live:** [askvelocity.ai](https://askvelocity.ai)
**Role:** Architect / Full-Stack & AI Engineer
**Stack:** Python · FastAPI · asyncpg · React · Material UI · Leaflet · PostgreSQL (PostGIS / pgvector) · Google Cloud Run · Aviation data APIs · HubSpot · SendGrid · Twilio
**Domain:** Private aviation / FBO demand generation

---

## The Problem

FBOs make money two ways: leasing hangar space and selling jet fuel. Both depend
on knowing **who is flying into their airports**, who owns and operates those
aircraft, and how to reach them — intelligence that normally means hours of
manual flight-log review and lead research, repeated every day, at every
location.

Ask Velocity turns that manual grind into an automated demand engine — and does
it for **many FBO clients at once** from one platform.

---

## Architecture

A **hub-and-spoke, multi-tenant** design: one shared backend, isolated data per
tenant.

```
        Per-tenant React dashboards (maps · live traffic · pipeline)
                              │
                    FastAPI backend (async)
                              │
        ┌─────────────────────┼─────────────────────┐
   Shared services        Per-tenant Postgres     Integrations
   (ingestion, ranking,   schemas (isolated       Flight data · HubSpot
    outreach, digests)    per FBO client)          SendGrid · Twilio
```

- **Multi-tenant with isolation** — each FBO client gets a dedicated Postgres
  schema; expensive shared logic (flight ingestion, enrichment, ranking,
  outreach, digests) lives in one codebase driven by a tenant registry.
- **Onboarding as configuration** — spinning up a new client is a new schema, a
  registry entry, and a branded frontend build — **weeks of development collapsed
  to about a day.**

---

## Key Features

### Flight-activity ingestion at scale
A daily pipeline pulls private-jet flight activity from **multiple aviation data
sources across 30+ airports**, then automatically identifies and enriches
high-value prospects —
**removing several hours of manual flight-log review each day**. The sync uses a
rolling window with a safety gate so an upstream outage halts loudly instead of
importing stale data, and a resumable backfill recovers cleanly.

### Lead ranking
A scoring system prioritizes aircraft owners and operators using
**data-completeness tiers** and a custom **contact-quality score**, so sales
teams work the best leads first — **cutting manual lead research by roughly 90%**.

### AI-personalized outreach
**AI-generated, personalized email and SMS** goes out through a **rate-limited
queue for up to 10,000+ prospects per campaign**, with open, click, and bounce
tracking via SendGrid and Twilio — reputation-aware sending, not a blast.

### Live demand & pipeline dashboards
React dashboards with **interactive maps and live air-traffic data**, plus a
**HubSpot CRM sync** that links closed deals back to specific aircraft and
contacts — one real-time view of demand and pipeline.

### What-If scenario planner
An interactive planner lets executives model levers — **tenant occupancy, fuel
volume, hangar utilization, traffic** — and **forecast revenue impact in real
time**, elevating the platform from static reporting into a forward-looking
decision-support tool.

---

## Engineering Challenges & Solutions

| Challenge | Solution |
|---|---|
| Serve many FBO clients without leaking data | Shared backend + per-tenant Postgres schemas; every query schema-qualified |
| Onboard clients fast | New client = schema + registry entry + branded build (~1 day) |
| Third-party flight data reliability | Rolling-window sync + safety gate + resumable backfill |
| Prioritize thousands of raw leads | Data-completeness tiers + contact-quality scoring |
| Send at volume without wrecking deliverability | Rate-limited queue, per-recipient tracking, machine-open filtering |
| Turn a CRM into analytics | TTL-cached HubSpot sync linking deals ↔ aircraft ↔ contacts |
| Move from reporting to planning | Real-time What-If model over occupancy/fuel/utilization/traffic |

---

## Infrastructure & Delivery

- **Google Cloud Run** (serverless, auto-scaling) · **Cloud SQL** (PostgreSQL,
  PostGIS/pgvector) · **Cloud Scheduler** (daily ingestion + digests)
- **Secret Manager** for zero-downtime credential rotation
- **GitHub Actions → Cloud Run** via **Workload Identity Federation** (no
  long-lived keys)

---

## Outcome

A production, multi-tenant demand platform where onboarding a new FBO is mostly
configuration, daily prospecting is automated end to end, and operators get a
live, forward-looking view of demand — from raw flight data all the way to
AI-driven outreach and revenue forecasting.

<sub>Individual FBO client identities generalized. Proprietary data and credentials omitted.</sub>
