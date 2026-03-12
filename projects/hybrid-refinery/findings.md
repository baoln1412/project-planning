# Findings: Hybrid Refinery

> Persistent memory file — updated across planning phases. Captures constraints, decisions, insights, and open questions.

## Constraints Discovered

### Technical
- **Sensor Tower API is enterprise-gated** — no public documentation. Must verify API access on current subscription before pipeline development begins.
- **DuckDB VSS is experimental** — labeled experimental by DuckDB team. Breaking changes between versions are possible. → **Decision**: Use Supabase pgvector for production vector search instead.
- **Google Ads Transparency Center has no official API** — must use third-party scrapers (Apify) or Playwright.
- **TikTok Ads API approval takes 1-2 weeks** and is more stringent as of Sep 2025. Initially EU-focused — VN data availability may be limited.
- **PhoBERT accuracy on gaming slang is unvalidated** — 92.74% on phone reviews, but gaming communities use heavy Viet-English code-switching and abbreviations.

### Business
- **Sensor Tower costs $25-40K/year** — migration to AppMagic ($400-1K/mo) or Appfigures ($450/mo) once pipeline is proven will save $15-30K/year.
- **Internal team only** — no multi-tenant requirements, Supabase auth sufficient for basic access control.

### Regulatory
- **Vietnamese Cybersecurity Law** applies to user PII, not public market data. Hybrid Refinery only processes public reviews, ads, and market statistics.
- **ABEI (abei.gov.vn)** publishes G1 license approvals publicly — scraping this is monitoring public government data.

## Key Market Insights

- VN mobile gaming: **$454M in 2025**, growing at **11.5% CAGR** to $712M by 2029
- **58.5M gamers**, 86.6% play on mobile — one of the highest mobile gaming penetration rates globally
- Vietnamese studios produced **37.3% of global game app downloads** in 2024
- Government targeting **$1B gaming revenue by 2030** with 5,000 developer training program
- Shift from hyper-casual to **hybrid-casual and midcore** monetization models

## Differentiation Strategy

No existing market intelligence tool combines:
1. **Vietnamese-language NLP** (PhoBERT sentiment analysis)
2. **Local Vietnamese gaming ecosystem** coverage (f.gvn.co, GameK)
3. **ABEI regulatory monitoring** (G1 license tracking)
4. **Multi-source data synthesis** (financial + sentiment + news + ads in one pipeline)

This combination is unmapped by Sensor Tower, data.ai, GameRefinery, or any other global tool.

## Decisions Made

| Decision | Rationale | Date |
|---|---|---|
| Keep Sensor Tower for now | Already subscribed. Migrate to AppMagic once pipeline proven. | 2026-03-11 |
| Next.js + Tailwind instead of Streamlit | Streamlit can't deploy on Vercel. Next.js provides better customization for report generation. | 2026-03-11 |
| Supabase pgvector over DuckDB VSS | DuckDB VSS is experimental. Gold Layer already in Supabase — pgvector is natural fit. | 2026-03-11 |
| Standard RAG over NotebookLM/Deep Research | NotebookLM MCP integration is undocumented and unreliable for production. | 2026-03-11 |
| Start with Meta Ads Library first | Free, well-documented API. TikTok needs approval. Google has no official API. | 2026-03-11 |
| f.gvn.co + GameK as Vietnamese news sources | User-specified. Only Vietnamese gaming-focused news sources. | 2026-03-11 |

## Open Questions

- [ ] **CRITICAL**: What Sensor Tower API access is available on the current subscription?
- [ ] PhoBERT accuracy on Vietnamese gaming reviews with slang — needs validation with sample dataset
- [ ] f.gvn.co site structure — is it a forum (dynamic JS) or static content? What anti-bot measures exist?
- [ ] GameK — does it have an RSS feed, or must we scrape?
- [ ] 50-word threshold for PhoBERT/Gemini routing — needs tuning with real gaming text
- [ ] What specific games/publishers should the MVP track initially?
