# Technology Stack: AI Agentic UA Engine

> **Evaluated**: 2026-03-13
> **Status**: Complete

## Tech Stack Summary

| Layer | Technology | Role | License | Decision |
|---|---|---|---|---|
| **Orchestration** | n8n (Internal Server) | Workflow engine, API calls, scheduling | Enterprise | ✅ ADR-010 |
| **User Interface** | Google Sheets | Control plane, dashboard, configuration | Google Workspace | ✅ ADR-011 |
| **Backend State** | Internal PostgreSQL | Campaign state, creative metadata, audit logs | Enterprise | ✅ ADR-013 |
| **Asset Storage** | Internal S3 / MinIO | Media files (creatives) storage | Enterprise | ✅ ADR-005 |
| **Analytics Layer**| Spark + Iceberg | LTV modeling, attribution analysis | Enterprise | ✅ ADR-007 |
| **AI Brain** | Direct LLM API | ReAct prompting, data analysis (Simple) | Commercial | ✅ ADR-012 |
| **LLM** | Claude 3.5 Sonnet | Reasoning and analysis | Commercial API | ✅ ADR-012 |
| **Creative Gen** | Creatify, OpenArt.ai, etc. | Video/image generation | Commercial API | ✅ ADR-014 |
| **Notifications** | Slack + Telegram | HITL alerts and approvals | Commercial | — |
| **Auth/Identity** | Internal Auth (SSO/LDAP) | User access control for n8n/Sheets | Enterprise | Existing |

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

#### ADR-011: Google Sheets as Interface and Database

- **Status**: ✅ Accepted
- **Context**: Need a single point of control that is understandable and editable by the UA team.
- **Decision**: **Google Sheets** — Direct interface for campaign planning, metrics dashboard, and audit logging.
- **Consequences**:
  - ✅ UA team already knows the tool; zero training
  - ✅ Fully transparent and easy to share/filter
  - ⚠️ Scalability limits (~10M cells) — enough for pilot and 3-5 games
  - ⚠️ Manual edits can break n8n watchers (mitigated by sheet protection/validation)

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
