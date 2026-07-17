# Outreach — AI Email Campaign Platform

> A multi-tenant platform that automates product-marketing outreach at scale —
> AI-assisted drafting, live-query audiences, reputation-aware "drip" sending on a
> durable queue, and tamper-proof engagement tracking — all sharing one hardened
> email-delivery backbone.

<table>
<tr><td><b>Role</b></td><td>Architect / Full-Stack &amp; AI Engineer</td></tr>
<tr><td><b>Domain</b></td><td>Marketing · outbound email infrastructure</td></tr>
<tr><td><b>Shape</b></td><td>3 services · shared email backbone · shared Postgres (schema-isolated)</td></tr>
<tr><td><b>Core stack</b></td><td>Python 3.11 · FastAPI · asyncpg · React · Vite · TypeScript · Tiptap · Anthropic Claude · SendGrid · Google Cloud Tasks</td></tr>
</table>

---

## Overview

Multiple products each needed to run targeted email campaigns — and having every
one reinvent sending, rate-limiting, deliverability, and engagement tracking would
be wasteful and fragile. The goal was a **single shared email backbone** any
product can drive, doing three things well:

1. **Composition** — help users write good emails fast (AI drafting).
2. **Delivery** — send organically without harming sender reputation.
3. **Feedback** — know exactly what happened (opens / clicks / bounces).

---

## Architecture

```
   Outreach UI (React · Vite · Tiptap editor)
              │
   Outreach API (FastAPI) ── campaigns · templates · contacts · senders · schedule
              │  thin AI proxy            │ batched recipients
              ▼                           ▼
   ┌───────────────────────────────────────────────────────┐
   │      Shared Email Service (FastAPI, multi-tenant)      │
   │  AI drafting (Claude) · durable send queue ·          │
   │  personalization · SendGrid dispatch · webhooks       │
   └───────┬───────────────────────────────┬───────────────┘
           ▼                               ▼
     Cloud Tasks (2/s, ≤5 concurrent)   SendGrid API (send + signed webhooks)
```

- **Outreach API** owns product-facing concerns; **Shared Email Service** owns
  delivery (AI, the durable queue, dispatch, inbound webhooks).
- **Multi-tenant isolation** — the shared service scopes every request to a
  `client_id` behind an API key; product contacts live in product-specific tables
  (e.g. `ask_abhi`, `ask_velocity`), so products share infrastructure, never data.

---

## Key Features & How They Work

### ✍️ AI drafting — centralized and audited
Drafting lives in the shared service (Anthropic **Claude**), not scattered across
products. The Outreach API is a **thin proxy**: it authenticates the user, injects
the product's brand name and available merge variables, and forwards the request.
The model returns strict JSON (`subject` + HTML `body`) using only the template's
real merge variables, and **every generation is logged** (client, user, action,
model, token counts) to an audit table — best-effort, so logging never blocks a
draft.

### 🚦 Reputation-aware "drip" sending
Blasting a campaign trips spam filters and damages sender reputation, so sends run
through **Google Cloud Tasks** as a **durable, rate-limited queue**: **2
dispatches/sec, ≤5 concurrent, up to 5 retries with 10s→300s exponential
backoff**, one idempotently-named task per recipient. Tasks survive restarts and
record per-recipient failures — steady, human-looking, no in-memory loss.

### 🎯 Live-query audiences (never email twice)
Campaigns store **filter criteria** (contact type / company / industry / tags) as
JSONB, not frozen lists. Each daily batch **re-evaluates** the filter against
current contacts and excludes anyone already in `campaign_contacts` via a
`NOT IN` subquery — so new matches are picked up automatically and **no contact is
ever emailed twice**. Batches are timezone-aware (IANA zones) and idempotent per
day via a `last_batch_date` guard; a **burst** mode sends a small list at once.

### 🧹 Personalization, greetings & suppression
`{{variable}}` substitution supports `{{name|default}}` fallbacks; an
auto-greeting is prepended unless the template already has one; and a per-product
**excluded-domains** suppression list is enforced in the batch query and takes
effect on the next batch with no restart.

### 📬 Tamper-proof engagement tracking
Inbound SendGrid event webhooks are **ECDSA signature-verified** before they're
trusted. The service records first-seen timestamps and cumulative open/click
counts per recipient, correlating events back to the exact recipient via a
custom argument (with message-id fallback), and **filters machine opens**
(Apple Mail Privacy Protection / scanners) so open metrics stay honest.

---

## Engineering Challenges & Solutions

| Challenge | Solution |
| :-- | :-- |
| Send at volume without wrecking deliverability | Cloud Tasks queue: 2/s, ≤5 concurrent, retries + backoff, one task/recipient |
| Atomic batch handoff across a network call | Insert `campaign_contacts` only after the delivery service accepts the batch, with `ON CONFLICT DO NOTHING` (safe retries) |
| Audiences that evolve mid-campaign | Live-query filter re-evaluated each batch + `NOT IN` never-email-twice guard |
| Cross-product data leakage | `client_id`-scoped queries + product-specific contact tables |
| Inflated open rates | `sg_machine_open` filtering on the webhook |
| Interactive AI latency | Async proxy, generous timeout, best-effort audit logging that never blocks the draft |

---

## Tech Stack

| | |
| :-- | :-- |
| **Outreach API** | ![Python](https://img.shields.io/badge/Python_3.11-3776AB?style=flat&logo=python&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI_0.115-009688?style=flat&logo=fastapi&logoColor=white) ![asyncpg](https://img.shields.io/badge/asyncpg-316192?style=flat&logo=postgresql&logoColor=white) ![Pydantic](https://img.shields.io/badge/Pydantic_v2-E92063?style=flat&logo=pydantic&logoColor=white) ![JWT](https://img.shields.io/badge/JWT-000000?style=flat&logo=jsonwebtokens&logoColor=white) ![httpx](https://img.shields.io/badge/httpx-2A6DB2?style=flat) |
| **Shared Email Service** | ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white) ![Anthropic](https://img.shields.io/badge/Anthropic_Claude-D97757?style=flat&logo=anthropic&logoColor=white) ![SendGrid](https://img.shields.io/badge/SendGrid-1A82E2?style=flat&logo=maildotru&logoColor=white) ![Cloud Tasks](https://img.shields.io/badge/Cloud_Tasks-4285F4?style=flat&logo=googlecloud&logoColor=white) ![GCS](https://img.shields.io/badge/Cloud_Storage-4285F4?style=flat&logo=googlecloud&logoColor=white) |
| **Outreach UI** | ![Vite](https://img.shields.io/badge/Vite_5-646CFF?style=flat&logo=vite&logoColor=white) ![React](https://img.shields.io/badge/React_18-61DAFB?style=flat&logo=react&logoColor=black) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white) ![Tiptap](https://img.shields.io/badge/Tiptap-000000?style=flat&logo=tiptap&logoColor=white) ![React Router](https://img.shields.io/badge/React_Router-CA4245?style=flat&logo=reactrouter&logoColor=white) ![nginx](https://img.shields.io/badge/nginx-009639?style=flat&logo=nginx&logoColor=white) |
| **Data** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white) ![JSONB](https://img.shields.io/badge/JSONB_filters-336791?style=flat&logo=postgresql&logoColor=white) ![Cloud SQL](https://img.shields.io/badge/Cloud_SQL-4285F4?style=flat&logo=googlecloud&logoColor=white) |
| **Cloud & CI/CD** | ![Cloud Run](https://img.shields.io/badge/Cloud_Run-4285F4?style=flat&logo=googlecloud&logoColor=white) ![Cloud Tasks](https://img.shields.io/badge/Cloud_Tasks-4285F4?style=flat&logo=googlecloud&logoColor=white) ![GCS](https://img.shields.io/badge/Cloud_Storage-4285F4?style=flat&logo=googlecloud&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white) |

---

## Outcome

One hardened email backbone any product can drive — with AI-assisted composition,
deliverability-safe organic sending, and real-time, tamper-proof engagement
analytics. A clean example of extracting shared infrastructure into a multi-tenant
service without sacrificing data isolation or metric integrity.

<sub>Client/product identities generalized. Credentials, infrastructure
identifiers, and proprietary data are omitted.</sub>
