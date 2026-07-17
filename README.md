# Hi there, I'm Prasad 👋

**Full-stack Software Engineer · Building and shipping production AI products end to end**
Applied AI/LLM systems · Full-stack · Cloud-native, event-driven infrastructure

[![Ask Velocity](https://img.shields.io/badge/Ask_Velocity-live-ff5a1f?style=for-the-badge&logo=rocket&logoColor=white)](https://askvelocity.ai)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/aswiniprasadyalavarthy/)
[![Email](https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:yap2k14@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/aswiniprasad)

---

## About

Full-stack software engineer with **9+ years** across logistics, scientific R&D,
and e-commerce — now focused on **building and shipping production AI products
end to end**. I've delivered live AI platforms for clients in **private
aviation**, **consumer goods**, and **marketing**, owning everything from data
architecture and LLM integration to React dashboards.

Every product is built around a simple goal — **save customers time and save the
business money** — by automating manual, error-prone work and turning raw data
into decisions, on cloud-native, event-driven infrastructure with strong CI/CD.

Currently **Lead Software Engineer** by day and an independent builder of AI
products by night.

---

## 🚀 Independent AI Products

_Actively building — deployed to production and serving live clients._

### ✈️ [Ask Velocity](https://askvelocity.ai) — AI Demand Platform for Private Aviation (FBOs)
A **multi-tenant SaaS** that scales across a growing roster of FBO clients on a
shared backend with isolated per-tenant databases — **new-client onboarding went
from weeks of development to about a day**.
- Daily pipeline ingests private-jet flight activity from the **JETNET API across 30+ airports**, then auto-identifies and enriches high-value prospects — removing hours of manual flight-log review each day.
- Lead-ranking system scores aircraft owners/operators by data-completeness tiers + contact-quality score — **cutting manual lead research ~90%**.
- **AI-personalized email & SMS outreach** through a rate-limited queue for **10,000+ prospects per campaign**, with open/click/bounce tracking (SendGrid + Twilio).
- React dashboards with interactive maps, **live air-traffic data**, and a **HubSpot CRM sync** linking closed deals to specific aircraft and contacts.
- Interactive **What-If scenario planner** — model occupancy, fuel volume, hangar utilization, and traffic to forecast revenue impact in real time.

### 🥃 Hooten Young — AI Sales & Marketing Intelligence Platform
An internal analytics platform for a premium American spirits brand that turns
weekly wholesale sales & depletion data into reporting the team used to assemble
by hand — **saving ~8–10 hours per week**.
- Coverage-gap, distributor-concentration-risk, and cross-sell analytics that surface growth opportunities and at-risk accounts previously invisible.
- Field-sales CRM generating **a prioritized daily call list per rep** across **5 user roles**.
- Automated weekly job analyzing **6,000+ social posts and 80+ GB of media** through a **multi-model LLM pipeline that routes each task to the best-fit provider** for cost and quality — producing content briefs and drafts.
- **TTB compliance review** on every AI-generated asset before it reaches the team, removing a manual legal review from the workflow.

### 📧 Outreach — AI Email Campaign Platform
A **multi-tenant** platform that automates product-marketing outreach at scale
across multiple brands and workspaces.
- **AI drafting** generates and refines on-brand campaign emails from a short prompt — **cutting campaign creation from 30–45 minutes to a couple of minutes**.
- Scheduled, throttled "drip" sending with audience targeting (tag / industry / company), timezone-aware delivery, and daily caps — hands-off campaigns that never email a contact twice.
- End-to-end engagement tracking (opens/clicks/bounces) with machine-open filtering, domain suppression, and contact-quality ranking — protecting sender reputation and giving real ROI visibility.

---

## 📂 Selected Case Studies

Deep-dives on the architecture, engineering challenges, and trade-offs:

- **[Ask Velocity — AI Demand Platform for FBOs](./case-studies/ask-velocity.md)** — Multi-tenant SaaS, JETNET ingestion, lead ranking, AI outreach, live CRM sync, What-If planner
- **[Hooten Young — Sales & Marketing Intelligence](./case-studies/hooten-young.md)** — Idempotent ingestion, field CRM, multi-model LLM pipeline, compliance-gated generation
- **[Outreach — AI Email Campaign Platform](./case-studies/outreach-platform.md)** — AI drafting, durable rate-limited sending, engagement tracking

---

## 🛠️ Tech Stack

**Languages** · Python · JavaScript · TypeScript · SQL · Bash

**Backend & APIs** · FastAPI · Flask · Django · REST · GraphQL · SQLAlchemy · async & event-driven microservices

**Frontend** · React · Vue.js · MUI · Recharts / ECharts · Leaflet

**AI / ML** · Claude ecosystem (Claude Code, Cowork) · OpenAI · Google Gemini · Vertex AI · LangChain · LangGraph · RAG & pgvector · Pandas · NumPy

**Databases & Caching** · PostgreSQL (PostGIS / pgvector) · MSSQL · Oracle · MongoDB · DynamoDB · Redis

**Cloud & DevOps** · AWS (Lambda, API Gateway, EC2, S3, RDS, DynamoDB Streams, SNS/SQS, EventBridge, CloudWatch, OpenSearch, Route 53) · GCP (Cloud Run, Cloud Tasks, Scheduler, GCS) · Docker · Kubernetes · GitHub Actions · GitLab CI/CD · Jenkins

**Messaging, Scheduling & Integrations** · Celery · RabbitMQ · ActiveBatch · SendGrid · Twilio · HubSpot · JETNET / FAA · FTP

**Practices & Tools** · TDD / PyTest · Agile · Git · Jira · Confluence

---

## 💼 Experience

- **Lead Software Engineer — Pilot Company (Pilot Flying J)** _(Jun 2021 – Present)_ — High-throughput logistics & fuel-delivery backends (TRIPS) supporting **millions of daily transactions**; architected an end-to-end automated fuel-allocation pipeline (DTN TABS via FTP) replacing a manual, error-prone process. AWS serverless (Lambda, API Gateway, DynamoDB, SNS/SQS, RDS).
- **Software Engineer — BASF Enzymes** _(Aug 2017 – Jun 2021)_ — Led the High-Throughput Screening dashboard and pipeline automation, **eliminating manual Excel workflows and reducing human error 90%+**; Riffyn integration Execution Engine; Celery/RabbitMQ pipelines; Docker/K8s/GitLab CI.
- **Application Developer — Kwikee** _(Nov 2016 – Aug 2017)_ — Full-stack PIM work (Hy-Vee Vendor Portal) and GDSN ingestion pipelines cutting manual effort **80%+**.

**Education** · M.S. Computer Information Systems & IT, University of Central Missouri · B.Tech ECE, K L University

**Certifications** · AWS Certified Developer – Associate · AWS Certified Cloud Practitioner · Python for Data Science (Cornell)

---

<sub>Case studies describe architecture and engineering approach. Proprietary
client data and credentials are intentionally omitted.</sub>
