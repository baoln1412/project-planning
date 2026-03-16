# Technology Stack: AI Agentic UA Engine

> **Evaluated**: 2026-03-13
> **Status**: Complete

## Tech Stack Summary

| Layer | Technology | Role | License | Decision |
|---|---|---|---|---|
| **Orchestration** | n8n (Internal Server) | Workflow engine, API calls, scheduling | Enterprise | ✅ ADR-010 |
| **User Interface** | Web App Console | Control plane, dashboard, campaign management | Internal | ✅ ADR-011 |
| **Backend State** | Internal PostgreSQL | Campaign state, creative metadata, audit logs | Enterprise | ✅ ADR-013 |
| **Asset Storage** | Internal S3 / MinIO | Media files (creatives) storage | Enterprise | ✅ ADR-005 |
| **Analytics Layer**| Spark + Iceberg | LTV modeling, attribution analysis | Enterprise | ✅ ADR-007 |
| **AI Brain** | Direct LLM API | ReAct prompting, data analysis (Simple) | Commercial | ✅ ADR-012 |
| **LLM** | Claude 3.5 Sonnet | Reasoning and analysis | Commercial API | ✅ ADR-012 |
| **Creative Gen** | Creatify, OpenArt.ai, etc. | Video/image generation | Commercial API | ✅ ADR-014 |
| **Notifications** | Slack + Telegram | HITL alerts and approvals | Commercial | — |
| **Auth/Identity** | Internal Auth (SSO/LDAP) | User access control for n8n/Sheets | Enterprise | Existing |
| **Data Standards** | [`game-publishing-data-architect`](../../.agents/skills/Analytics%20%26%20Data/game-publishing-data-architect/SKILL.md) | Log schemas, metrics definitions, data architecture | Internal Skill | ✅ ADR-015 |

---

## Layer Evaluations

### Layer 1: Orchestration — n8n

#### ADR-010: n8n Cloud as Workflow Orchestrator

- **Status**: ✅ Accepted
- **Context**: Need a workflow engine to schedule cron jobs, call ad platform APIs, and route data. Want to avoid infrastructure management.
- **Decision**: **n8n Cloud** — Eliminates server maintenance, team has experience, handles scaling/queuing natively.
- **Consequences**:
  - ✅ Fast development, zero infra maintenance
  - ⚠️ Monthly SaaS cost ($20-50/mo)
  - ⚠️ Dependent on n8n Cloud availability (covered by 99.9% SLA)

---

### Layer 2: AI Brain — Dify

#### ADR-011: Web App Console as User Interface

- **Status**: ✅ Accepted (Updated)
- **Context**: Need a purpose-built interface with premium UX for the UA team — dashboards, campaign management, pLTV analytics.
- **Decision**: **Web App Console** — Custom dark-themed web application backed by PostgreSQL, communicating with n8n via REST API.
- **Consequences**:
  - ✅ Premium UX with real-time dashboards, AI agent activity feeds, and campaign management
  - ✅ No scalability limits — backed by PostgreSQL
  - ✅ Better data validation and user workflows than spreadsheet
  - ⚠️ Requires frontend development (mitigated by using modern framework + Stitch for rapid UI design)

---

### Layer 3: LLM Selection

#### ADR-012: Claude Sonnet 4 Primary, GPT-4o Fallback

- **Status**: ✅ Accepted
- **Context**: Agents need strong reasoning (budget analysis, audience targeting), structured JSON output, and low hallucination risk
- **Candidates Evaluated**:

| Criteria | Claude Sonnet 4 | GPT-4o | Gemini 2.5 Pro |
|---|---|---|---|
| Reasoning quality | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Structured output | ✅ Excellent | ✅ Good | ✅ Good |
| Cost (per 1M tokens) | ~$3 in / $15 out | ~$2.50 in / $10 out | ~$1.25 in / $5 out |
| Hallucination risk | Low | Medium | Medium |
| Tool use | ✅ Strong | ✅ Strong | ✅ Strong |

- **Decision**: **Claude Sonnet 4** as primary LLM for all agents. GPT-4o as fallback if rate-limited. Both supported natively by Dify.
- **Consequences**:
  - ✅ Strong reasoning reduces hallucination risk on budget decisions
  - ✅ Dual-LLM strategy prevents single-vendor downtime
  - ⚠️ API costs (~$0.01-0.05 per agent call, estimated $50-150/mo at scale)

---

### Layer 4: Data & Queuing

#### ADR-013: PostgreSQL + Redis for State and Queuing

- **Status**: ✅ Accepted
- **Context**: Need persistent storage for campaign configs, creative metadata, audit logs. Need job queue for workflow scheduling.
- **Decision**: PostgreSQL as primary database (campaign state, audit trail, creative metadata). Redis for job queuing, caching, and rate limit counters.
- **Alternatives**: SQLite (too limited), MongoDB (overkill), MySQL (PostgreSQL has better JSON support).
- **Consequences**:
  - ✅ Both are battle-tested, OSS, lightweight
  - ✅ PostgreSQL JSONB for flexible schema (campaign configs vary by platform)
  - ✅ Redis for fast rate limit counting and API response caching
  - ⚠️ Adds 2 services to Docker Compose stack

---

### Layer 5: Creative Production

#### ADR-014: 6-Tool GenAI Creative Pipeline

- **Status**: ✅ Accepted
- **Context**: Need to produce 50-100 creative variants per week across video and image formats
- **Tool Selection**:

| Tool | Primary Use Case | Output | Pricing | Sprint |
|---|---|---|---|---|
| **Creatify** | UGC videos, 700+ AI avatars, URL→video | Video | Freemium, ~$49/mo | 2 |
| **OpenArt.ai** | Text→image, A/B variants, brand consistency | Image | Freemium, ~$12/mo | 2 |
| **Apptovid** | App Store URL→video ads | Video | Custom pricing | 3 |
| **Veo3** (Google) | Cinematic hero/trailer videos | Video | API-based | 4 |
| **Viggle AI** | 2D→3D character animation | Video | Freemium | 4 |
| **Runway Gen-4** | Image/text→gameplay action clips | Video | ~$12-76/mo | 4 |

- **Decision**: Start with **Creatify + OpenArt.ai** in Sprint 2 (both have free tiers). Add remaining tools as the pipeline scales.
- **Consequences**:
  - ✅ Low initial cost (free tiers for Sprint 2 pilot)
  - ✅ 6 tools = diverse creative types (UGC, cinematic, character animation, gameplay)
  - ⚠️ Each tool has different API patterns — need per-tool n8n nodes
  - ⚠️ Output quality varies — need quality gate before deployment

---

### Layer 6: Deployment & Operations

#### ADR-015: Docker Compose (Dev) → Docker Swarm (Production)

- **Status**: ✅ Accepted
- **Context**: Need containerized deployment for n8n + Dify + PostgreSQL + Redis. Local dev, company server for production.
- **Decision**: Docker Compose for local development. Docker Swarm for company server production (simpler than Kubernetes for this team size).
- **Consequences**:
  - ✅ Single `docker-compose.yml` for entire stack
  - ✅ Docker Swarm is simple enough for 1-2 engineers to maintain
  - ⚠️ If scaling beyond 10 games, may need Kubernetes

---

## Compatibility Matrix

| | n8n | Dify | PostgreSQL | Redis | OpenClaw |
|---|---|---|---|---|---|
| **n8n** | — | ✅ HTTP/Webhook | ✅ Native node | ✅ Native node | ✅ HTTP |
| **Dify** | ✅ HTTP/Webhook | — | ✅ Native | ❌ N/A | ✅ MCP |
| **PostgreSQL** | ✅ Native | ✅ Native | — | ❌ N/A | ❌ N/A |
| **Redis** | ✅ Native | ❌ N/A | ❌ N/A | — | ❌ N/A |
| **OpenClaw** | ✅ HTTP | ✅ MCP | ❌ N/A | ❌ N/A | — |

All components communicate via HTTP/REST or direct database connections. No complex messaging middleware needed.

---

## Cost Estimation

| Layer | Component | Dev (Local) | Production (Company Server) | Scale (10 games) |
|---|---|---|---|---|
| Orchestration | n8n | $0 | $0 (self-hosted) | $0 |
| AI Brain | Dify | $0 | $0 (self-hosted) | $0 |
| LLM APIs | Claude / GPT-4o | ~$5/mo | ~$50-150/mo | ~$300-500/mo |
| GenAI Tools | Creatify + OpenArt.ai | Free tier | ~$60/mo | ~$200/mo |
| Database | PostgreSQL + Redis | $0 | $0 (self-hosted) | $0 |
| Hosting | Docker Compose | $0 (local) | $50-100/mo (server share) | $200-500/mo |
| Ad Spend | TikTok pilot | $500/wk | $2-5K/wk | $10-50K/wk |
| Attribution | AppsFlyer | Existing | Existing | Existing |
| **Total (excl. ad spend)** | | **~$5/mo** | **~$160-310/mo** | **~$700-1200/mo** |

---

## Team Capability Fit

| Technology | Team Experience | Learning Curve | Risk |
|---|---|---|---|
| n8n | ✅ High | None | Low |
| Dify | ✅ High | None | Low |
| Docker Compose | ✅ Assumed | Low | Low |
| PostgreSQL | ✅ Common | Low | Low |
| Redis | ⚠️ Some | Low | Low |
| OpenClaw / MCP | ❌ New | Medium | Medium |
| Ad Platform APIs | ⚠️ Some | Medium | Medium |
| GenAI tool APIs | ⚠️ Some | Low-Medium | Low |

**Overall**: Team is well-positioned. Only OpenClaw/MCP is genuinely new territory (covered by ADR-005 fallback strategy).

---

## pLTV Extension: Two-Branch Technology Stack (ADR-016)

> **Added**: 2026-03-16
> **Status**: Accepted — extends Sprint 6-8

### Branch A: Internal pLTV Model Stack

| Layer | Technology | Role | Difficulty | Who |
|---|---|---|---|---|
| **Feature Pipeline** | Spark SQL + PySpark | D0-D3 feature extraction from Lakehouse | ⭐⭐ Medium | Data Engineer |
| **Probabilistic CLV** | Python `lifetimes` | BG-NBD (frequency/churn) + Gamma-Gamma (monetary) | ⭐ Easy | Data Analyst |
| **Payer Classifier** | XGBoost / LightGBM | D0-D3 behavioral features → payer probability | ⭐⭐ Medium | Data Analyst |
| **Revenue Forecast** | Prophet (Meta) | Cohort revenue curve extrapolation | ⭐ Easy | Data Analyst |
| **Retention Model** | scikit-survival (`sksurv`) | Kaplan-Meier + CoxPH for churn timing | ⭐⭐ Medium | Data Analyst |
| **Model Serving** | PySpark UDF (batch) | Daily scoring job on Spark/Iceberg | ⭐⭐ Medium | Data Engineer |
| **S2S Integration** | Meta CAPI, Google S2S, TikTok Events API | Push Synthetic Events (pLTV as `valueToSum`) | ⭐⭐ Medium | Data Engineer |

**Why these libraries work without a data scientist:**
- `lifetimes`: Literally 5-10 lines → `BetaGeoFitter().fit()` → `predict()`
- `XGBoost`: scikit-learn compatible API, defaults work well on tabular data
- `Prophet`: Input is just `(ds, y)` — dates and values, auto-handles seasonality
- `sksurv`: Standard survival analysis, interpretable coefficients

#### Branch A Cost Estimate

| Component | Monthly Cost | Notes |
|---|---|---|
| Spark compute (incremental) | $0 | Uses existing enterprise Spark cluster |
| Python libraries | $0 | All open-source |
| S2S API calls | $0 | Free (Meta CAPI, Google S2S) |
| **Total** | **$0** | All infrastructure already exists |

---

### Branch B: External Vendor Stack

| Layer | Technology | Role | Integration | Who |
|---|---|---|---|---|
| **UA Optimization** | AppLovin Axon 2.0 | Predictive bidding, ROAS optimization, LTV-based targeting | SDK + MMP | Data Engineer |
| **Monetization + UA** | Appodeal AutoBid | Real-time unified auction, LTV forecasting (365d), budget automation | Single SDK | Data Engineer |
| **Attribution** | AppsFlyer (existing) | MMP event forwarding to both vendors | Already integrated | — |

**Key integration requirements (MMP events to forward):**
- `Install` event
- `tutorial_complete` (or equivalent early progression signal)
- `Purchase` (with revenue value parameter)
- `Ad Impression` (with ad revenue value parameter)

#### Branch B Cost Estimate

| Component | Monthly Cost | Notes |
|---|---|---|
| AppLovin Axon 2.0 | Performance-based | % of ad spend (included in media cost) |
| Appodeal AutoBid | Revenue share | % of ad revenue (typically 10-20%) |
| SDK maintenance | $0 | Standard game SDK updates |
| **Total** | **~Performance-based** | No fixed monthly fees |

---

### Branch Comparison

| Criteria | Branch A (Internal) | Branch B (External Vendors) |
|---|---|---|
| **Control** | Full — you own the model | Limited — vendor black box |
| **Cross-platform** | Yes — works on all ad platforms | No — each vendor optimizes own network |
| **Build time** | 4-8 weeks | 2-3 weeks |
| **Maintenance** | Monthly recalibration | Vendor-managed |
| **Data requirement** | 2-3 years historical (available ✅) | 14-day warmup per campaign |
| **Cost** | $0 (existing infra) | Performance-based revenue share |
| **Team needed** | Data Analyst + Data Engineer | Data Engineer |
| **Meta VO support** | ✅ via Synthetic Events | ❌ No Meta VO integration |
| **Vendor lock-in** | None | Yes |
