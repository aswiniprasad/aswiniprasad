# Outreach — AI Email Campaign Platform

> A multi-product email-campaign platform with AI-assisted drafting, organic
> reputation-aware sending, and real-time engagement tracking — built so several
> products can share one hardened email-delivery backbone.

**Role:** Architect & full-stack engineer
**Stack:** Python 3.11 · FastAPI · asyncpg · React 18 · Vite · TypeScript · Tiptap · Anthropic Claude · SendGrid · Google Cloud Tasks · Cloud Run
**Domain:** Marketing / outbound email infrastructure

---

## The Problem

Multiple products each needed to run targeted email campaigns — but every one
reinventing sending, rate-limiting, deliverability, and engagement tracking would
be wasteful and fragile. The goal was a **single shared email backbone** that any
product could drive, with three things done right:

1. **Composition** — help users write good emails fast (AI drafting).
2. **Delivery** — send organically without torching sender reputation.
3. **Feedback** — know what happened (opens, clicks, bounces) in real time.

---

## Architecture

```
   Outreach UI (React + Vite + Tiptap editor)
              │
   Outreach API (FastAPI) ── campaigns, templates, contacts, senders
              │  (thin AI proxy)          │ (batched recipients)
              ▼                           ▼
   ┌───────────────────────────────────────────────┐
   │        Shared Email Service (FastAPI)          │
   │  AI drafting (Claude) · durable send queue     │
   │  · SendGrid dispatch · engagement webhooks     │
   └───────┬───────────────────────────┬────────────┘
           ▼                           ▼
     Cloud Tasks                   SendGrid API
   (rate-limited queue)      (send + event webhooks)
```

- **Outreach API** owns product-facing concerns: campaigns, templates, contacts,
  senders, scheduling.
- **Shared Email Service** owns delivery: AI generation, the durable send queue,
  SendGrid dispatch, and inbound engagement webhooks. It's **multi-tenant** —
  each product is an isolated `client_id` behind an API key.

---

## Key Features

- **Campaign editor** with live audience preview and a **live-query audience
  model** — campaigns store *filter criteria*, not frozen lists, so each daily
  batch re-evaluates against fresh contact data.
- **Rich template editor** (Tiptap) with merge variables (`{{first_name}}`,
  `{{company}}`, …) and GCS-backed attachments.
- **AI email drafting** — generate a draft from plain-English intent, or refine
  an existing draft (shorten, formalize, rewrite).
- **Organic sending** — reputation-aware, rate-limited delivery.
- **Engagement tracking** — per-recipient open/click/bounce counts and
  timestamps, updated in real time.

---

## Engineering Challenges & Solutions

### 1. AI drafting, centralized and audited
Rather than scatter LLM calls across every product, drafting lives in the
**shared service**. The Outreach API is a **thin proxy**: it authenticates the
user, injects product/brand context and the available merge-variable set, and
forwards the request. Every generation is logged with user identity for audit.

### 2. Sending organically without wrecking deliverability
Blasting a campaign trips spam filters and damages sender reputation. Sends go
through **Google Cloud Tasks** as a **durable, rate-limited queue** (a steady
few messages per second, bounded concurrency). Tasks survive restarts, retry
with exponential backoff, and record per-recipient failures — no in-memory loss,
no thundering send.

### 3. Trustworthy engagement data
Inbound SendGrid event webhooks are **signature-verified (ECDSA)** before
they're trusted, so engagement data can't be spoofed. Events correlate back to
the exact recipient via a custom argument, and machine-open events are filtered
out for cleaner open metrics.

### 4. Sharing a schema without leaking across products
Campaign recipients reference product contacts by a bare id (no cross-schema
foreign keys). The service resolves the correct product contact table by slug at
query time, and every query is scoped by `client_id` — products share
infrastructure, never data.

---

## Infrastructure & Delivery

- **Google Cloud Run** (Outreach API, UI, Shared Email Service)
- **Cloud SQL** (PostgreSQL) · **Cloud Tasks** (send queue) · **Cloud Storage**
  (attachments)
- **GitHub Actions → Cloud Run** via **Workload Identity Federation**
- Cron-driven daily batch scheduling with per-campaign send windows and daily
  limits

---

## Outcome

One hardened email backbone that any product can drive — with AI-assisted
composition, deliverability-safe organic sending, and real-time, tamper-proof
engagement analytics. A clean example of extracting shared infrastructure into a
multi-tenant service without sacrificing data isolation.

<sub>Client/product identities generalized. Credentials and proprietary data omitted.</sub>
