# Hooten Young — AI Sales & Marketing Intelligence Platform

> A two-sided intelligence platform for a premium American spirits brand: an
> idempotent, auditable sales/depletions analytics engine on one side, and a
> content-intelligence engine (task-routed LLMs + RAG + multimodal media
> generation) with built-in regulatory compliance on the other.

<table>
<tr><td><b>Role</b></td><td>Forward Deployed AI Engineer</td></tr>
<tr><td><b>Domain</b></td><td>Consumer goods · spirits — sales analytics + marketing AI</td></tr>
<tr><td><b>Shape</b></td><td>Monorepo · 3 independently deployed services · shared Postgres</td></tr>
<tr><td><b>Core stack</b></td><td>Python 3.12 · FastAPI · SQLAlchemy 2.0 (async) · PostgreSQL + pgvector · React · Vite · TypeScript · Anthropic Claude · Replicate · Vertex AI</td></tr>
</table>

---

## Overview

A growing spirits brand had two data-dependent needs:

1. **Sales leadership** wanted weekly visibility into wholesale sales and
   *depletions* (true retail sell-through) — reporting the team previously
   rebuilt by hand in spreadsheets — plus a field-rep tool to work accounts.
2. **Marketing** wanted to understand what was working across social media,
   generate on-brand content fast, and stay inside strict alcohol-advertising law.

Both reduce to one hard requirement: **trust** — reproducible numbers with an
audit trail, and AI output that is on-brand *and* legally compliant.

The platform is a monorepo of three independently deployed Cloud Run services —
a **sales** backend, a **marketing** backend, and a **React dashboard** — over a
shared PostgreSQL database.

---

## Part 1 — Sales & Depletions Analytics

Turns weekly wholesale and depletion exports into decision-ready reporting the
team used to assemble by hand.

### Idempotent, auditable ingestion
Weekly xlsx files are unreliable (duplicates, retransmissions, corrections), so
ingestion is modeled as a **ledger**:

- Every uploaded file is **SHA-256 hashed** against a `file_uploads` table with a
  `UNIQUE` constraint — re-uploading the same file is a no-op.
- Fact rows use natural keys (e.g. `account × product × month`) and
  **`INSERT … ON CONFLICT DO UPDATE`** with a guard so unchanged rows are true
  no-ops; a corrected file upserts only what changed, in place.
- Every upload records processed/inserted/updated/skipped/failed counts and a
  status — full lineage, no destructive deletes.

### High-volume, format-resilient parsing
A ~130k-row file is ingested in a **three-phase bulk pipeline** — resolve
products, resolve accounts, then batch-upsert facts (500 per batch, each in a
savepoint for fault isolation) — collapsing ~130k round-trips into a few hundred
queries (minutes → seconds). Broker exports have shipped in **four different
layouts** over time, so parsers **auto-detect the layout** (by locating the
`Products` header) and convert every variant into one canonical model; a new
layout swaps a parser, not the pipeline.

### Strategic analytics
Beyond KPIs and trend/product/distributor/state breakdowns, the sales API
computes genuinely strategic views:

- **White-space matrix** — product × state coverage, quantifying untapped
  combinations (gap count / gap %).
- **Order analysis** — order-size buckets, **cross-sell pairs** (which SKUs ship
  together), and per-distributor order frequency.
- **Risk dashboard** — distributor-**concentration risk** via a Herfindahl index
  and top-N cumulative share.

### Field-sales CRM
A role-gated field tool generates a **prioritized daily call list** per rep,
bucketing accounts by recency (Active / At-Risk / Cold) and honoring manual
pins, with territory assignment (auto-seeded from the rep's home state) and an
audited visit-note timeline. Access is governed by a **data-driven RBAC** with
**5 roles** (admin, distribution, depletions, marketing, field rep) — adding a
role is a row insert, not a code change.

---

## Part 2 — AI Marketing Intelligence

A weekly pipeline ingests social + editorial signal, mines patterns, and
generates compliance-checked, on-brand content.

### Task-routed, multi-model LLM pipeline
Rather than force one model to do everything, each task is **routed to the
best-fit model for cost and quality**:

| Task | Model tier |
| :-- | :-- |
| Content briefs, scripts, compliance reasoning | Claude **Sonnet** |
| High-volume article/feature classification | Claude **Haiku** (cheaper, faster) |
| Semantic embeddings (RAG) | embedding model → **pgvector** |
| Image generation | **Replicate / Flux** (text + reference-conditioned) |
| Video generation | **Vertex AI / Veo** |

### The three-mode brief engine
An orchestration layer measures how saturated a trend is across tracked
competitor accounts and picks a strategy:

- **Follow** — apply proven patterns when competitors are converging on what
  works.
- **Counter** — recommend a contrarian angle when everyone looks the same.
- **Pioneer** — surface cultural trends (trends feeds, editorial sources, forum
  culture) the category hasn't touched yet.

Signal is pulled by pluggable scrapers, embedded into **pgvector** for
similarity/pattern mining, and grounds the brief the model writes.

### Compliance-gated generation
Alcohol marketing is heavily regulated, so **every** AI-generated asset passes an
automated **compliance review** (federal TTB + state + platform rules) that
returns a verdict of `compliant | revise | block`. On `revise` it returns quoted
phrases and suggested rewrites that preserve brand voice and can be auto-applied;
compliance is a **hard gate in the pipeline**, not an afterthought.

### On-brand multimodal media
Image (Flux) and video (Veo) generation prepend a fixed brand-aesthetic style and
inject reference images as few-shot examples, keeping generated media visually
consistent; reference-conditioned modes preserve product/bottle shape.

---

## Engineering Challenges & Solutions

| Challenge | Solution |
| :-- | :-- |
| Reproducible numbers from messy weekly files | SHA-256 file ledger + natural-key upserts + per-row lineage |
| 130k-row files without minute-long ingests | Three-phase bulk resolve/upsert with per-batch savepoints |
| Broker format drift | Layout auto-detection → single canonical model |
| Best model per job | Task-routed Claude tiers (Sonnet ↔ Haiku) + dedicated media models |
| Grounding AI in real signal | RAG over pgvector-embedded social data |
| Regulated AI output | Compliance review as a hard `compliant/revise/block` gate |
| Frontend/backend type drift | Pydantic schemas mirrored in TypeScript; `Decimal` carried as string to preserve precision |

---

## Tech Stack

| | |
| :-- | :-- |
| **Sales backend** | ![Python](https://img.shields.io/badge/Python_3.12-3776AB?style=flat&logo=python&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white) ![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy_2.0-D71F00?style=flat&logo=sqlalchemy&logoColor=white) ![asyncpg](https://img.shields.io/badge/asyncpg-316192?style=flat&logo=postgresql&logoColor=white) ![pandas](https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white) ![uv](https://img.shields.io/badge/uv-DE5FE9?style=flat&logo=uv&logoColor=white) |
| **Marketing backend** | ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white) ![pgvector](https://img.shields.io/badge/pgvector-4169E1?style=flat&logo=postgresql&logoColor=white) ![Alembic](https://img.shields.io/badge/Alembic-6BA81E?style=flat) ![Anthropic](https://img.shields.io/badge/Anthropic_Claude-D97757?style=flat&logo=anthropic&logoColor=white) ![Replicate](https://img.shields.io/badge/Replicate_Flux-000000?style=flat&logo=replicate&logoColor=white) ![Vertex AI](https://img.shields.io/badge/Vertex_AI_Veo-4285F4?style=flat&logo=googlecloud&logoColor=white) |
| **Dashboard** | ![React](https://img.shields.io/badge/React_18-61DAFB?style=flat&logo=react&logoColor=black) ![Vite](https://img.shields.io/badge/Vite_5-646CFF?style=flat&logo=vite&logoColor=white) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white) ![MUI](https://img.shields.io/badge/MUI_6-007FFF?style=flat&logo=mui&logoColor=white) ![TanStack Query](https://img.shields.io/badge/TanStack_Query-FF4154?style=flat&logo=reactquery&logoColor=white) ![Recharts](https://img.shields.io/badge/Recharts-22B5BF?style=flat) |
| **Data** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white) ![pgvector](https://img.shields.io/badge/pgvector-4169E1?style=flat&logo=postgresql&logoColor=white) ![Cloud SQL](https://img.shields.io/badge/Cloud_SQL-4285F4?style=flat&logo=googlecloud&logoColor=white) |
| **Cloud & CI/CD** | ![Cloud Run](https://img.shields.io/badge/Cloud_Run-4285F4?style=flat&logo=googlecloud&logoColor=white) ![Cloud Storage](https://img.shields.io/badge/Cloud_Storage-4285F4?style=flat&logo=googlecloud&logoColor=white) ![Secret Manager](https://img.shields.io/badge/Secret_Manager-4285F4?style=flat&logo=googlecloud&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white) |

---

## Outcome

Sales leadership gets trustworthy, auditable analytics and a prioritized daily
call list from otherwise-messy weekly data; marketing gets a task-routed AI
engine that produces on-brand, compliance-checked content grounded in real
competitive signal. Two genuinely hard problems — **data integrity** and
**regulated generative AI** — solved in one platform.
