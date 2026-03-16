# Findings: AI Agentic UA Engine

> **Persistent memory file** — Updated after every planning phase.
>
> **Update Triggers**: After completing each phase and after running any domain skill via `/plan-explore`.

## Architecture Decision Records (ADRs)

| ADR# | Decision | Status | Date | Phase |
|---|---|---|---|---|
| ADR-001 | Hybrid orchestration: n8n + Dify + OpenClaw as 3-layer architecture | ✅ Accepted | 2026-03-13 | Phase 0 |
| ADR-002 | 3-tier HITL framework (Full Auto / Semi-Auto / Human Only) | ✅ Accepted | 2026-03-13 | Phase 1 |
| ADR-003 | LTV-based bidding via AppLovin Axon 2.0 + Appodeal AutoBid | ✅ Accepted | 2026-03-13 | Phase 1 |
| ADR-004 | Creative tagging via platform metrics + LLM, not GPU video analysis | ✅ Accepted | 2026-03-13 | Phase 2 |
| ADR-005 | OpenClaw with n8n-native fallback for API gateway | ✅ Accepted | 2026-03-13 | Phase 2 |
| ADR-006 | Pilot: TikTok-only, $500/wk, 1 game to validate core loop | ✅ Accepted | 2026-03-13 | Phase 2 |
| ADR-007 | Use existing AppsFlyer data pipeline as primary data source; ad platform APIs for writes + spend only | ✅ Accepted | 2026-03-13 | Phase 2 |
| ADR-008 | 10-week / 5-sprint phasing: Foundation → Creative+Deploy → Monitor+HITL → Multi-platform → Analytics | ✅ Accepted | 2026-03-13 | Phase 3 |
| ADR-009 | Internal cost-center model; ROI = headcount reduction + ROAS improvement + cost savings | ✅ Accepted | 2026-03-13 | Phase 3 |
| ADR-010 | n8n as workflow orchestrator (self-hosted, team experience, $0 cost) | ✅ Accepted | 2026-03-13 | Phase 4 |
| ADR-011 | Dify as AI reasoning platform (RAG + ReAct built-in, low-code) | ✅ Accepted | 2026-03-13 | Phase 4 |
| ADR-012 | Claude Sonnet 4 (primary) + GPT-4o (fallback) for LLM | ✅ Accepted | 2026-03-13 | Phase 4 |
| ADR-013 | PostgreSQL + Redis for state management and queuing | ✅ Accepted | 2026-03-13 | Phase 4 |
| ADR-014 | 6-tool GenAI pipeline; start with Creatify + OpenArt.ai (free tier) | ✅ Accepted | 2026-03-13 | Phase 4 |
| ADR-015 | Docker Compose (dev) → Docker Swarm (production) | ✅ Accepted | 2026-03-13 | Phase 4 |

### ADR-001: Hybrid Orchestration Architecture

- **Status**: ✅ Accepted
- **Date**: 2026-03-13
- **Context**: Need a system that can orchestrate complex workflows, run AI reasoning agents, and standardize external API connections. No single platform handles all three well.
- **Options Considered**:
  1. Monolithic custom build — full flexibility but high development cost
  2. Single platform (n8n or Dify alone) — simpler but limited in AI reasoning or workflow orchestration
  3. Hybrid 3-layer (n8n + Dify + OpenClaw) — decoupled, each layer does what it's best at
- **Decision**: Hybrid 3-layer architecture. n8n handles workflow orchestration/scheduling/API calls. Dify handles AI reasoning with ReAct agents and RAG. OpenClaw (MCP) standardizes external API connections.
- **Consequences**:
  - ✅ Each layer can be scaled, updated, or replaced independently
  - ✅ Team already has n8n and Dify experience
  - ⚠️ More operational complexity (3 systems to maintain)
  - ⚠️ Inter-layer communication needs careful design

### ADR-002: 3-Tier HITL Decision Framework

- **Status**: ✅ Accepted
- **Date**: 2026-03-13
- **Context**: AI making budget decisions is high-risk. Need clear boundaries for autonomous vs. supervised actions.
- **Decision**: Three tiers: Full Auto (pause bad ads, scale ≤20%), Semi-Auto (new campaigns, realloc >30% — requires Slack approval), Human Only (total budgets, brand direction).
- **Consequences**:
  - ✅ AI can optimize quickly within safe bounds
  - ✅ High-risk actions require human sign-off
  - ⚠️ Slack approval flow must have SLA to avoid blocking the system

### ADR-003: LTV-Based Bidding Strategy

- **Status**: ✅ Accepted
- **Date**: 2026-03-13
- **Context**: Standard CPI bidding doesn't account for player quality. Need predictive bidding that targets high-LTV players.
- **Decision**: Integrate AppLovin Axon 2.0 and Appodeal AutoBid for LTV prediction and automated bid optimization.
- **Consequences**:
  - ✅ Targets high-value players, improving ROAS
  - ⚠️ Vendor dependency on AppLovin and Appodeal
  - ⚠️ LTV prediction accuracy depends on historical data quality

---

## NFR Tracker

| NFR | Target | Current Status | Architecture Commitment | Notes |
|---|---|---|---|---|
| Performance | Campaign launch < 15 min | ⬜ Not validated | n8n parallel workflow execution | Need to benchmark |
| Availability | 99.5% uptime | ⬜ Not validated | Self-hosted with health monitoring | DR plan needed |
| Scalability | 50-100 variants/week | ⬜ Not validated | GenAI tool parallelization | API rate limits are bottleneck |
| Monitoring | Hourly pulls, 4h off-peak | ⬜ Not validated | n8n cron triggers | Rate limit handling critical |
| Anomaly Reaction | < 1 hour | ⬜ Not validated | Real-time alerting + auto-actions | Depends on monitoring frequency |
| Security | Credentials secured | ⬜ Not validated | Secret management TBD | Token refresh automation needed |
| Data Integrity | Full audit trail | ⬜ Not validated | Google Sheets logging | May need dedicated DB at scale |
| Recovery | RPO 1hr, RTO 4hr | ⬜ Not validated | TBD | DR plan needed |
| Budget Safety | Guardrails enforced | ⬜ Not validated | Deterministic rules, not LLM-based | Anti-hallucination critical |

---

## Discovered Constraints

### Technical
- Facebook Long-lived Access Tokens expire (need auto-refresh workflow)
- Each ad platform has different API rate limits (need per-platform queue management)
- GenAI tools have varying output quality and API stability
- LLM hallucination risk on budget decisions requires deterministic guardrails (not prompt-based)
- Video analysis for creative tagging requires GPU-capable infrastructure or vision API

### Business
- Southeast Asia market (latency to API endpoints may vary)
- Team size: 1-2 UA personnel per game (down from 3-5) — system must be self-managing
- Pilot scope: 1 game on TikTok Ads first, then expand

### Regulatory / Legal
- iOS ATT/IDFA compliance — limited user-level tracking
- SKAdNetwork constraints on attribution
- Ad platform Terms of Service compliance for automation
- No PII storage beyond platform requirements

## Market Insights

- Mobile gaming UA costs rising 20-30% YoY across major platforms
- Creative fatigue cycles shortening — need to refresh ads more frequently
- AI-generated creatives are becoming standard practice (30-40% of top publishers using GenAI)
- LTV-based bidding delivers 15-30% better ROAS vs. CPI-based bidding

## Risk Log

| Risk | Category | Likelihood | Impact | Status | Mitigation |
|---|---|---|---|---|---|
| API rate limits block pipeline | Tech | High | High | 🟡 Open | Queue systems, wait nodes, retry logic in n8n |
| Token expiry causes disconnection | Tech | High | Medium | 🟡 Open | Auto-refresh workflows, expiration alerts |
| LLM hallucination on budget allocation | Tech | Medium | High | 🟡 Open | Deterministic guardrails + constraint validation |
| GenAI creative quality inconsistency | Tech | Medium | Medium | 🟡 Open | Quality scoring before deployment, A/B testing |
| Ad platform policy changes | Business | Medium | High | 🟡 Open | Abstraction layer (OpenClaw), policy monitoring |
| Single point of failure (n8n) | Ops | Low | High | 🟡 Open | Health monitoring, failover strategy |
| LTV prediction accuracy insufficient | Tech | Medium | Medium | 🟡 Open | Requires sufficient historical data, calibration |
| Creative tagging pipeline latency | Tech | Medium | Low | 🟡 Open | Async processing, priority queuing |

## Key Decisions

| Decision | Date | Rationale | ADR# |
|---|---|---|---|
| Hybrid 3-layer architecture (n8n + Dify + OpenClaw) | 2026-03-13 | Decoupled, team has experience, each layer specialized | ADR-001 |
| 3-tier HITL framework | 2026-03-13 | Balance speed with safety for budget decisions | ADR-002 |
| LTV-based bidding via Axon + AutoBid | 2026-03-13 | Target player quality not just quantity | ADR-003 |
| AppsFlyer pipeline as primary data source | 2026-03-13 | Existing infra, real-time LTV signals, 75-85% fewer API calls | ADR-007 |
| 10-week / 5-sprint phasing with Week 6 Review Gate | 2026-03-13 | Validate core loop on TikTok before expanding to all platforms | ADR-008 |
| Internal tool (cost center), not SaaS product | 2026-03-13 | Value = operational efficiency, not direct revenue | ADR-009 |

## Skills Used

| Date | Skill | Category | Key Output |
|---|---|---|---|
| 2026-03-13 | `data-collection-plan` | Analytics & Data | Data inventory framework: UA metrics (CPI, ROAS, CTR, LTV), creative performance data, audience segments. Privacy compliance checklist for SEA market. Data quality rules for campaign naming conventions |
| 2026-03-13 | `customer-lifetime-value` | Analytics & Data | LTV modeling approach: Subscription CLV formula for gaming (daily revenue / churn rate × margin). CAC:LTV ratio target ≥3:1. Sensitivity analysis: reducing churn by 10% can increase CLV by 25%. LTV segmentation by acquisition channel critical for budget allocation |
| 2026-03-13 | `automation-workflow` | AI & Technology | Workflow design patterns: trigger-action sequences for n8n. Error handling matrix for each API step (detection + recovery). Monitoring: failure alerts via Slack, daily digest, monthly review. ROI calculation framework for automation. Anti-pattern: keep workflows under 7 steps to maintain reliability |
| 2026-03-13 | `benchmarking-report` | Analytics & Data | UA benchmarking methodology — compare CPI, ROAS, LTV against industry medians. Gap analysis for underperforming channels |
| 2026-03-13 | `unit-economics` | Finance & Pricing | Per-game UA unit economics: LTV:CAC ratio tracking, payback period calculation, sensitivity analysis. Key: if unit economics don't work for 1 game, they won't work for 10 |
| 2026-03-13 | `scope-of-work` | Operations & Systems | Sprint deliverables structure: specific outputs per sprint, acceptance criteria, exclusions. Review Gate at Week 6 with measurable lift criteria |

## Ideas & Future Scope

- Multi-game portfolio optimization (cross-game budget allocation)
- Playable ad generation (interactive HTML5 ads)
- Influencer campaign automation
- Organic/paid attribution blending
- Custom MMP integration for deeper funnel analysis
- Self-learning creative templates based on genre-specific winning patterns

## Open Questions

- [x] ~~API rate limits~~ → Meta: 9,000 pts/hr (1hr window, HTTP 429). Google: 15K ops/day (Basic), unlimited (Standard), RESOURCE_TEMPORARILY_EXHAUSTED. TikTok: 600 req/min (video endpoint, 1-min sliding window, HTTP 429)
- [x] ~~Cloud hosting~~ → Local for testing/dev, company server for production deployment
- [x] ~~Historical data~~ → Years of campaign data available (strong foundation for LTV modeling)
- [x] ~~MMP~~ → AppsFlyer (integrate for attribution + funnel data)
- [x] ~~Video analysis GPU~~ → User prefers ad/creative-level metrics from platforms (CPI, ROAS, CTR per creative) over GPU-intensive video analysis. Creative tagging can use platform-reported metrics + LLM analysis of creative metadata instead of computer vision
- [x] ~~Pilot budget~~ → $500/week for 1 game on TikTok Ads

## Session Notes

### 2026-03-13 — Phase 0: Intake & Sizing
- Project classified as 🔴 Large
- Key complexity: 3 ad platforms, 6 GenAI tools, AI reasoning agents, HITL framework, real-time monitoring
- NotebookLM notebook connected and activated

### 2026-03-13 — Phase 1: Requirements & Discovery
- Completed full overview.md with 9 NFRs, 16 integrations, HITL framework
- Queried NotebookLM 3 times for domain-specific details
- Auto-ran 3 domain skills: data-collection-plan, customer-lifetime-value, automation-workflow
- Key insight from LTV skill: CAC:LTV ratio ≥3:1 should be a system-level guardrail
- Key insight from automation-workflow: Keep n8n workflows under 7 steps for reliability
- User provided API rate limit data, hosting strategy, MMP (AppsFlyer), pilot budget ($500/wk)
- **Architecture pivot**: Creative tagging will use ad-level metrics + LLM metadata analysis instead of GPU-based video analysis. This significantly reduces infrastructure complexity

### 2026-03-13 — Phase 2: Feasibility & Research
- Build-vs-buy analysis: n8n (build), Dify (build), GenAI tools (buy), bidding (buy), analytics (hybrid)
- API feasibility confirmed: Meta 9K pts/hr safe, Google needs Standard access, TikTok 600 req/min safe
- Creative tagging pivot logged as ADR-004 — no GPU needed
- OpenClaw fallback strategy logged as ADR-005
- Pilot strategy: TikTok-only, $500/wk, validate core loop (ADR-006)
- 4 PoC recommendations defined
- Auto-ran benchmarking-report skill
- NotebookLM queried for rate limit mitigation strategies

### 2026-03-13 — Phase 2 Revision: AppsFlyer Pipeline
- **Critical discovery**: Company already has real-time AppsFlyer data pipeline (attribution, user activities, transactions)
- This provides early LTV signals without hitting ad platform APIs
- **API call reduction**: 75-85% fewer ad platform API calls needed
- Ad platform APIs now used primarily for: (1) campaign CRUD operations, (2) spend data pulls 2-4x/day
- Monitoring agent reads from AppsFlyer pipeline (real-time) instead of polling ad APIs hourly
- Logged as ADR-007. This is a significant simplification of the monitoring architecture

### 2026-03-13 — Phase 3: Solution Strategy
- Phasing strategy: 10-week / 5-sprint roadmap (from NotebookLM's blueprint)
- Sprint 1: Infrastructure + Planning Agent (Weeks 1-2)
- Sprint 2: Creative Pipeline + TikTok Deploy (Weeks 3-4)
- Sprint 3: Monitoring + HITL + **Review Gate at Week 6** (Weeks 5-6)
- Sprint 4: Multi-Platform Expansion (Weeks 7-8)
- Sprint 5: Analytics + Optimization + Closed Loop (Weeks 9-10)
- Business model: Internal cost center (not SaaS product)
- 6 KPIs defined: ROAS +15-25%, launch <2 days, 50-100 variants/wk, costs -50-70%, anomaly <1hr, headcount -60%
- Auto-ran unit-economics and scope-of-work skills
- Logged ADR-008 (phasing) and ADR-009 (business model)

### 2026-03-13 — Phase 4: Technology Evaluation
- Evaluated 6 technology layers with ADR-format decisions
- n8n over Make/Temporal (team experience, self-hosted, $0) — ADR-010
- Dify over LangChain/AutoGen (low-code, built-in RAG) — ADR-011
- Claude Sonnet 4 primary + GPT-4o fallback — ADR-012
- PostgreSQL + Redis for state and queuing — ADR-013
- 6 GenAI tools, start with Creatify + OpenArt.ai free tiers — ADR-014
- Docker Compose → Docker Swarm deployment — ADR-015
- Cost estimate: $5/mo (dev) to $700-1200/mo (10-game scale), excluding ad spend
- Auto-ran ai-use-case-finder skill

### 2026-03-13 — Phase 5: Solution Architecture
- Wrote `architecture.md` with C4 Level 1 (System Context), Level 2 (Container), Level 3 (Components)
- 3 data contracts defined: CampaignPlan, AgentDecision, CreativeAsset (JSON schemas)
- 2 sequence diagrams: Campaign Launch flow, Automated Monitoring flow
- 7 containers: n8n, Dify, PostgreSQL, Redis, OpenClaw, File Storage, Monitoring
- 8 n8n workflows defined with triggers, inputs, outputs, and frequencies
- 3 Dify agents: Campaign Strategist, Campaign Monitor, Creative Analyst
- Security: 8 controls including anti-hallucination guardrails (6 specific rules)
- Observability: 7 signal types, 5 SLOs (99% workflow success, <30s agent response, <15min launch)
- Deployment: Docker Compose (dev) → Docker Swarm (prod)

### 2026-03-13 — Phase 6: Implementation Planning
- Wrote `implementation-plan.md` with 41 tasks across 5 sprints
- 5 milestones: Foundation Live (Wk2), First AI Campaign (Wk4), Closed Loop (Wk6), Multi-Platform (Wk8), Full Engine (Wk10)
- Review Gate at Week 6 is critical go/no-go for multi-platform expansion
- Each sprint is ~10 person-days (1-2 engineers)
- Mermaid Gantt chart with task dependencies
- 7 assumptions, 5 constraints, 7 risk mitigations, Definition of Done checklist

### 2026-03-13 — Phase 7: Technical Handoff
- Wrote `handoff.md` — developer-ready document
- Repo structure, Docker Compose setup, PostgreSQL schema (4 tables with SQL)
- Environment variable template (15+ config vars)
- Anti-hallucination guardrails section (critical for implementers)
- Sprint 1 checklist as starting point

### 2026-03-13 — Phase 8: Business Proposal
- Wrote `business-proposal.md` — stakeholder-facing
- Executive summary, problem/solution framing, before/after comparison
- ROI projections: conservative 2.3x → optimistic 35x
- Pilot cost: ~$50 total (excl. $5K ad spend), production ~$310/mo
- Clear decision request: engineering allocation + pilot budget approval
- Review Gate at Week 6 as risk control mechanism

### Planning Complete ✅
- 8 phases completed in single session
- 15 ADRs logged
- 8 project documents produced
- 6 domain skills auto-ran
- 5 NotebookLM queries
- Total: `overview.md`, `research.md`, `tech-stack.md`, `architecture.md`, `implementation-plan.md`, `handoff.md`, `business-proposal.md`, `findings.md`

### 2026-03-13 — Domain Exploration: Alternative Architecture (No Dify/OpenClaw)
**Skills Used**: Sequential Thinking (8 thoughts), `automation-workflow`, `process-automation-audit`
- Evaluated 4 alternative approaches: (A) No-Code SaaS, (B) Lean n8n, (C) Spreadsheet + AI Copilot, (D) Spreadsheet Command Center
- **Recommended**: "Spreadsheet Command Center" — n8n Cloud + Google Sheets + direct Claude/GPT API calls
- Eliminates: Dify, OpenClaw, PostgreSQL, Redis, Docker, company server
- UA team interface: Google Sheets (daily tool) + Slack (alerts/approvals)
- Cost reduction: ~$70-180/mo vs ~$160-310/mo (current plan)
- Timeline: 6-8 weeks vs 10 weeks (current plan)
- Tradeoff: Lower scalability ceiling (3-5 games vs 10+), acceptable for pilot
- All safety guardrails preserved (HITL tiers, budget caps, anti-hallucination are in n8n workflows, not in Dify)
- **Key insight**: The current architecture is engineer-facing. The UA team needs a business-facing system built on tools they already use daily
- **Decision needed**: Does the team want to pivot to this simpler approach?
