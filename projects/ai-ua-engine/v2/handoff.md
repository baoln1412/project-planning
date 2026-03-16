# Technical Handoff: AI Agentic UA Engine

> **Version**: 1.0
> **Date**: 2026-03-13
> **For**: Engineering team implementing Sprint 1

## Project Overview

Build an AI-powered UA Engine that automates campaign planning, creative production, deployment, and monitoring across Facebook, Google, and TikTok for mobile gaming. The system uses n8n (orchestration), Dify (AI reasoning), and an existing AppsFlyer data pipeline (analytics).

**North Star**: Fully automated closed-loop UA — from data analysis to live campaign in <15 minutes.

> **📚 Data Reference**: All data schemas, metrics definitions, and log requirements follow the
> [`game-publishing-data-architect`](../../.agents/skills/Analytics%20%26%20Data/game-publishing-data-architect/SKILL.md) skill.
> Consult it for AppsFlyer schemas, in-game log formats, metric formulas, and data architecture standards.

### Required Data Sources (4 inputs)

| # | Data Source | Provider | Data |
|---|-----------|----------|------|
| 1 | **Marketing Data** | Facebook, Google, TikTok Ads | Cost, Campaign Info, Ads Performance |
| 2 | **Event Tracking** | AppsFlyer (MMP) | Installs, In-app Events, Attribution |
| 3 | **In-game Data** | Game Server (Kafka/Rsync) | Register, Login, Logout, Recharge, CCU |
| 4 | **Platform Data** | VNG SDK | SDK Login + Billing/Payment |

---

## Repository Structure

```
ai-ua-engine/
├── docker-compose.yml          # All services (n8n, Dify, PG, Redis)
├── docker-compose.prod.yml     # Production overrides
├── .env.example                # Environment variable template
├── README.md                   # Setup instructions
│
├── n8n/
│   ├── workflows/              # Exported n8n workflow JSONs
│   │   ├── campaign-planning.json
│   │   ├── creative-production.json
│   │   ├── tiktok-deploy.json
│   │   ├── facebook-deploy.json
│   │   ├── google-deploy.json
│   │   ├── spend-data-pull.json
│   │   ├── monitor-trigger.json
│   │   ├── hitl-notification.json
│   │   ├── daily-summary.json
│   │   └── token-refresh.json
│   └── credentials/            # Credential setup docs (NOT actual keys)
│
├── dify/
│   ├── agents/
│   │   ├── campaign-strategist.yml   # ReAct agent config
│   │   ├── campaign-monitor.yml      # ReAct agent config
│   │   └── creative-analyst.yml      # CoT agent config
│   └── knowledge-base/
│       └── README.md                 # How to load historical data
│
├── db/
│   └── migrations/
│       ├── 001_create_campaigns.sql
│       ├── 002_create_creatives.sql
│       ├── 003_create_agent_decisions.sql
│       └── 004_create_api_errors.sql
│
├── docs/
│   ├── architecture.md         # C4 diagrams and data contracts
│   ├── runbook.md              # Operational troubleshooting
│   └── api-reference.md        # Ad platform API cheat sheet
│
└── scripts/
    ├── setup.sh                # First-time setup script
    ├── backup-db.sh            # PostgreSQL backup
    └── import-knowledge.sh     # Load data into Dify KB
```

---

## Tech Stack

| Component | Technology | Version | Port |
|---|---|---|---|
| Orchestrator | n8n | Latest stable | 5678 |
| AI Brain | Dify | Latest stable | 5001 (API), 3000 (Web) |
| Database | PostgreSQL | 16+ | 5432 |
| Cache/Queue | Redis | 7+ | 6379 |
| MCP Gateway | OpenClaw | Latest | 8080 |
| Runtime | Docker + Docker Compose | 24+ | — |

---

## Environment Setup

### Prerequisites
- Docker Desktop (or Docker Engine + Compose)
- API keys: TikTok Ads, Claude/OpenAI, Creatify, OpenArt.ai
- Access to company AppsFlyer data pipeline endpoint

### Quick Start

```bash
# 1. Clone and configure
cp .env.example .env
# Edit .env with your API keys

# 2. Start all services
docker compose up -d

# 3. Run database migrations
docker compose exec postgres psql -U ua_engine -d ua_engine -f /migrations/001_create_campaigns.sql

# 4. Import knowledge base
./scripts/import-knowledge.sh

# 5. Access services
# n8n:  http://localhost:5678
# Dify: http://localhost:3000
```

### Environment Variables

```env
# -- LLM --
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# -- Ad Platforms --
TIKTOK_APP_ID=...
TIKTOK_ACCESS_TOKEN=...
FACEBOOK_APP_ID=...
FACEBOOK_APP_SECRET=...
FACEBOOK_ACCESS_TOKEN=...
GOOGLE_ADS_DEVELOPER_TOKEN=...
GOOGLE_ADS_CLIENT_ID=...
GOOGLE_ADS_CLIENT_SECRET=...
GOOGLE_ADS_REFRESH_TOKEN=...

# -- GenAI Tools --
CREATIFY_API_KEY=...
OPENART_API_KEY=...

# -- AppsFlyer --
APPSFLYER_WEBHOOK_URL=...

# -- Database --
POSTGRES_USER=ua_engine
POSTGRES_PASSWORD=...
POSTGRES_DB=ua_engine

# -- HITL --
SLACK_WEBHOOK_URL=...
SLACK_BOT_TOKEN=...
```

---

## Database Schema

### Core Tables

```sql
-- Campaign configurations
CREATE TABLE campaigns (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id VARCHAR(50) NOT NULL,
    platform VARCHAR(20) NOT NULL, -- facebook, google, tiktok
    status VARCHAR(20) DEFAULT 'draft',
    config JSONB NOT NULL, -- CampaignPlan JSON
    platform_campaign_id VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Creative asset metadata
CREATE TABLE creatives (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES campaigns(id),
    game_id VARCHAR(50) NOT NULL,
    tool VARCHAR(50) NOT NULL, -- creatify, openart, etc.
    type VARCHAR(10) NOT NULL, -- video, image
    file_path VARCHAR(500),
    brief JSONB,
    tags JSONB DEFAULT '[]',
    performance JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI decision audit trail (append-only)
CREATE TABLE agent_decisions (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    agent VARCHAR(30) NOT NULL, -- strategist, monitor, analyst
    action VARCHAR(30) NOT NULL, -- pause, scale, refresh, launch, alert
    tier VARCHAR(20) NOT NULL, -- full_auto, semi_auto, human_only
    campaign_id UUID REFERENCES campaigns(id),
    reasoning TEXT NOT NULL,
    data_inputs JSONB NOT NULL,
    outcome VARCHAR(20) NOT NULL, -- executed, pending_approval, rejected
    approved_by VARCHAR(100),
    pre_change_state JSONB -- for rollback capability
);

-- API error log
CREATE TABLE api_errors (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    platform VARCHAR(20) NOT NULL,
    endpoint VARCHAR(200),
    error_code VARCHAR(50),
    error_message TEXT,
    retry_count INT DEFAULT 0,
    resolved BOOLEAN DEFAULT FALSE
);
```

---

## Data Contracts

### Input: AppsFlyer Data (from existing pipeline)

Two primary tables — full schemas defined in [`game-publishing-data-architect`](../../.agents/skills/Analytics%20%26%20Data/game-publishing-data-architect/SKILL.md) skill, Section 3:

| Table | Key Fields | Use Case |
|-------|-----------|----------|
| `install` | `appsflyer_id`, `media_source`, `campaign`, `af_cost_value`, `install_time`, `customer_user_id` | Attribution, CPI, Install counts |
| `in_app_event` | `event_name`, `event_revenue_usd`, `event_time`, `customer_user_id` | Revenue tracking, LTV, ROAS |

**ID mapping**: AppsFlyer `customer_user_id` → In-game `user_id`

### Input: In-Game Logs (via Kafka/Rsync)

Full schemas defined in skill Section 4:

| Log | Key Fields | Use Case |
|-----|-----------|----------|
| Register | `user_id`, `role_id`, `register_time`, `device_id` | NRU calculation |
| Login | `user_id`, `login_time`, `vip_level`, `fighting_power` | DAU, session tracking |
| Logout | `user_id`, `logout_time`, `online_time` | Session duration, compliance |
| Recharge | `user_id`, `transaction_id`, `recharged_value`, `currency_code` | Revenue, PU, LTV |
| CCU | `event_time`, `server_id`, `online_users` | ACU/PCU/LCU |
| Money Flow | `user_id`, `money_type`, `flow_type`, `before_value`, `after_value` | Economy audit |
| Item Flow | `user_id`, `item_id`, `item_quantity`, `item_price` | Item economy |

### Output: CampaignPlan (AI → n8n)
See [architecture.md](./architecture.md#data-contracts) for full schema.

### Output: AgentDecision (audit log)
See [architecture.md](./architecture.md#data-contracts) for full schema.

---

## Key Architecture Decisions

| ADR | Decision | Rationale |
|---|---|---|
| ADR-001 | 3-layer architecture (n8n + Dify + OpenClaw) | Decoupled, each layer specialized |
| ADR-007 | AppsFlyer pipeline as primary data source | 75-85% fewer ad API calls |
| ADR-010 | n8n (self-hosted) | Team experience, $0 cost |
| ADR-012 | Claude Sonnet 4 primary LLM | Best reasoning, low hallucination |
| ADR-013 | PostgreSQL + Redis | State + queue + cache |

See [findings.md](./findings.md#architecture-decision-records-adrs) for all 15 ADRs.

---

## Anti-Hallucination Guardrails (CRITICAL)

These are **deterministic rules** — never implemented as LLM prompts:

1. **Daily budget ceiling**: n8n workflow checks `budget_daily_max` before executing any budget change
2. **% change cap**: Auto-actions limited to ≤20%. Semi-auto (Slack approval) for >20%
3. **Data threshold**: Agent must have ≥24hr data before optimizing
4. **Schema validation**: n8n validates Dify JSON output against expected schema before any API call
5. **Confidence gate**: Agent outputs confidence score. Below 0.7 → requires HITL
6. **Rollback**: All changes logged with `pre_change_state` for instant rollback

---

## Sprint 1 Checklist (Start Here)

```
Week 1:
□ docker-compose.yml working (n8n + Dify + PG + Redis)
□ TikTok API credentials configured in n8n
□ AppsFlyer webhook connected to n8n
□ PostgreSQL schema migrated (4 tables)
□ Historical data loaded into Dify KB

Week 2:
□ Campaign Strategist agent built in Dify
□ Campaign Planning workflow built in n8n
□ Token refresh workflow built
□ Integration test: trigger → Dify → plan JSON stored in PG
□ Demo to stakeholder ✅
```

---

## CI/CD Recommendation

| Stage | Tool | Approach |
|---|---|---|
| Dev | Docker Compose | Local development |
| Staging | Same server, separate compose file | Test with sandbox APIs |
| Production | Docker Swarm | Deploy from Git with `docker stack deploy` |
| Backup | CRON + pg_dump | Daily PostgreSQL backup |
| Monitoring | n8n execution logs + Slack alerts | No separate monitoring stack needed initially |

---

## Reference Links

- [Architecture](./architecture.md) — C4 diagrams, sequence flows, security
- [Tech Stack](./tech-stack.md) — Technology evaluations and ADRs
- [Implementation Plan](./implementation-plan.md) — Sprint tasks and Gantt chart
- [Research](./research.md) — Build-vs-buy, API feasibility
- [Findings](./findings.md) — All ADRs, risk log, session notes
