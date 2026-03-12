# Hybrid Refinery

## Summary

Hybrid Refinery is an automated market intelligence platform for the Vietnamese mobile gaming sector. It transforms raw market data (download/revenue estimates, player reviews, gaming news) into commercial-grade analytical reports for clients — replacing manual research with a structured data pipeline and LLM-powered insight generation.

The system uses a **hybrid architecture**: heavy data processing runs locally on a 32GB Mac for cost efficiency, while the enriched "Gold Layer" data is served via Supabase for persistence and RAG queries, with Vercel as a potential delivery endpoint.

## Problem Statement

Vietnamese mobile gaming market research is currently manual, fragmented, and slow. Analysts piece together data from multiple paid tools, news sites, and social channels to produce reports. This project automates the entire pipeline — from data ingestion to insight generation — producing faster, more consistent, data-backed reports at a fraction of the cost.

## Target Users

- **Primary**: Internal market research team (report authors)
- **Secondary**: Clients who receive the generated commercial-grade reports

> [!NOTE]
> The dashboards and tools are internal-only. Clients receive finalized reports, not direct platform access.

## Discovery Answers

| # | Question | Answer |
|---|---|---|
| 🎯 | **North Star** | Speed and quality of generating commercial-grade market intelligence reports with minimal operational cost |
| 🔌 | **Integrations** | Sensor Tower (primary, migrate to AppMagic later), f.gvn.co, GameK, Meta Ads, Google Ads, TikTok Ads, abei.gov.vn, PhoBERT (local NLP), Gemini API |
| 💾 | **Source of Truth** | DuckDB (local, Bronze/Silver layers) → Supabase (Gold Layer, vector storage for RAG) |
| 📦 | **Delivery Payload** | Internal Next.js + Tailwind dashboard (Vercel); PDF/Markdown commercial reports for clients |
| 🚫 | **Behavioral Rules** | Local-first processing (32GB Mac), Polars only (no Pandas), Apache Arrow for data transfer, Supabase for persistence, minimize cloud spend |

## Key Ideas / Features

### MVP (Phase 1) — Core Pipeline + 2 Data Sources

1. **Medallion Data Pipeline** (DuckDB + Polars + dlt)
   - Bronze: Raw JSON/CSV ingestion
   - Silver: Cleaned data with window functions (7-day MA, DoD/WoW deltas)
   - Gold: Enriched "Hybrid Matrix" pushed to Supabase

2. **Data Source 1: Sensor Tower** (current, with migration plan)
   - Using existing Sensor Tower subscription for estimated revenue, downloads, category rankings
   - **Migration path**: Transition to AppMagic (~$400-1000/mo) or Appfigures (~$450/mo) once pipeline is proven
   - Delivers: Estimated revenue, downloads, category rankings, publisher performance

3. **Data Source 2: Vietnamese Gaming News & Ad Intelligence**
   - **f.gvn.co** — Vietnamese gaming forum/news (web scraping via Playwright)
   - **GameK** — Vietnamese gaming news portal (RSS/scraping)
   - **Meta Ads Library API** — Facebook/Instagram ad creative tracking
   - **Google Ads Transparency Center** — Google ad creative tracking
   - **TikTok Ads** — TikTok ad creative intelligence
   - **abei.gov.vn** — ABEI regulatory/licensing portal (G1 license monitoring)

4. **Data Source 3: Market Research & Industry Reports**
   - **AppSamurai** — Vietnam Mobile Gaming Market Report
   - **Statista** — Vietnam Gaming Revenue Projections
   - **VietnamNet** — VN Gaming Industry Overview
   - **DigitalInAsia** — Vietnamese Game Studios Download Statistics
   - **IMARC Group** — Vietnam Mobile Gaming Market 2024-2033
   - **KenResearch** — Vietnam Gaming Market Trends
   - **Antom** — Vietnam Gamer Demographics

4. **Hybrid NLP Router**
   - Short text (<50 words): Local PhoBERT model (`wonrax/phobert-base-vietnamese-sentiment`)
   - Complex text (50+ words): Gemini API for deeper context extraction
   - Output: Sentiment tags, topic classification, intent detection

5. **Standard RAG Pipeline** (replaces NotebookLM/Deep Research)
   - DuckDB `vss` vector extensions for embeddings storage
   - Gemini API for synthesis and Q&A
   - Enables "talk to the data" chat interface

6. **Next.js + Tailwind Dashboard** (internal, deployed on Vercel)
   - Contextualized trend visualizations
   - Interactive LLM chat (RAG-powered)
   - Report generation tooling
   - Supabase as backend (reads Gold Layer directly)

### Phase 2 — Expanded Intelligence

7. **App Store & Google Play Reviews** — Player sentiment NLP via `google-play-scraper`
8. **Automated Alert Scenarios** (with configurable thresholds):
   - Gacha Backlash: Revenue spike + sentiment drop
   - Silent Churn: Revenue decline + zero engagement
   - Viral Optimization: Download spike + viral social topic

### Phase 3 — Social & Deep Analytics

9. **Community & Social Listening** — YouTube Data API, Facebook Groups (via official APIs where possible)
10. **Payment Portal Monitoring** — Zing, MoMo, Garena top-up tracking
11. **Sensor Tower Migration** — Transition to AppMagic/Appfigures for cost savings

## Architecture Overview

```
┌─────────────────────────────────────┐
│        LOCAL MAC (32GB)             │
│  ┌─────────┐  ┌─────────────────┐  │
│  │   dlt   │→ │  DuckDB         │  │
│  │ (ingest)│  │  Bronze → Silver│  │
│  └─────────┘  └────────┬────────┘  │
│                         │           │
│  ┌──────────┐  ┌────────▼────────┐  │
│  │ PhoBERT  │← │  Polars         │  │
│  │ (local   │  │  (transforms)   │  │
│  │  NLP)    │  │                 │  │
│  └──────────┘  └────────┬────────┘  │
│                         │ .arrow()  │
└─────────────────────────┼───────────┘
                          │
              ┌───────────▼───────────┐
              │     SUPABASE          │
              │  Gold Layer (Postgres)│
              │  Vector Store (RAG)   │
              │  Auth (if needed)     │
              └───────────┬───────────┘
                          │
              ┌───────────▼───────────┐
              │       VERCEL          │
              │  Next.js + Tailwind   │
              │  Internal Dashboard   │
              │  Report Generation    │
              └───────────────────────┘
```

> [!IMPORTANT]
> **Cost Migration Plan**: Currently using Sensor Tower. Once the pipeline is proven, migrate to **AppMagic** (~$400-1000/mo) or **Appfigures** (~$450/mo) to reduce data costs from $25-40K/year to ~$5-12K/year.

## Cost Efficiency Strategy

| Component | Cost (Current) | Cost (After Migration) | Notes |
|---|---|---|---|
| Sensor Tower | ~$2K-3.3K/mo | → AppMagic ~$400-1000/mo | Market intelligence data |
| Supabase | Free tier → $25/mo | Same | Gold Layer + vector storage |
| Gemini API | Pay-per-use (~$5-50/mo) | Same | Complex NLP + RAG synthesis |
| PhoBERT | Free (local) | Same | Short-text sentiment analysis |
| Vercel | Free tier → $20/mo | Same | Next.js dashboard hosting |
| DuckDB/Polars/dlt | Free (OSS) | Same | Local pipeline |
| **Total MVP** | **~$2K-3.4K/mo** | **~$450-1095/mo** | Major savings post-migration |

## Resolved Questions

- [x] ~~Confirm AppMagic vs. Sensor Tower~~ → Keeping Sensor Tower for now, migrate later
- [x] ~~Clarify Vercel's role~~ → Hosts Next.js + Tailwind internal dashboard
- [x] ~~Define Vietnamese news sources~~ → f.gvn.co, GameK, Meta/Google/TikTok Ads, abei.gov.vn

## Open Questions

- [ ] PhoBERT accuracy validation on Vietnamese gaming slang/abbreviations
- [ ] Sensor Tower API access level — which endpoints/data are available on current plan?
- [ ] f.gvn.co scraping feasibility — anti-bot measures, data structure, update frequency

## Status

🟡 Planning

## Created

2026-03-11
