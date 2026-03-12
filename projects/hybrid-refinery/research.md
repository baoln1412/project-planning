# Research: Hybrid Refinery

## Executive Summary

The Vietnamese mobile gaming market represents a substantial and growing opportunity for market intelligence tooling. At **$454M in 2025** with an **11.5% CAGR**, and backed by **58.5M gamers** (86.6% on mobile), Vietnam is one of the fastest-growing gaming markets globally. The government has recognized gaming as a national priority, targeting $1B in gaming revenue by 2030.

The market intelligence space is dominated by expensive enterprise platforms (Sensor Tower at $25-40K/year, data.ai at similar price points), with emerging cost-efficient alternatives like AppMagic ($400-1000/mo). No existing tool specifically targets the Vietnamese mobile gaming sector with localized sentiment analysis and regulatory monitoring — this is Hybrid Refinery's core differentiation opportunity.

The proposed tech stack (dlt + DuckDB + Polars + Supabase) is well-validated for medium-scale analytical workloads. The primary risk is verifying Sensor Tower API access on the current subscription plan.

## Market Analysis

### Market Size & Trends

| Metric | Value | Source |
|---|---|---|
| VN Gaming Industry Revenue (2025) | $1.66B | Statista |
| VN Mobile Gaming Revenue (2025) | $454M | AppSamurai |
| VN Mobile Gaming Revenue (2029) | $712M | Statista |
| CAGR 2025-2030 | 11.5% | AppSamurai |
| Total VN Gamers | 58.5M | Antom |
| Mobile Gamers (% of total) | 86.6% | DigitalInAsia |
| Smartphone Penetration | 84%+ | IMARC Group |
| Internet Penetration | 75%+ (70M+ users) | KenResearch |

**Key growth drivers:**
- Mobile payment systems projected to reach $15-20B in transaction volume
- Government targeting $1B gaming revenue by 2030 with 5,000 developer training commitment
- Vietnamese studios accounted for 37.3% of all global game app downloads in 2024
- Shift from hyper-casual to hybrid-casual and midcore monetization models
- 5G rollout reducing latency for cloud gaming

### Target Demographics

- **Primary consumers**: 18-35 year old Vietnamese mobile gamers
- **Key genres**: Strategy games (growing), RPGs, casual/hybrid-casual
- **Spending trends**: Consumer spending per download is rising despite declining total downloads
- **Emerging markets**: Turkey, Mexico, India, Thailand, Saudi Arabia driving global growth

### Demand Signals

- Vietnamese gaming studios exported more downloads than any other country globally in 2024
- Esports tournaments and gaming influencer ecosystem are well-established
- Growing interest in localized game content and strategic publisher partnerships

## Competitor Analysis

### Direct Competitors

| Name | Pricing | Strengths | Weaknesses | VN Focus |
|---|---|---|---|---|
| **Sensor Tower** | $25-40K/yr | Comprehensive data, Game IQ, Live Ops insights | Expensive, API access gated | No |
| **data.ai** | Similar to ST | Industry standard, broad coverage | Expensive, complex UI | No |
| **GameRefinery** | Enterprise | Feature-level game deconstruction, 100K+ games | Gaming-only, no sentiment | No |
| **AppMagic** | $400-1K/mo | Gaming-focused, 50+ API endpoints, affordable | Smaller dataset than ST | No |

### Indirect Competitors

| Name | Pricing | Strengths | Weaknesses |
|---|---|---|---|
| **Appfigures** | $450/mo (Grow) | REST API, 1K free req/day, affordable | General-purpose, not gaming-specific |
| **MobileAction** | Quote-based | Publisher performance, market trends | Limited gaming intelligence |
| **Similarweb** | Freemium | App intelligence + web data | Broad focus, not gaming-specific |
| **AppTweak** | Cost-effective | ASO + market intelligence combined | More ASO than market intelligence |

### Hybrid Refinery's Differentiation

> [!IMPORTANT]
> **No existing tool combines**: Vietnamese NLP sentiment analysis + local gaming news aggregation + ad creative intelligence + regulatory monitoring (ABEI) in a single platform. This is the core differentiation.

- **Localized NLP**: PhoBERT for Vietnamese sentiment — no competitor offers this
- **Vietnamese ecosystem coverage**: f.gvn.co, GameK, ABEI portal — unmapped by global tools
- **Cost efficiency**: Local processing + Supabase vs. full-cloud SaaS pricing
- **Data synthesis**: Medallion pipeline joining financial data with sentiment and news

## Technical Feasibility

### Data Pipeline: dlt + DuckDB + Polars

| Component | Status | Notes |
|---|---|---|
| **dlt** | ✅ Production-ready | Open-source, supports DuckDB destination natively, schema inference, incremental loading |
| **DuckDB** | ✅ Production-ready | In-process OLAP, handles TB-scale data out-of-core, zero-copy Arrow integration |
| **Polars** | ✅ Production-ready | Rust-based, 10-100x faster than Pandas, lazy execution, multithreaded |
| **Arrow transfer** | ✅ Validated | DuckDB ↔ Polars via `.arrow()` for zero-copy memory safety |
| **Medallion pattern** | ✅ Well-documented | Bronze→Silver→Gold pattern validated with DuckDB+Polars in multiple production deployments |

### NLP: PhoBERT + Gemini

| Component | Status | Notes |
|---|---|---|
| **PhoBERT** (`wonrax/phobert-base-vietnamese-sentiment`) | ⚠️ Needs validation | 92.74% accuracy on VN phone reviews. Gaming slang accuracy unknown |
| **Gemini API** | ✅ Available | Pay-per-use, handles complex Vietnamese text well |
| **Hybrid router (50-word threshold)** | ⚠️ Needs tuning | Threshold may need adjustment based on gaming text characteristics |

### RAG Pipeline

| Component | Status | Notes |
|---|---|---|
| **DuckDB VSS** | ⚠️ Experimental | HNSW indexing works, but labeled experimental. Breaking changes possible |
| **Supabase pgvector** | ✅ Production-ready | **Recommended alternative** — Gold Layer already in Supabase, pgvector is natural fit |
| **Gemini embeddings** | ✅ Available | `text-embedding-004` model for generating embeddings |

### Data Source APIs

| Source | Access Method | Status | Notes |
|---|---|---|---|
| **Sensor Tower** | API / CSV export | ⚠️ Verify access | Enterprise-gated API, no public docs. Must verify current plan includes API |
| **Meta Ads Library** | Graph API `/ads_archive` | ✅ Available | Free, requires developer app + access token. Gaming ads accessible without verification |
| **TikTok Ads Library** | Commercial Content API | ⚠️ Approval needed | 1-2 week approval. EU-focused initially, VN data may be limited |
| **Google Ads Transparency** | No official API | ❌ Must scrape | Use third-party (Apify) or Playwright scraping |
| **f.gvn.co** | Playwright scraping | ⚠️ Untested | Forum with dynamic content — JS rendering required |
| **GameK** | RSS / Playwright | ⚠️ Untested | Vietnamese gaming news — check for RSS feed availability |
| **abei.gov.vn** | Playwright scraping | ⚠️ Untested | Government portal — typically stable HTML but may have CAPTCHAs |

### Dashboard: Next.js + Supabase + Vercel

| Component | Status | Notes |
|---|---|---|
| **Next.js** | ✅ Production-ready | App Router, Server Components, API routes for RAG |
| **Tailwind CSS** | ✅ Production-ready | Rapid UI development |
| **Supabase client** | ✅ Production-ready | `@supabase/supabase-js` for real-time data access |
| **Vercel** | ✅ Production-ready | Seamless Next.js deployment, free tier sufficient for internal use |

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Sensor Tower API access unavailable on current plan | Technical | Medium | **High** | Verify immediately. If blocked, fast-track AppMagic migration |
| Web scraping fragility (f.gvn.co, GameK, abei.gov.vn) | Technical | High | Medium | Playwright + robust error handling + retry logic. RSS fallbacks where available |
| PhoBERT low accuracy on gaming slang | Technical | Medium | Medium | Validate on sample dataset early. Use Gemini for all text initially as fallback |
| DuckDB VSS breaking changes | Technical | Low | High | Use **Supabase pgvector** instead for production. DuckDB VSS for local dev only |
| TikTok/Google Ads API access limitations | Technical | Medium | Medium | Start with Meta Ads (most accessible). Add others incrementally |
| Single Mac hardware failure | Operational | Low | High | Gold Layer persisted in Supabase. DuckDB files backed up to cloud storage. Pipeline scripts in Git |
| Gemini API cost spikes | Business | Medium | Low | Budget alerts, request batching, caching. PhoBERT handles short text for free |
| Vietnamese data privacy laws | Legal | Low | Medium | Only process public data (reviews, ads, market data). No user PII |

## Key Insights

- **#1 priority action**: Verify Sensor Tower API access level on current subscription before writing any pipeline code
- **Cost savings opportunity**: Switch Gold Layer vectors from DuckDB VSS to Supabase pgvector — eliminates experimental risk AND simplifies architecture (one less component)
- **PhoBERT is a competitive moat**: No competitor offers local Vietnamese NLP — validate accuracy and this becomes a defensible advantage
- **Ad intelligence is the bonus**: Meta Ads Library is free and accessible — high value, low cost. Start here for ad tracking
- **Vietnamese ecosystem data is unmapped**: f.gvn.co, GameK, ABEI are unique data sources no global tool covers

## Sources

1. AppSamurai — Vietnam Mobile Gaming Market Report 2025
2. Statista — Vietnam Gaming Revenue Projections
3. VietnamNet — VN Gaming Industry Overview
4. DigitalInAsia — Vietnamese Game Studios Download Statistics
5. IMARC Group — Vietnam Mobile Gaming Market 2024-2033
6. KenResearch — Vietnam Gaming Market Trends
7. Antom — Vietnam Gamer Demographics
8. Sensor Tower — Connect API & Gaming Insights
9. data365.co — Meta Ads Library API Documentation
10. TikTok Developer Platform — Commercial Content API
11. DuckDB.org — VSS Extension Documentation
12. dlthub.com — dlt + DuckDB Integration Guide
13. HuggingFace — wonrax/phobert-base-vietnamese-sentiment
14. CTU.edu.vn — PhoBERT Sentiment Analysis Performance Study
