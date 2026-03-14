# Research: Market Insights Platform

## Executive Summary

The Market Insights Platform sits at the intersection of three rapidly maturing technology domains: **mobile game market intelligence** (a $4.67B BI market in 2024, doubling by 2029), **multi-LLM orchestration** (with proven open-source gateways like LiteLLM and Portkey), and **web scraping infrastructure** (Apify's 15K+ actors covering all target platforms). 

The technical feasibility is strong. Sensor Tower data is already ingested into Postgres. Apify provides ready-made scrapers for Reddit, Discord, Facebook, and App Store reviews. Vietnamese gaming regulation data (G1–G4 license categories under Decree 147/2024) is publicly accessible on abei.gov.vn. The multi-LLM routing pattern (cheap Minimax for simple tasks, Claude for deep analysis) is a well-established industry practice with multiple open-source implementations.

The primary risks are operational (scraping stability, LLM output quality) rather than technical (all APIs exist and are proven). An MVP is highly feasible with current tooling within a 6–8 week timeline for a team of 5 users.

## Market Analysis

### Market Size & Trends

| Metric | Value | Source |
|---|---|---|
| Mobile games market size (2025) | $188.2B | Straits Research |
| Projected growth (2033) | $476B (12.3% CAGR) | Straits Research |
| BI market in mobile games (2024) | $4.67B, doubling by 2029 | Keewano |
| Sensor Tower acquired data.ai | Consolidated market leader | PocketGamer |
| Hybrid-casual games | Dominant genre category in 2025 | Mistplay |

**Key trends affecting this project:**
1. **AI-powered analytics** — GenAI is used extensively for user acquisition creatives and early build testing
2. **Alternative app stores & DTC webshops** — Regulatory changes creating new distribution channels, impacting revenue tracking visibility
3. **Retention & LiveOps focus** — Downloads slowing; retention is the critical KPI
4. **IP-based games growth** — Games based on existing IP gaining market share
5. **Vietnam market** — Decree 147/2024 tightens regulation, creating need for compliance monitoring

### Target Demographics

- **Primary**: Internal market analysts at the company (5 people)
- **Market focus**: Vietnam and Southeast Asian mobile gaming markets
- **Data coverage**: Top 10,000 apps across multiple markets (already in Postgres)

### Demand Signals

- Manual report generation currently takes days per report
- Fragmented data across 5+ sources with no unified view
- Growing regulatory complexity in Vietnam (Decree 147/2024 effective Dec 2024)
- Community sentiment increasingly influences game publishing decisions

## Competitor Analysis

### Direct Competitors (Market Intelligence Platforms)

| Name | Pricing | Strengths | Weaknesses |
|---|---|---|---|
| **Sensor Tower** | Enterprise (quote-based annual) | Comprehensive data, Game Intelligence, proprietary taxonomy, acquired data.ai | No LLM analysis, no community sentiment, no regulation tracking |
| **GameRefinery** | Enterprise | Deep feature-level analysis, player motivations, LiveOps tracking | No Vietnamese regulation data, no community scraping |
| **AppMagic** | ~$200-500/mo | Good download/revenue estimates, competitive pricing | Smaller dataset than Sensor Tower, no AI analysis |
| **FoxData** | Freemium | GameIQ tags, ad creative insights, player behavior | Limited market coverage, newer platform |
| **Newzoo** | Enterprise (~$5K+/yr) | Strong market reports, global coverage | Expensive, no custom report generation, no regulation data |

### Indirect Competitors (Alternative Approaches)

| Approach | Description | Limitation |
|---|---|---|
| **In-house manual process** (current) | Manually pulling data from Sensor Tower + monitoring communities + checking regulations | Slow (days per report), error-prone, no standardization |
| **General BI tools** (Tableau, Looker, Metabase) | Dashboard visualization of structured data | No LLM analysis, no scraping integration, no report generation |
| **ChatGPT/Claude manual use** | Copy-paste data into LLM chat for ad-hoc analysis | No automation, no data integration, not reproducible |
| **Custom consulting** | Hire analysts to produce reports | Expensive, slow, doesn't scale |

### Open Source Alternatives

| Project | Description | Relevance |
|---|---|---|
| **LiteLLM** | Open-source LLM proxy — unified API for 100+ providers, routing, fallbacks, cost tracking | ⭐ Core component for multi-LLM routing |
| **Portkey AI Gateway** | Enterprise LLM gateway — smart routing, observability, guardrails, semantic caching | ⭐ Alternative to LiteLLM with better observability |
| **Openkoda AI Reporting** | Open-source AI-powered reporting with natural language queries | 🔍 Worth evaluating for report generation |
| **LangChain** | LLM orchestration framework — chaining calls, data integration, memory | ⭐ For building LLM pipeline logic |
| **Metabase** | Open-source BI dashboard | 🔍 Could complement for data visualization |

## Technical Feasibility

### Available APIs & Services

| API/Service | Status | Rate Limits | Auth | Data Quality | Feasibility |
|---|---|---|---|---|---|
| **Sensor Tower API** | ✅ Already integrated (data in Postgres) | Per subscription tier | API Key | High (industry standard estimates) | ✅ Proven |
| **Apify Reddit Scraper** | ✅ Ready-made actor | ~1000 req/min | API Token | Medium (unstructured posts) | ✅ Ready |
| **Apify Discord Scraper** | ✅ Ready-made actor | Varies by actor | API Token | Medium (server-dependent) | ⚠️ May need server access |
| **Apify Facebook Scraper** | ✅ Ready-made actor | Limited by Facebook | API Token | Medium (public pages only) | ⚠️ Facebook anti-scraping |
| **Apify App Store Scraper** | ✅ Ready-made actor (200 reviews/sec) | High throughput | API Token | High (structured reviews) | ✅ Ready |
| **abei.gov.vn scraping** | ⚠️ Custom scraper needed | Public site, no limits | None | Medium (HTML parsing) | ⚠️ Site structure changes |
| **Minimax API** | ✅ Available (M2/M2.5) | Per subscription | API Key | High (OpenAI-compatible) | ✅ Ready |
| **Claude API** | ✅ Available (Sonnet/Opus) | Tier-based | API Key | Very High (best for analysis) | ✅ Ready |

### Multi-LLM Routing Architecture (Recommended)

Based on research, the optimal routing strategy is:

```
┌─────────────────────────────────────────────────┐
│                  Task Router                     │
│  (classify task complexity → route to model)     │
├─────────────────────────────────────────────────┤
│                                                  │
│  SIMPLE TASKS → Minimax M2/M2.5                 │
│  ├─ Data summarization                           │
│  ├─ Formatting & structuring                     │
│  ├─ Translation (Vi ↔ En)                        │
│  ├─ Basic Q&A about data                         │
│  └─ Template-based report sections               │
│                                                  │
│  COMPLEX TASKS → Claude Sonnet                   │
│  ├─ Cross-source synthesis                       │
│  ├─ Trend analysis & predictions                 │
│  ├─ Strategic recommendations                    │
│  ├─ Competitive analysis narratives              │
│  └─ Regulatory impact assessment                 │
│                                                  │
│  CRITICAL/LONG TASKS → Claude Opus               │
│  ├─ Full market report generation                │
│  ├─ Multi-market comparative deep-dives          │
│  └─ Complex regulatory analysis                  │
│                                                  │
└─────────────────────────────────────────────────┘
```

**Cost comparison (per 1M tokens):**

| Model | Input | Output | Best For |
|---|---|---|---|
| Minimax M2 | $0.26 | $1.00 | Simple tasks, high volume |
| Minimax M2.1 | $0.27 | $0.95 | Multilingual, concise outputs |
| Claude Haiku | $0.25 | $1.25 | Fast simple classification |
| Claude Sonnet | $3.00 | $15.00 | Complex analysis |
| Claude Opus | $15.00 | $75.00 | Critical deep analysis |

**Routing implementation options:**
1. **LiteLLM** (recommended for MVP) — Open-source, self-hosted, supports routing + fallbacks + cost tracking
2. **Portkey AI Gateway** — Better observability, but adds dependency
3. **Custom router** — Simple if/else based on task type, embedded in backend

### Vietnamese Gaming Regulation (G1–G4 License Categories)

Based on Decree 147/2024/ND-CP (effective Dec 25, 2024, superseding Decree 72/2013):

| Category | Description | Licensing Requirement |
|---|---|---|
| **G1** | Multi-player games interacting through enterprise server (MMORPGs) | "Decision to Publish" from MIC + content/script approval |
| **G2** | Single-player games interacting only with server (no player-to-player) | "Notice of Publishing Confirmation" to ABEI 30 days before launch |
| **G3** | Multi-player games without server interaction (peer-to-peer) | "Notice of Publishing Confirmation" to ABEI 30 days before launch |
| **G4** | Downloaded games with no interaction (offline/standalone) | "Notice of Publishing Confirmation" to ABEI 30 days before launch |

**Key regulatory data points to track:**
- Game approval decisions published on abei.gov.vn (2015–2025 archive available)
- G1 games: must have server in Vietnam, Vietnamese user data storage, minor playtime limits (60 min/day per game)
- All categories: foreign companies must establish local Vietnamese entity (49% foreign capital limit)
- Content restrictions: no violence, gambling, content contradicting Vietnamese cultural values
- App store compliance: Google Play / App Store must remove unlicensed games upon MIC request

**Data source**: `https://abei.gov.vn/danh-sach-cap-phep/69` — published license lists

### Recommended Libraries & Frameworks

| Layer | Recommended | Alternatives | Rationale |
|---|---|---|---|
| **Frontend** | Next.js 15 | Remix, Nuxt | SSR, API routes, excellent DX, Vercel deployment |
| **UI Components** | shadcn/ui | Radix + custom | Copy-pasteable components, great for MVP speed |
| **Rich Text Editor** | Tiptap or Plate | Draft.js, Slate | Collaborative editing, extensible, good LLM integration |
| **Backend API** | Next.js API Routes | Express, Fastify | Co-located with frontend, simpler deployment |
| **Database** | Postgres (existing) | — | Already has Sensor Tower data |
| **ORM** | Prisma or Drizzle | TypeORM | Type-safe, great DX, migration support |
| **LLM Gateway** | LiteLLM (self-hosted) | Portkey, custom | OpenAI-compatible, routing, cost tracking |
| **LLM SDK** | Vercel AI SDK | LangChain.js | Streaming, structured outputs, multi-provider |
| **Scraping** | Apify SDK | Puppeteer, Playwright | Pre-built actors, cloud execution, scheduling |
| **Auth** | NextAuth.js or Supabase Auth | Clerk | Simple for 5 users, email-based |
| **Hosting** | Vercel | Railway, self-hosted | Easy deployment, preview URLs, edge functions |
| **Job Queue** | Inngest or BullMQ | — | Background data ingestion, report generation |

### Infrastructure Requirements

| Component | MVP Requirement | Scale Requirement |
|---|---|---|
| **Web server** | Vercel Hobby/Pro ($0-20/mo) | Vercel Pro ($20/mo) |
| **Database** | Supabase Free/Pro or existing Postgres | Managed Postgres (Supabase Pro $25/mo) |
| **LLM APIs** | ~$50-100/mo (estimated for 5 users) | ~$200-500/mo at scale |
| **Apify** | Free tier (10K results/mo) or $49/mo | $149-499/mo |
| **Background jobs** | Inngest Free or Vercel Cron | Inngest Pro ($50/mo) |
| **Total MVP cost** | **~$100-200/mo** | **~$500-1,200/mo at scale** |

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **LLM hallucination in market analysis** | Technical | High | High | All LLM outputs are editable drafts; human review mandatory before finalization |
| **Facebook scraping blocked** | Integration | Medium | Medium | Facebook has aggressive anti-scraping; use Apify's managed actors with proxy rotation; have manual import fallback |
| **Discord server access required** | Integration | Medium | Low | Need server membership for scraping; ensure team has access to target servers |
| **abei.gov.vn site restructure** | Integration | Low | Medium | Use loose CSS selectors; add scraping health checks; maintain manual import option |
| **Claude API cost overrun** | Operational | Medium | Medium | Route 70%+ tasks to Minimax; set monthly budget alerts; cache repeated analyses |
| **Minimax API reliability** | Technical | Low | Medium | LiteLLM provides automatic fallback to Claude if Minimax is unavailable |
| **Data freshness mismatch** | Technical | Low | Low | Sensor Tower monthly, Apify weekly, regulation monthly — clearly label data timestamps in reports |
| **Team adoption resistance** | Operational | Low | Medium | Build simple, intuitive UI; involve team in template design; start with supervised report generation |
| **Vietnamese content processing by LLMs** | Technical | Medium | Medium | Test Minimax and Claude Vietnamese language quality; may need translation step for some content |
| **Regulatory data interpretation accuracy** | Technical | Medium | High | LLM regulation summaries must be verified by team; flag as "AI-generated, verify before publishing" |

## PoC Recommendations

Before full MVP development, validate these 3 key assumptions:

### PoC 1: Multi-LLM Report Quality (1-2 days)
- **Test**: Send identical market data to Minimax M2 and Claude Sonnet, compare output quality
- **Success criteria**: Minimax produces acceptable summaries (~80% quality of Claude) for simple tasks
- **Method**: Use LiteLLM to route identical prompts to both models; have team rate outputs blind

### PoC 2: Community Sentiment Pipeline (1-2 days)  
- **Test**: Run Apify Reddit + App Store scrapers → feed results to Claude for sentiment analysis
- **Success criteria**: Pipeline produces actionable sentiment summaries from 100+ posts/reviews
- **Method**: Set up Apify actors, store results, run Claude analysis, evaluate output

### PoC 3: Regulation Scraping (0.5-1 day)
- **Test**: Scrape abei.gov.vn game license list → parse → store in Postgres
- **Success criteria**: Successfully extract game names, license categories, dates from HTML
- **Method**: Write Apify custom actor or simple fetch + cheerio parser

## Key Insights

1. **LiteLLM is the optimal LLM gateway for MVP** — It's open-source, supports routing/fallbacks/cost tracking, and is OpenAI-compatible. Minimax's OpenAI-compatible API makes this seamless.

2. **Minimax M2 at $0.26/M input tokens is ~12x cheaper than Claude Sonnet** — Routing 70% of tasks (summarization, formatting, translation) to Minimax could reduce LLM costs by ~60-70%.

3. **Vietnamese regulation landscape just changed significantly** — Decree 147/2024 replaces the 11-year-old Decree 72/2013, making a regulation tracker more valuable than ever for market participants.

4. **No existing tool combines all 4 data sources** — The unique value proposition is the synthesis of market data + community sentiment + regulation + LLM analysis. No competitor offers this combination.

5. **The collaborative editing requirement is achievable** — Libraries like Tiptap or Plate provide production-ready collaborative rich text editing that integrates well with LLM-generated content.

## Sources

1. [Sensor Tower Platform](https://sensortower.com) — Mobile game analytics, Game Intelligence
2. [Apify Platform](https://apify.com) — Web scraping actors for Reddit, Discord, Facebook, App Store
3. [abei.gov.vn](https://abei.gov.vn/danh-sach-cap-phep/69) — Vietnamese game license lists (Cục PTTH)
4. [Minimax API](https://minimax.io) — LLM API, M2/M2.5 models
5. [Claude API (Anthropic)](https://docs.anthropic.com) — Claude Sonnet/Opus for deep analysis
6. [LiteLLM](https://litellm.ai) — Open-source LLM proxy with routing
7. [Portkey AI Gateway](https://portkey.ai) — Enterprise LLM gateway
8. [Decree 147/2024/ND-CP](https://game.gov.vn) — Vietnam gaming regulation
9. [GameRefinery](https://gamerefinery.com) — Mobile game feature analysis
10. [PocketGamer.biz](https://pocketgamer.biz) — Mobile gaming industry news and analysis
