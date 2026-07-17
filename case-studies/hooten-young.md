# Hooten Young — AI Sales & Marketing Intelligence Platform

> An internal intelligence platform for a premium American spirits brand: an
> idempotent, auditable sales/depletions analytics engine on one side, and a
> multi-model AI marketing engine with built-in regulatory compliance on the
> other — replacing hand-built spreadsheets and a mostly manual content process.

**Role:** Architect / Full-Stack & AI Engineer
**Stack:** Python 3.12 · FastAPI · SQLAlchemy 2.0 (async) · PostgreSQL + pgvector · React · Vite · TypeScript · Claude · OpenAI · Gemini / Vertex AI · Replicate (Flux) · Google Cloud Run
**Domain:** Consumer goods / spirits — sales analytics + marketing AI

---

## The Problem

A growing spirits brand had two data-dependent needs:

1. **Sales leadership** wanted weekly visibility into wholesale sales and
   *depletions* (true retail sell-through) — reporting the team previously
   assembled by hand in spreadsheets.
2. **Marketing** wanted to understand what was working across social media,
   generate on-brand content fast, and stay inside strict alcohol-advertising
   regulations.

Both came down to **trust**: reproducible numbers with an audit trail, and AI
output that's on-brand *and* legally compliant.

---

## Part 1 — Sales & Depletions Analytics

Turns weekly wholesale sales and depletion data into reporting the sales team
used to build by hand — **saving roughly 8–10 hours per week**.

- **Idempotent, auditable ingestion** — every uploaded file is SHA-256 hashed
  (re-uploads are no-ops), fact tables carry natural-key uniqueness (corrections
  upsert in place), and every row traces back to its source file. No destructive
  deletes.
- **Format-resilient parsing** — broker-specific parsers convert messy exports
  into a stable canonical model; when a format changes, only the parser swaps.
- **Decision-ready analytics** — coverage gaps, **distributor-concentration
  risk**, and cross-sell patterns that **surface growth opportunities and
  at-risk accounts previously invisible**.
- **Field-sales CRM** — generates **a prioritized daily call list per rep**
  across **5 user roles**, ranked by account value, recent activity, and
  follow-up timing.

---

## Part 2 — AI Marketing Intelligence

An automated weekly job analyzes **6,000+ social posts and 80+ GB of media**,
replacing a mostly manual, multi-week content process.

- **Multi-model LLM pipeline** — each task is **routed to the best-fit provider
  for cost and quality** (Claude / OpenAI / Gemini), rather than forcing one
  model to do everything.
- **RAG over social signal** — content and engagement signals are embedded into
  **pgvector** for similarity search and pattern mining (hook types, timing,
  caption-length effects), grounding briefs in real competitive data.
- **Content briefs & drafts** — the pipeline produces briefs and on-brand draft
  copy the team can act on immediately.
- **On-brand multimodal media** — image generation (Replicate / Flux) prepends a
  fixed brand-aesthetic style and injects reference images as few-shot examples
  for visual consistency.
- **Compliance-gated output** — a **TTB (alcohol-marketing) compliance review**
  runs on **every** AI-generated asset before it reaches the team, removing a
  manual legal review from the workflow.

---

## Engineering Challenges & Solutions

| Challenge | Solution |
|---|---|
| Reproducible numbers from messy weekly files | SHA-256 dedup + natural-key upserts + per-row file lineage |
| Broker format drift | Parser/model separation — only the parser changes |
| Best model for each job | Multi-model routing by cost/quality per task |
| Grounding AI in real signal | RAG over pgvector-embedded social data |
| Regulated AI output | Automated TTB compliance review as a hard gate |
| Consistent AI media | Fixed brand-style prompt prefix + reference-image few-shots |
| Frontend/backend type drift | Pydantic schemas mirrored 1:1 in TypeScript; `Decimal` as string to preserve precision |

---

## Infrastructure & Delivery

- **Google Cloud Run** (per-service, per-environment) · **Cloud SQL** (Postgres +
  pgvector) · **Cloud Storage** (raw + generated media) · **Secret Manager**
- **GitHub Actions** — auto-deploy to dev on `main`; tagged releases to prod with
  approval
- Strict typing throughout (`mypy --strict`, TypeScript strict); ruff + pytest in
  pre-commit

---

## Outcome

Sales leadership gets trustworthy, auditable analytics and a prioritized daily
call list from otherwise-messy weekly data, and marketing gets a multi-model AI
engine that produces on-brand, compliance-checked content grounded in real
competitive signal — two hard problems (data integrity and regulated generative
AI) solved in one platform.

<sub>Proprietary data, sources, and credentials omitted.</sub>
