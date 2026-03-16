# Technology Stack: AI Agentic UA Engine

> **Evaluated**: 2026-03-13
> **Status**: Complete

## Tech Stack Summary

| Layer | Technology | Role | License | Decision |
|---|---|---|---|---|
| **Orchestration** | n8n (self-hosted) | Workflow engine, API calls, scheduling | Fair-code | ✅ ADR-010 |
| **AI Brain** | Dify (self-hosted) | ReAct agents, RAG, LLM orchestration | Apache 2.0 | ✅ ADR-011 |
| **MCP Gateway** | OpenClaw | API abstraction, protocol standardization | Apache 2.0 | ✅ ADR-005 |
| **LLM** | Claude Sonnet 4 (primary), GPT-4o (fallback) | Agent reasoning, brief writing, data analysis | Commercial API | ✅ ADR-012 |
| **Data Pipeline** | Existing AppsFlyer pipeline | Attribution, events, transactions (real-time) | Commercial | ✅ ADR-007 |
| **Database** | PostgreSQL + Redis | Campaign state, job queues, caching | OSS | ✅ ADR-013 |
| **Creative Gen** | Creatify, OpenArt.ai, Apptovid, Veo3, Viggle, Runway | Video/image generation | Commercial API | ✅ ADR-014 |
| **Bidding** | AppLovin Axon 2.0, Appodeal AutoBid | LTV-based bid optimization | Commercial | ✅ ADR-003 |
| **Attribution** | AppsFlyer | Install attribution, in-app events | Commercial | Existing |
| **Notifications** | Slack + Telegram | HITL alerts and approvals | Commercial | — |
| **Containerization** | Docker Compose → Docker Swarm | Dev → Production deployment | OSS | ✅ ADR-015 |
| **Monitoring** | n8n execution logs + Slack alerts | Workflow health, error tracking | — | — |

---

## Layer Evaluations

### Layer 1: Orchestration — n8n

#### ADR-010: n8n as Workflow Orchestrator

- **Status**: ✅ Accepted
- **Context**: Need a workflow engine to schedule cron jobs, call ad platform APIs, route data between services, and handle retries
- **Candidates Evaluated**:

| Criteria | n8n | Make | Temporal.io |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Self-hosted | ✅ Yes | ❌ SaaS only | ✅ Yes |
| Visual builder | ✅ Yes | ✅ Yes | ❌ Code-only |
| API integrations | ✅ 400+ nodes | ✅ 1000+ | ⚠️ Code-based |
| Error handling | ✅ Built-in retry | ✅ Built-in | ✅ Advanced |
| Cost | Free (self-hosted) | $99-599/mo | Free (self-hosted) |
| Team familiarity | ✅ High | ⚠️ Low | ❌ None |
| Scalability | ⚠️ Medium | ⚠️ Limited by plan | ✅ Excellent |
| DX | ✅ Good | ✅ Good | ⚠️ Steep curve |

- **Decision**: **n8n** — Team has experience, self-hosted = free, visual builder accelerates development. For this project's scale (1-5 games), n8n's capacity is sufficient.
- **Migration path**: If n8n hits scaling limits at 10+ games, evaluate Temporal.io for critical high-volume workflows.
- **Consequences**:
  - ✅ Zero SaaS cost, full control
  - ✅ Fast development with visual builder
  - ⚠️ Fair-code license restricts reselling (not a concern for internal use)
  - ⚠️ Single-threaded execution — may need worker scaling at volume

---

### Layer 2: AI Brain — Dify

#### ADR-011: Dify as AI Reasoning Platform

- **Status**: ✅ Accepted
- **Context**: Need an LLM orchestration platform for ReAct agents (Campaign Strategist, Campaign Monitor), RAG knowledge base, and structured JSON output
- **Candidates Evaluated**:

| Criteria | Dify | LangChain/LangGraph | AutoGen (Microsoft) |
|---|---|---|---|
| Self-hosted | ✅ Yes | ✅ Yes (library) | ✅ Yes |
| Visual agent builder | ✅ Yes | ❌ Code-only | ❌ Code-only |
| RAG built-in | ✅ Yes | ⚠️ Needs setup | ❌ No |
| ReAct support | ✅ Yes | ✅ Yes | ✅ Yes |
| Multi-agent | ⚠️ Limited | ✅ LangGraph | ✅ Yes |
| Structured output | ✅ JSON | ✅ JSON | ✅ JSON |
| Team familiarity | ✅ High | ⚠️ Low | ❌ None |
| Dev effort | Low | High | High |

- **Decision**: **Dify** — Low-code agent building, built-in RAG, team familiarity. Faster time-to-value than code-first alternatives.
- **Consequences**:
  - ✅ Agents built in days, not weeks
  - ✅ Built-in knowledge base for campaign data
  - ⚠️ Less flexibility than LangGraph for complex multi-agent flows
  - ⚠️ Dependent on Dify's agent capabilities (may need custom tools)

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
