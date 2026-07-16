# Aviation Intelligence Platform

> A multi-tenant SaaS platform serving several private-aviation operators from a
> single shared backend — flight-history ingestion, FBO fuel-price intelligence,
> aircraft enrichment, and email/SMS outreach — with strict per-client data
> isolation and zero-downtime operations.

**Role:** Architect & full-stack engineer (backend, frontend, data, infra)
**Stack:** Python · FastAPI · asyncpg · React · Material UI · PostgreSQL · Google Cloud Run
**Domain:** Private aviation / FBO (Fixed-Base Operator) intelligence

---

## The Problem

Several private-aviation operators each needed their own branded dashboard to
answer the same core questions: *Who is flying into my airports? What are
competitors charging for fuel? Who owns and operates these aircraft, and how do
I reach them?*

Building four independent apps would mean four copies of the same expensive
logic — flight-data ingestion, enrichment, digests — drifting out of sync.
Building one shared app would risk leaking one client's data into another's.

The design goal: **share everything expensive, isolate everything sensitive.**

---

## Architecture

A **hub-and-spoke, multi-tenant** design:

```
        Per-client UIs (React + MUI, one branded build each)
                              │
        Per-client APIs (FastAPI) ── isolated Postgres schema per tenant
                              │
                 ┌────────────┴────────────┐
                 │   Aviation Shared Service │  (single codebase, tenant registry)
                 └────────────┬────────────┘
             ┌────────────────┼────────────────┐
        JetNet API       SendGrid / Twilio   LinkedIn / FAA
      (flight history)   (digest + outreach)  (enrichment)
```

- **Per-tenant isolation** — each client gets a dedicated Postgres **schema**
  (`clientA`, `clientB`, …). Every query is schema-qualified, so cross-tenant
  leakage is structurally impossible rather than a matter of careful `WHERE`
  clauses.
- **Shared service** — one codebase owns the compute-heavy, cross-cutting work
  (flight ingestion, daily digest emails, enrichment lookups). A Python **tenant
  registry** maps each client to its schema, email identity, and feature flags,
  and all reads are parameterized by that registry.
- **Async-first** — asyncpg connection pooling (5–20 connections) with query
  timeouts; every endpoint is `async/await`.

---

## Key Features

- **FBO fuel-price intelligence** — nearest-FBO Jet A / AvGas pricing with
  distance sorting and airport/fuel-type filters.
- **Flight-history visualization** — map-based arrival/departure tracking
  (Leaflet), filterable by direction, operator, and date range.
- **Aircraft enrichment** — tail-number lookup joining FAA registration data
  with owner/operator network contacts.
- **Outreach hub** — templated email (SendGrid) and SMS (Twilio) campaigns with
  per-recipient send history.
- **Daily digest** — a cross-tenant 6 AM email summarizing logins, searches,
  feedback, and campaign engagement per client.

---

## Engineering Challenges & Solutions

### 1. Keeping four clients in sync without four codebases
A single **reference implementation** holds canonical feature logic; client
services differ only by schema name and branding. Shared, expensive logic lives
in the shared service — so a fix to flight ingestion ships once, for everyone.

### 2. Resilient third-party data ingestion
Flight data syncs on a **7-day rolling window** per airport with a **14-day
safety gate**: if a scheduled sync falls too far behind (e.g. an outage), it
halts loudly instead of silently importing stale data. A **resumable backfill
script** recovers month-by-month, tracking how far each airport has been
fetched. Adding a new airport is a single config-table insert — the cron picks
it up with no redeploy.

### 3. Zero-downtime credential rotation
The flight-data API token is fetched from **Secret Manager** with a short-lived
in-memory cache. Rotating the upstream password is a new secret version — no
redeploy, no downtime.

### 4. Correct time everywhere
All timestamps are stored as `timestamptz` (true UTC instants) and converted to
local (New York) time only at read/display time — eliminating a whole class of
off-by-one and DST bugs in digests and analytics.

---

## Infrastructure & Delivery

- **Google Cloud Run** (auto-scaling, serverless) for every service
- **Cloud SQL** (PostgreSQL) as the shared database, schema-isolated per tenant
- **Cloud Scheduler** for daily ingestion + digest cron jobs
- **GitHub Actions → Cloud Run** deploys via **Workload Identity Federation**
  (no long-lived service-account keys)
- **pre-commit** (black, isort, pylint) enforced on every backend

---

## Outcome

A production, multi-tenant analytics platform where onboarding a new client is
mostly configuration — a new schema, a registry entry, a branded frontend build —
rather than a new codebase. The architecture demonstrates enterprise-grade
multi-tenancy, operational resilience, and cloud-native best practices end to end.

<sub>Client identities generalized. Proprietary data and credentials omitted.</sub>
