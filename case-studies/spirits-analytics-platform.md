# Spirits Sales & Marketing Intelligence

> A two-sided intelligence platform for a premium American whiskey brand: an
> idempotent, auditable sales/depletions analytics pipeline on one side, and an
> AI marketing engine (RAG + multimodal generation) with built-in regulatory
> compliance review on the other.

**Role:** Architect & full-stack engineer
**Stack:** Python 3.12 · FastAPI · SQLAlchemy 2.0 (async) · PostgreSQL + pgvector · React · Vite · TypeScript · Anthropic Claude · Replicate (Flux) · Vertex AI (Veo) · Google Cloud Run
**Domain:** Consumer packaged goods / spirits — sales analytics + marketing AI

---

## The Problem

A growing spirits brand had two distinct needs that both depended on trustworthy
data:

1. **Sales leadership** wanted Monday-morning visibility into distribution and
   *depletions* — what actually sold through to retail — from weekly
   spreadsheet exports (QuickBooks, broker pivots) that were messy, occasionally
   re-sent, and sometimes corrected.
2. **Marketing** wanted to understand what was working across social media,
   generate on-brand content, and stay inside strict alcohol-advertising
   regulations.

The engineering challenge in both cases was **trust**: reproducible numbers with
an audit trail, and AI output that's on-brand *and* legally compliant.

---

## Architecture

A modular monorepo of independently deployed services:

```
   Sales Dashboard (React + Vite + TS, TanStack Query)
             │
   ┌─────────┴──────────┐
   │  Sales API         │   idempotent xlsx ingestion → canonical fact tables
   │  (FastAPI)         │   (QuickBooks + broker depletion pivots)
   └─────────┬──────────┘
             │              shared PostgreSQL (Cloud SQL)
   ┌─────────┴──────────┐
   │  Marketing API     │   social intelligence + AI brief/media generation
   │  (FastAPI)         │   Postgres + pgvector · Claude · Flux · Veo · GCS
   └────────────────────┘
```

---

## Part 1 — Sales & Depletions Analytics

### Idempotent, auditable ingestion
Weekly xlsx files are unreliable: duplicates, retransmissions, and corrections
are normal. The pipeline treats ingestion as a **ledger**:

- Every uploaded file is **SHA-256 hashed** — re-uploading the same file is a
  no-op.
- Fact tables carry **natural-key uniqueness**; a corrected file **upserts**
  affected rows in place.
- Every fact row references the `file_uploads` entry it came from — full audit
  lineage, no destructive deletes.

### Format-resilient parsing
Broker export layouts change. **Broker-specific parsers** convert raw rows into
a stable canonical model; if a format changes, only the parser swaps and the
rest of the system is untouched.

### The dashboards
KPI strips (revenue, cases, commission, geographic reach, active SKUs), revenue
trends, product performance, distributor analysis, a **white-space matrix**
(untapped state × account-tier combinations), and a depletions view showing true
retail sell-through vs. distribution.

---

## Part 2 — AI Marketing Intelligence

### Social pattern mining (RAG)
Content and engagement signals from tracked competitor accounts are embedded
into **pgvector**, enabling similarity search and pattern extraction — hook
types, posting-time effects, caption-length correlations.

### A three-mode brief engine
An orchestration layer measures how saturated a trend is among competitors and
picks a strategy:

- **Follow** — apply proven patterns when they're working.
- **Counter** — recommend a contrarian angle when competitors converge.
- **Pioneer** — surface cultural trends (trends feeds, editorial sources, forum
  culture) not yet tapped by category competitors.

Claude receives the best-fit briefing prompt and drafts the content.

### Compliance-gated generation
Spirits marketing is heavily regulated. **Every** AI-generated piece of copy is
routed through an automated **compliance review** (federal + state alcohol-
advertising rules) *before* it's returned to the UI — compliance is a gate in
the pipeline, not an afterthought.

### On-brand multimodal media
Image (Replicate / Flux) and video (Vertex AI / Veo) generation prepend a fixed
brand-aesthetic style and inject reference images as few-shot examples, so
generated media stays visually consistent with the brand.

---

## Engineering Challenges & Solutions

| Challenge | Solution |
|---|---|
| Reproducible numbers from messy weekly files | SHA-256 dedup + natural-key upserts + per-row file lineage |
| Broker format drift | Parser/model separation — only the parser changes |
| Frontend/backend type drift | Pydantic schemas mirrored 1:1 in TypeScript; `Decimal` carried as string to preserve precision |
| Regulated AI output | Automated compliance review as a hard gate before any copy is surfaced |
| Consistent AI media | Fixed brand-style prompt prefix + reference-image few-shots |

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

Sales leadership gets trustworthy, auditable analytics from otherwise-messy
weekly exports, and marketing gets an AI engine that generates on-brand,
compliance-checked content grounded in real competitive signal — two hard
problems (data integrity and regulated generative AI) solved in one platform.

<sub>Client identity generalized. Proprietary data, sources, and credentials omitted.</sub>
