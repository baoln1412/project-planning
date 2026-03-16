# Architecture: AI Agentic UA Engine

> **Architect**: Solutions Architect (AI-assisted)
> **Date**: 2026-03-13
> **Status**: Complete

## Data Contracts (Data-First)

### Campaign Plan (Dify Agent → n8n)
```json
{
  "game_id": "total-football",
  "platform": "tiktok",
  "budget_allocation": { "daily": 71, "total": 500 },
  "target_audience": {
    "geos": ["TH", "VN", "ID"],
    "age_min": 18, "age_max": 45,
    "interests": ["football", "sports games", "mobile gaming"]
  },
  "creative_brief": {
    "theme": "competitive gameplay",
    "style": "UGC",
    "cta": "Play Now",
    "variants_count": 10
  },
  "bidding_strategy": { "type": "CPI", "target": 0.50 }
}
```

### Agent Decision (Audit Log)
```json
{
  "timestamp": "2026-03-13T10:00:00Z",
  "agent": "monitor",
  "action": "pause",
  "tier": "full_auto",
  "campaign_id": "tiktok_tf_001",
  "reasoning": "CTR dropped below 0.5% threshold for 3 consecutive hours",
  "data_inputs": { "ctr": 0.32, "cpi": 1.20, "threshold_ctr": 0.5 },
  "outcome": "executed"
}
```

### Creative Asset Metadata
```json
{
  "id": "creative_001",
  "game_id": "total-football",
  "tool": "creatify",
  "type": "video",
  "url": "s3://creatives/tf/creative_001.mp4",
  "brief_id": "brief_001",
  "tags": ["UGC", "gameplay", "competitive", "green-CTA"],
  "performance": { "cpi": 0.45, "roas": 2.8, "ctr": 1.2, "spend": 150 }
}
```

---

## C4 Level 1: System Context

```mermaid
graph TB
    subgraph Users
        UA["UA Manager"]
        CD["Creative Director"]
    end

    UAEngine["AI Agentic UA Engine"]

    subgraph External["External Systems"]
        AF["AppsFlyer Pipeline<br/>(Real-time Attribution)"]
        FB["Facebook Ads API"]
        GA["Google Ads API"]
        TT["TikTok Ads API"]
        GenAI["GenAI Tools<br/>(Creatify, OpenArt.ai, etc.)"]
        LLM["LLM APIs<br/>(Claude, GPT-4o)"]
        Bid["Bidding Systems<br/>(Axon, AutoBid)"]
        Slack["Slack / Telegram"]
    end

    UA -->|"Reviews, approves"| UAEngine
    CD -->|"Reviews briefs"| UAEngine
    AF -->|"Real-time events"| UAEngine
    UAEngine -->|"Campaign CRUD"| FB
    UAEngine -->|"Campaign CRUD"| GA
    UAEngine -->|"Campaign CRUD"| TT
    UAEngine -->|"Creative briefs"| GenAI
    UAEngine -->|"Agent reasoning"| LLM
    UAEngine -->|"Bid optimization"| Bid
    UAEngine -->|"HITL alerts"| Slack

    style UAEngine fill:#2d6a4f,color:#fff
    style AF fill:#1d3557,color:#fff
```

---

## C4 Level 2: Container Diagram

```mermaid
graph TB
    subgraph UAEngine["AI Agentic UA Engine"]
        N8N["n8n Orchestrator<br/>(Workflow Engine)<br/>Docker Container"]
        DIFY["Dify AI Brain<br/>(ReAct Agents + RAG)<br/>Docker Container"]
        PG["PostgreSQL<br/>(Campaign State + Audit)<br/>Docker Container"]
        REDIS["Redis<br/>(Queue + Cache + Rate Limits)<br/>Docker Container"]
        CLAW["OpenClaw Gateway<br/>(MCP Protocol)<br/>Docker Container"]
        FS["File Storage<br/>(Creative Assets)<br/>Docker Volume"]
    end

    AF["AppsFlyer Pipeline"] -->|"Events stream"| N8N
    N8N -->|"Agent calls<br/>(HTTP/Webhook)"| DIFY
    DIFY -->|"LLM calls"| LLM["Claude / GPT-4o"]
    N8N -->|"Campaign CRUD"| CLAW
    CLAW -->|"Standardized API"| ADS["Ad Platforms<br/>(FB, Google, TikTok)"]
    N8N -->|"Creative requests"| GenAI["GenAI Tools"]
    GenAI -->|"Assets"| FS
    N8N -->|"Write/Read"| PG
    N8N -->|"Queue/Cache"| REDIS
    DIFY -->|"RAG queries"| PG
    N8N -->|"HITL alerts"| Slack["Slack"]

    style N8N fill:#e76f51,color:#fff
    style DIFY fill:#2a9d8f,color:#fff
    style PG fill:#264653,color:#fff
    style REDIS fill:#e9c46a,color:#000
    style CLAW fill:#457b9d,color:#fff
```

---

## C4 Level 3: Key Components

### n8n Orchestrator — Workflow Components

| Workflow | Trigger | Input | Output | Frequency |
|---|---|---|---|---|
| **Campaign Planning** | Manual / CRON (weekly) | Game config + AppsFlyer data | Campaign Plan JSON → Dify | On-demand |
| **Creative Production** | Webhook (from Dify) | Creative Brief JSON | Media files → File Storage | On-demand |
| **Platform Deployment** | Webhook (from production) | Creative assets + campaign config | API calls to ad platforms | On-demand |
| **Spend Data Pull** | CRON (6hr/12hr) | Ad platform credentials | Spend data → PostgreSQL | 2-4x/day |
| **Monitor Trigger** | CRON (hourly peak / 4hr off-peak) | AppsFlyer events | Agent Decision → audit log | Hourly / 4hr |
| **Token Refresh** | CRON (daily) | Stored tokens | Refreshed tokens | Daily |
| **HITL Notification** | Webhook (from Dify) | Agent decision (semi-auto tier) | Slack message + approval buttons | On-demand |
| **Daily Summary** | CRON (daily 9am) | PostgreSQL audit logs | Slack digest message | Daily |

### Dify AI Brain — Agent Components

| Agent | Strategy | Knowledge Base | Input | Output |
|---|---|---|---|---|
| **Campaign Strategist** | ReAct (Reason-Act-Observe) | Historical campaigns, benchmarks, brand guidelines | Game ID + AppsFlyer metrics | Campaign Plan JSON |
| **Campaign Monitor** | ReAct | Performance thresholds, HITL rules | AppsFlyer real-time events + spend data | Agent Decision JSON |
| **Creative Analyst** | Chain-of-Thought | Creative performance history, winning patterns | Creative metadata + performance data | Tags + pattern insights |

---

## Sequence Diagrams

### Flow 1: Campaign Planning & Launch

```mermaid
sequenceDiagram
    participant UA as UA Manager
    participant N8N as n8n Orchestrator
    participant DIFY as Dify (Strategist Agent)
    participant LLM as Claude Sonnet 4
    participant GenAI as Creatify / OpenArt.ai
    participant TT as TikTok Ads API
    participant Slack as Slack

    UA->>N8N: Trigger campaign planning (game_id)
    N8N->>DIFY: Request campaign plan (game + metrics)
    DIFY->>LLM: ReAct: Analyze data, allocate budget
    LLM-->>DIFY: Campaign Plan JSON
    DIFY-->>N8N: Campaign Plan JSON
    N8N->>Slack: Notify UA Manager (review plan)
    UA->>Slack: Approve plan ✅
    Slack->>N8N: Approval webhook
    N8N->>GenAI: Send creative briefs (x10 variants)
    GenAI-->>N8N: Creative assets (videos/images)
    N8N->>TT: Create campaign + upload creatives
    TT-->>N8N: Campaign live ✅
    N8N->>Slack: "Campaign launched: Total Football TikTok"
```

### Flow 2: Automated Monitoring & Optimization

```mermaid
sequenceDiagram
    participant AF as AppsFlyer Pipeline
    participant N8N as n8n (CRON hourly)
    participant PG as PostgreSQL
    participant DIFY as Dify (Monitor Agent)
    participant LLM as Claude Sonnet 4
    participant TT as TikTok Ads API
    participant Slack as Slack

    AF->>N8N: Real-time event stream
    N8N->>PG: Store latest metrics
    N8N->>DIFY: Analyze campaign performance
    DIFY->>LLM: ReAct: Check thresholds, decide action
    
    alt Full Auto (CTR < 0.5%)
        LLM-->>DIFY: Action: PAUSE ad (tier: full_auto)
        DIFY-->>N8N: Decision JSON
        N8N->>TT: Pause ad via API
        N8N->>PG: Log decision (audit trail)
    else Semi-Auto (budget increase > 20%)
        LLM-->>DIFY: Action: SCALE +25% (tier: semi_auto)
        DIFY-->>N8N: Decision JSON
        N8N->>Slack: "🔔 Approval needed: Scale TF_001 budget +25%"
        Slack->>N8N: UA Manager approves ✅
        N8N->>TT: Update budget via API
        N8N->>PG: Log decision (approved)
    end
```

---

## Security Architecture

| Layer | Control | Implementation |
|---|---|---|
| **Credentials** | API keys encrypted at rest | n8n credentials vault (AES-256) |
| **LLM Access** | API key rotation | Quarterly rotation, Dify environment variables |
| **Database** | Network isolation | PostgreSQL on internal Docker network only |
| **n8n UI** | Authentication | n8n built-in auth (email + password) |
| **HITL** | Authorized users only | Slack workspace access controls |
| **Budget Guardrails** | Deterministic rules (not LLM) | Hard-coded limits in n8n workflows: daily cap, % change cap |
| **Audit Trail** | All AI decisions logged | PostgreSQL `agent_decisions` table — immutable append-only |
| **Token Management** | Auto-refresh workflows | n8n CRON job checks token expiry, triggers refresh |

### Anti-Hallucination Guardrails

| Guardrail | Type | Rule |
|---|---|---|
| Daily budget ceiling | Hard limit | AI cannot set daily budget > $X (configured per game) |
| Budget change cap | Percentage | Auto-actions limited to ≤20% change. >20% requires HITL |
| Minimum data threshold | Data quality | Agent must have ≥24hr of data before making optimization decisions |
| Confidence scoring | LLM output | Agent must include confidence score (0-1). Actions below 0.7 require HITL |
| Double-check pattern | Validation | n8n validates Dify's JSON output against schema before executing any API call |
| Rollback capability | Safety net | All campaign changes logged with pre-change state for instant rollback |

---

## Observability Strategy

| Signal | Source | Storage | Alert |
|---|---|---|---|
| **Workflow execution** | n8n execution history | n8n internal DB | Slack on failure |
| **Agent decisions** | Dify → PostgreSQL | `agent_decisions` table | Slack daily digest |
| **API errors** | n8n error handler | PostgreSQL `api_errors` table | Slack immediate |
| **Rate limit hits** | Redis counters | Redis → n8n dashboard | Slack if >80% capacity |
| **Budget anomalies** | n8n guardrail checks | PostgreSQL | Slack immediate |
| **Token expiry** | n8n CRON check | n8n credentials vault | Slack 48hr warning |
| **System health** | Docker health checks | — | Slack on container restart |

### SLOs (Service Level Objectives)

| SLO | Target | Measurement |
|---|---|---|
| Workflow success rate | >99% | n8n executions succeeded / total |
| Agent response time | <30 seconds | Dify API call latency |
| Campaign launch time | <15 minutes | Workflow start to ad platform confirmation |
| Anomaly detection | <1 hour | AppsFlyer event to agent decision |
| HITL approval SLA | <4 hours | Slack alert to human response |

---

## Deployment Topology

### Development (Local)

```yaml
# docker-compose.yml
services:
  n8n:        # Port 5678
  dify-api:   # Port 5001
  dify-web:   # Port 3000
  postgres:   # Port 5432 (internal)
  redis:      # Port 6379 (internal)
  openclaw:   # Port 8080
volumes:
  postgres_data:
  redis_data:
  n8n_data:
  creative_assets:
```

### Production (Company Server)

```
┌─────────────────────────────────────────────┐
│              Docker Swarm                    │
│                                             │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  │
│  │  n8n    │──│  Dify    │──│ OpenClaw │  │
│  │  :5678  │  │  :5001   │  │  :8080   │  │
│  └────┬────┘  └────┬─────┘  └────┬─────┘  │
│       │            │              │         │
│  ┌────┴────────────┴──┐   ┌──────┴─────┐  │
│  │   PostgreSQL       │   │   Redis    │  │
│  │   (internal only)  │   │ (internal) │  │
│  └────────────────────┘   └────────────┘  │
│                                             │
│  📁 /data/creatives (volume mount)          │
└─────────────────────────────────────────────┘
         │
    ───── HTTPS (n8n UI only) ────→ UA Manager
```

### Environment Matrix

| Setting | Dev | Production |
|---|---|---|
| n8n EXECUTIONS_MODE | regular | queue |
| DB backup | None | Daily pg_dump |
| Redis persistence | None | AOF enabled |
| Creative storage | Local disk | Mounted volume |
| LLM | Claude Sonnet 4 | Claude Sonnet 4 |
| HITL channel | Test Slack channel | Production Slack channel |
| Ad platform | Sandbox / test accounts | Production accounts |

---

## Architecture by Phase

| Phase | What Ships | Architecture Impact |
|---|---|---|
| **MVP (Sprint 1-2)** | Planning Agent + TikTok deploy | n8n + Dify + PostgreSQL + Redis (4 containers) |
| **Production (Sprint 3-4)** | Monitoring + HITL + Multi-platform | Add OpenClaw, expand workflows, add audit trail |
| **Scale (Sprint 5+)** | Analytics + LTV bidding + full loop | Add creative analytics, integrate Axon/AutoBid |

---

## Cost by Architecture Phase

| Phase | Containers | Estimated Infra | LLM API | Total |
|---|---|---|---|---|
| MVP | 4 | $0 (local) | ~$5/mo | ~$5/mo |
| Production | 6 | $50-100/mo | ~$50-150/mo | ~$100-250/mo |
| Scale (10 games) | 6 | $200-500/mo | ~$300-500/mo | ~$500-1000/mo |
