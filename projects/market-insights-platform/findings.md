# Findings: Market Insights Platform

> **Persistent memory file** — Updated throughout all planning phases. Prevents context loss between sessions.

## Discovered Constraints

### Technical
- Sensor Tower data already resides in Postgres DB (monthly batch, 10K apps, multi-market) — architecture must integrate with this existing store
- Minimax API is OpenAI-compatible (Chat Completions format) — simplifies routing layer
- Claude API uses Anthropic format — need adapter or unified gateway
- Apify actors exist for Reddit, Discord, Facebook Pages, App Store reviews — minimal custom scraper development
- abei.gov.vn game license data is published as HTML pages (not API) — requires web scraping
- Community data quality varies significantly across platforms; Reddit/Discord often unstructured

### Business
- Team of 5 internal users — no need for multi-tenancy, complex RBAC, or billing
- MVP scope — prioritize report generation workflow over comprehensive dashboards
- Cost optimization is important — hence the Minimax + Claude routing strategy
- Reports must be editable before finalization — LLM outputs are drafts, not final products

### Regulatory / Legal
- **Vietnamese gaming regulations**: abei.gov.vn (Cục Phát thanh, Truyền hình và Thông tin điện tử) publishes game license categories
  - "Danh sách các trò chơi được cấp quyết định phê duyệt nội dung, kịch bản (2015 – 2025)" — game approval decisions
  - G1/G2/G3 license categories determine what types of games can operate in Vietnam
  - Data is updated periodically (approximately monthly snapshots as of Aug 2025)
- LLM API usage: no personal data processing through LLMs — only market/public data

## Market Insights

- Multi-LLM routing is an established pattern: route simple tasks (summarization, formatting) to cheaper models (Minimax M2 at $0.26/M input), reserve expensive models (Claude Sonnet/Opus) for complex analysis
- Minimax M2.5 achieves ~100 TPS with strong coding/agent/tool-calling capabilities at ~$1/hr continuous operation
- Claude excels at long-context synthesis (200K context window), strategic recommendations, and nuanced analysis
- Apify has 15,000+ ready-made actors and provides MCP integration for AI agent workflows
- Sensor Tower's Game Intelligence includes proprietary game taxonomy beyond standard app store categories

## AI Use Case Analysis (from ai-use-case-finder skill)

| Task | Current State | AI Solution | Model | Time Saved | Priority Score |
|---|---|---|---|---|---|
| **Data summarization** | Manually reading through raw data exports | LLM summarization of key metrics and trends | Minimax M2 | 3-4 hrs/report | ⭐⭐⭐⭐⭐ (High V, High F, High I) |
| **Community sentiment analysis** | Manually reading Reddit/Discord posts | LLM-powered sentiment classification and theme extraction | Claude Sonnet | 4-6 hrs/report | ⭐⭐⭐⭐⭐ (High V, High F, High I) |
| **Cross-source synthesis** | Manually comparing data across sources to find patterns | LLM synthesis of market data + community + regulations | Claude Sonnet | 5-8 hrs/report | ⭐⭐⭐⭐⭐ (High V, Med F, High I) |
| **Report drafting** | Writing reports from scratch based on analysis | LLM generates structured report draft from analyzed data | Minimax/Claude (routed) | 3-5 hrs/report | ⭐⭐⭐⭐ (High V, High F, Med I) |
| **Regulation change tracking** | Manual checking of abei.gov.vn updates | Automated scraping + LLM diff summarization | Minimax M2 | 1-2 hrs/week | ⭐⭐⭐ (Med V, Med F, Med I) |
| **Translation** | Manual Vietnamese ↔ English for regulation data | LLM translation of regulation content | Minimax M2 | 1 hr/report | ⭐⭐⭐ (Med V, High F, Low I) |

## Data Collection Plan (from data-collection-plan skill)

| Data Point | Source | Collection Method | Storage | Frequency | Privacy Level |
|---|---|---|---|---|---|
| App rankings (top 10K) | Sensor Tower API | API → Postgres (existing) | Postgres | Monthly | Low |
| App downloads estimates | Sensor Tower API | API → Postgres (existing) | Postgres | Monthly | Low |
| App revenue estimates | Sensor Tower API | API → Postgres (existing) | Postgres | Monthly | Low |
| Reddit posts (gaming subs) | Apify Reddit Scraper | Apify Actor → API → DB | Postgres | Weekly/on-demand | Low |
| Discord messages | Apify Discord Scraper | Apify Actor → API → DB | Postgres | Weekly/on-demand | Low |
| Facebook gaming groups | Apify Facebook Scraper | Apify Actor → API → DB | Postgres | Weekly/on-demand | Low |
| App Store reviews | Apify App Store Scraper | Apify Actor → API → DB | Postgres | Weekly/on-demand | Low |
| Game license decisions | abei.gov.vn | Web scraping (Apify custom) | Postgres | Monthly | Low |
| LLM-generated insights | Minimax/Claude APIs | API response → DB | Postgres | Per-report | Low |
| Team report edits | Web app | Direct save | Postgres | Real-time | Low |

## Dashboard Design Brief (from data-dashboard-design skill)

**Audience**: 5 internal market analysts + 1 team lead  
**Purpose**: Monitor mobile gaming market trends, track regulatory changes, generate insight reports  
**Key Questions**:
1. "Which games/genres are growing fastest in which markets?"
2. "What is community sentiment around key games/events?"
3. "What new games have been licensed and what opportunities/threats does that create?"
4. "How do market trends align with or diverge from community sentiment?"

**Top-line KPIs**: Top movers (downloads growth), Revenue leaders, Community sentiment score, New licenses issued  
**Trends**: Monthly download/revenue trends, Sentiment over time  
**Breakdowns**: By market, By genre/category, By platform (iOS/Android)

## Competitor Intelligence

| Competitor/Alternative | Key Takeaway | Differentiation Opportunity |
|---|---|---|
| **Sensor Tower Platform** (direct) | Comprehensive data but no LLM analysis or community data integration | Our platform adds community sentiment + regulation context + LLM-driven synthesis |
| **data.ai (App Annie)** | Similar market data, no regulation tracking | We combine market data with Vietnamese regulation awareness |
| **Newzoo** | Strong gaming market reports but expensive, no custom generation | Self-generated reports tailored to specific games/markets at lower cost |
| **Manual process** (current) | Slow, error-prone, no standardization | Automation reduces report generation from days to hours |

## Risk Log

| Risk | Status | Mitigation |
|---|---|---|
| LLM hallucination in market analysis | 🟡 Open | All LLM outputs are drafts requiring human review/edit before finalization |
| Apify scraping blocked by platform changes | 🟡 Open | Use official actors with auto-update; have fallback manual import |
| abei.gov.vn site structure changes break scraping | 🟡 Open | Design scraping with loose selectors; add monitoring for failures |
| Minimax API availability/reliability | 🟡 Open | Implement fallback to Claude for all tasks if Minimax is down |
| Cost overrun on Claude API for deep analysis | 🟡 Open | Task routing ensures Claude is only used for complex tasks; monitor usage |
| Sensor Tower API rate limits | 🟢 Mitigated | Data already batch-ingested to Postgres — no real-time API dependency |

## Solution Strategy (Phase 3)

### Build-vs-Buy Analysis

| Component | Decision | Cost/Risk/Timeline Tradeoffs |
|---|---|---|
| **Market data ingestion** | ✅ Already built | Sensor Tower data already in Postgres — zero build cost |
| **Community data scraping** | 🛒 Buy (Apify) | Ready-made actors save 2-3 weeks vs. custom scrapers; $49-149/mo |
| **Regulation data scraping** | 🔨 Build (custom) | Simple HTML scraping, no existing actor for abei.gov.vn; <1 day effort |
| **LLM routing gateway** | 🛒 Buy (LiteLLM OSS) | Open-source, self-hostable; saves 1-2 weeks vs. custom router; zero cost |
| **LLM APIs** | 🛒 Buy (Minimax + Claude) | No feasible alternative to API-based LLMs for this use case |
| **Collaborative report editor** | 🧩 Extend (Tiptap/Plate) | Open-source editors with collaboration plugins; extend for LLM integration |
| **Web application** | 🔨 Build (Next.js) | Custom build for integrated experience; no off-the-shelf tool combines all features |

### Phasing Strategy

#### 🟢 Phase 1: MVP (Weeks 1-6)
**Goal**: Working report generation pipeline with basic collaboration

| Capability | Scope |
|---|---|
| Data sources | Sensor Tower (existing Postgres) + 1 Apify source (Reddit or App Store) |
| LLM pipeline | Single-model (Claude Sonnet only, add Minimax routing in Phase 2) |
| Reports | 2 template types: Market Overview, Competitive Brief |
| Editor | Basic rich text editing with LLM-generated content insertion |
| Users | Simple email auth for 5 users |
| Deployment | Vercel (free/hobby tier) |

#### 🟡 Phase 2: Production (Weeks 7-12)
**Goal**: Full data pipeline + multi-LLM routing + all report types

| Capability | Scope |
|---|---|
| Data sources | All 5 (Sensor Tower + Reddit + Discord + Facebook + App Store + abei.gov.vn) |
| LLM pipeline | Multi-LLM routing via LiteLLM (Minimax for simple, Claude for complex) |
| Reports | 4+ template types: Market Overview, Competitive Brief, Regulatory Update, Sentiment Analysis |
| Editor | Full collaborative editing with comments, version history |
| Dashboard | Data overview dashboard with key metrics and trends |
| Scheduling | Automated weekly data ingestion via background jobs |

#### 🔴 Phase 3: Scale (Months 4-6+)
**Goal**: Optimization, insights quality improvement, extended coverage

| Capability | Scope |
|---|---|
| Data sources | Additional markets, new community sources |
| LLM pipeline | Prompt optimization, RAG for historical context, report quality scoring |
| Reports | Custom template builder, report scheduling, auto-distribution |
| Analytics | Usage analytics, report quality metrics, cost optimization dashboard |
| Scale | Support 20+ users, API for external integrations |

### Success Metrics & KPIs

| KPI | Target (MVP) | Target (Production) | Measurement |
|---|---|---|---|
| Report generation time | < 30 min (vs. days currently) | < 15 min | Timer from request to draft |
| Reports generated per week | 2-3 | 5-10 | Count in system |
| Data source coverage per report | 2 sources | 4-5 sources | Sources cited in report |
| Team adoption | 3/5 users active weekly | 5/5 users active weekly | Login + report interaction |
| LLM cost per report | < $5 | < $2 (with Minimax routing) | API cost tracking |
| Report edit rate (before publish) | < 40% content changed | < 25% content changed | Diff analysis |
| User satisfaction | 7/10 NPS | 8/10 NPS | Monthly survey |

### Business Model

This is an **internal productivity tool** — no direct revenue generation. Value is measured in:
- **Time savings**: 15-20 hrs/week across team × analyst hourly cost = significant labor savings
- **Decision quality**: Better data coverage → better market decisions → competitive advantage
- **Regulatory compliance**: Proactive tracking of regulation changes reduces compliance risk

**Future monetization potential** (out of scope for MVP):
- License the platform to other gaming companies as a SaaS product
- Offer market intelligence reports as a paid subscription service
- White-label the LLM analysis pipeline for consulting engagements

## Key Decisions

| Decision | Date | Rationale |
|---|---|---|
| Classified as 🟡 Medium project | 2026-03-14 | Multi-component app with defined scope, not multi-tenant SaaS |
| Phase sequence: 0→1→2→3→4→5→6→7→8 | 2026-03-14 | User requested full phase coverage |
| Dual-LLM strategy (Minimax + Claude) | 2026-03-14 | Cost optimization: Minimax for simple tasks (~12x cheaper), Claude for deep analysis |
| Use Apify for community data collection | 2026-03-14 | Ready-made actors for Reddit, Discord, Facebook, App Store — minimal development |
| Internal-only MVP for 5 users | 2026-03-14 | No need for multi-tenancy, complex auth, or billing in prototype |
| MVP starts with Claude-only, add Minimax in Phase 2 | 2026-03-14 | Reduces integration complexity in MVP; validate quality first, optimize cost second |
| Build custom regulation scraper | 2026-03-14 | abei.gov.vn has no existing Apify actor; simple HTML scraping is <1 day effort |
| 3-phase rollout: MVP (6w) → Production (6w) → Scale (3mo+) | 2026-03-14 | MVP validates core workflow, Production adds full data coverage, Scale optimizes |

## Open Questions

- [ ] What specific Reddit subreddits and Discord servers to target?
- [ ] What report templates are most commonly needed? (e.g., weekly market brief, competitive deep-dive, regulatory update)
- [ ] Should the regulation scraper also track G4/other newer license categories?
- [ ] Is there a preference for deployment platform? (e.g., Vercel, self-hosted, internal server)

## Skills Used

| Skill | Key Findings |
|---|---|
| `data-collection-plan` | Mapped all data sources with collection methods, frequency, and storage. Privacy risk is low (all public/market data). |
| `ai-use-case-finder` | 6 AI use cases identified with priority scoring. Highest priority: data summarization, community sentiment analysis, cross-source synthesis. |
| `data-dashboard-design` | Dashboard architecture defined: top-line KPIs (movers, revenue, sentiment, licenses), trend charts, breakdowns by market/genre/platform. |

## Session Notes

### 2026-03-14 — Phase 0: Intake & Sizing
- Project idea: Combine Sensor Tower + community + regulation data → LLM-generated market insights
- Classified as 🟡 Medium (multi-component app, 5+ integrations, greenfield MVP)
- User confirmed: Sensor Tower data already in Postgres, targets Reddit/Discord/Facebook/App Store, regulation from abei.gov.vn

### 2026-03-14 — Phase 1: Requirements & Discovery
- Completed discovery questions, NFRs, integration inventory
- Researched abei.gov.vn — Cục PTTH publishes game license approval lists (2015–2025)
- Researched Minimax API — M2/M2.5 best for simple tasks ($0.26/M input), OpenAI-compatible
- Researched multi-LLM routing patterns — route by task complexity, use AI gateway
- Ran 3 domain skills: data-collection-plan, ai-use-case-finder, data-dashboard-design

### 2026-03-14 — Phase 2: Feasibility & Research
- Deep-dived into multi-LLM routing frameworks: LiteLLM (recommended for MVP), Portkey (enterprise alternative)
- Researched mobile game market intelligence landscape: Sensor Tower dominates (acquired data.ai), but no tool combines market data + community + regulation
- Detailed Vietnamese gaming regulation: G1–G4 categories under Decree 147/2024 (effective Dec 2024), supersedes Decree 72/2013
- Identified 3 PoC experiments to validate before full build
- Estimated infrastructure costs: ~$100-200/mo for MVP, ~$500-1,200/mo at scale
- ADR: LiteLLM selected as LLM gateway (open-source, routing, cost tracking)
- ADR: Tiptap or Plate recommended for collaborative rich text editing
- Phase sequence updated to include Phase 2: 0→1→2→4→5→6→7→8

### 2026-03-14 — Phase 3: Solution Strategy
- Build-vs-buy analysis: 2 build, 3 buy, 1 extend, 1 already built
- 3-phase rollout designed: MVP (6w, Claude-only, 2 data sources) → Production (6w, multi-LLM, all sources) → Scale (3mo+)
- Key strategic decision: MVP starts with Claude-only to validate quality, add Minimax cost optimization in Phase 2
- KPIs defined: report generation time (<30 min), team adoption (3/5 weekly users), LLM cost per report (<$5)
- Business model: internal productivity tool, future SaaS potential
- ADR: 3-phase rollout strategy with incremental data source integration
- ADR: MVP starts Claude-only, add Minimax routing in Production phase

### 2026-03-14 — Phase 4: Technology Evaluation
- User clarified: project involves real code (Apify setup, regulation scraper, web app build) — planning produces actionable handoff
- Evaluated 4 critical tech layers with scored ADR matrices:
  - Rich Text Editor: Tiptap (4.9) beats Plate (4.2) — AI Agent toolkit + Yjs collaboration
  - LLM SDK: Vercel AI SDK v6 (4.7) beats LangChain.js (4.0) — streaming-first, `useChat` hooks
  - ORM: Drizzle (4.9) beats Prisma (4.0) — lighter serverless cold starts, SQL-native
  - Background Jobs: Inngest (4.8) beats BullMQ (3.8) — serverless-native, no Redis
- 16-technology stack finalized across all layers
- Cost estimate: ~$30-50/mo MVP, ~$155-235/mo Production, ~$475-825/mo Scale

### 2026-03-14 — Phase 5: Solution Architecture
- Designed 3-phase architecture diagrams (Mermaid): MVP (Next.js + Claude + Apify), Production (+ LiteLLM + all scrapers + Yjs), Scale (+ RAG + Redis + Vector DB)
- Defined 7-table ERD: users, sensor_tower_apps, sensor_tower_metrics, community_posts, regulation_entries, reports, report_sections, scrape_jobs
- 3 data flow diagrams: Ingestion Pipeline, Report Generation Pipeline, Community Sentiment Pipeline
- 13 API endpoints with TypeScript request/response contracts
- Full Next.js folder structure: app router, components, lib (db, ai, scraping, inngest), types
- Key data-first decisions: Tiptap JSON for report storage, frozen data snapshots for reproducibility, section-level LLM cost tracking

### 2026-03-14 — Phase 6: Implementation Planning
- 6 milestones across 6-week MVP timeline: Foundation (20h), Data Layer (23h), Scraping (30h), LLM Integration (26h), Report Editor (27h), Polish (26h)
- 56 individual tasks with effort estimates, dependencies, and status tracking
- Mermaid Gantt chart for visual timeline
- Definition of Done criteria for each milestone
- 5 implementation risks identified with mitigations
- Post-MVP roadmap: 6 Production-phase features (LiteLLM, more scrapers, collaboration, templates, scheduling)

### 2026-03-14 — Phase 7: Technical Handoff
- Consolidated all 6 prior phases into self-contained `handoff.md`
- Includes: complete Drizzle schema (TypeScript), API contracts with TS interfaces, exact npm setup commands
- 7 implementation notes for the AI coding agent (existing Postgres handling, abei.gov.vn URL, Apify actor, prompt templates, streaming approach, Claude-only MVP, design premium)
- Acceptance criteria: 9 must-pass items covering full user workflow

### 2026-03-14 — Phase 8: Business Proposal
- Translated technical planning into stakeholder-friendly `business-proposal.md`
- Quantified ROI: ~$24,000/year labor savings vs ~$2,400/year platform cost = 10x ROI
- Monthly cost trajectory: $50 (MVP) → $200 (Production) → $650 (Scale)
- Responsible AI usage framing: all outputs are editable drafts, human review mandatory
- 4-item approval checklist: timeline, Claude budget, Apify budget, Vercel hosting
- **All 8 phases complete** — project planning is done




