# Tech Stack: Hybrid Refinery

## Summary

The stack is built around a **local-first Python pipeline** (dlt + DuckDB + Polars) for data processing and NLP, with **Supabase** as the single cloud dependency for Gold Layer storage, vector search (pgvector), and auth. The dashboard is **Next.js + Tailwind** deployed on **Vercel**. **Gemini API** handles complex NLP and RAG synthesis, while **PhoBERT** runs locally for free Vietnamese sentiment analysis.

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| **Language** | Python | 3.12+ | Required by dlt, PhoBERT, Polars; richest ML/data ecosystem |
| **Data Ingestion** | dlt | latest | Open-source ETL, DuckDB destination, schema inference, incremental loading |
| **Local OLAP** | DuckDB | 1.x | In-process analytics, TB-scale, zero-copy Arrow integration |
| **Data Transforms** | Polars | 1.x | Rust-based, 10-100x faster than Pandas, lazy execution, memory-safe |
| **Data Transfer** | Apache Arrow | — | Zero-copy between DuckDB ↔ Polars (`.arrow()`) |
| **Local NLP** | PhoBERT | `wonrax/phobert-base` | Free Vietnamese sentiment, 92.74% accuracy on VN reviews |
| **Cloud LLM** | Gemini API | 2.0 Flash + Pro | Cost-effective, Vietnamese text support, native embeddings |
| **Embeddings** | Gemini `text-embedding-004` | latest | Generates vectors for RAG pipeline |
| **Cloud DB** | Supabase (PostgreSQL) | 15+ | Gold Layer + pgvector + auth + real-time subscriptions |
| **Vector Search** | pgvector (via Supabase) | 0.7+ | Production-ready, integrated with Gold Layer — no separate vector DB |
| **Frontend** | Next.js | 15.x | App Router, SSR, API routes for RAG chat |
| **Styling** | Tailwind CSS | 4.x | Utility-first, rapid dashboard development |
| **Charts** | Recharts | latest | React-native, composable chart components |
| **Hosting** | Vercel | — | Free tier, seamless Next.js, edge functions |
| **Scraping** | Playwright (Python) | latest | JS rendering for f.gvn.co, GameK, abei.gov.vn |
| **CI/CD** | GitHub Actions | — | Free for public repos, pipeline automation |
| **Monitoring** | Sentry | — | Error tracking across pipeline + dashboard |

## Detailed Evaluation

### Cloud Database (Gold Layer)

| Criterion | Supabase | Neon | Firebase |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Community | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Weighted Total** | **4.50** | **3.80** | **3.70** |

**Recommendation**: Supabase — pgvector replaces DuckDB VSS, built-in auth and real-time, excellent Next.js integration, generous free tier.

### LLM for Analysis

| Criterion | Gemini API | GPT-4o | Claude API |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Community | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.55** | **4.20** | **3.80** |

**Recommendation**: Gemini — Flash model is the most cost-effective for high-volume text processing, native `text-embedding-004` for RAG, excellent Vietnamese language support.

### Frontend Framework

| Criterion | Next.js | SvelteKit |
|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Community | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.85** | **3.90** |

**Recommendation**: Next.js — Best-in-class Vercel + Supabase integration, largest component ecosystem for dashboards.

## Cost Estimate

| Service | Free/Hobby | Production Est. | After ST Migration |
|---|---|---|---|
| Sensor Tower | N/A (existing) | ~$2-3.3K/mo | → AppMagic $400-1K/mo |
| Supabase | Free (500MB) | $25/mo (Pro) | Same |
| Gemini API | Free tier | $5-50/mo | Same |
| Vercel | Hobby (free) | $20/mo (Pro) | Same |
| Sentry | Free (5K events) | $26/mo | Same |
| GitHub Actions | Free (2K min) | Free | Same |
| OSS (DuckDB, Polars, PhoBERT, dlt) | Free | Free | Free |
| **Total** | **~$2K/mo** | **~$2.1-3.5K/mo** | **~$475-1,120/mo** |

## Alternatives Considered

- **Pandas** — Rejected per user constraint. Polars is faster and more memory-efficient.
- **DuckDB VSS** — Replaced by Supabase pgvector due to experimental status.
- **Streamlit** — Replaced by Next.js for Vercel deployment compatibility and UI customization.
- **NotebookLM/Deep Research MCP** — Replaced by standard RAG (pgvector + Gemini) for reliability.
- **SQLite** — DuckDB is purpose-built for OLAP; SQLite is OLTP-optimized.
