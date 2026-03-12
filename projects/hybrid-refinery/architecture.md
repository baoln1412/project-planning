# Architecture: Hybrid Refinery

## Overview

Hybrid Refinery uses a **hybrid local-cloud architecture** — heavy data processing (ETL, NLP, transforms) runs locally on a 32GB Mac for cost efficiency, while the enriched Gold Layer data is served via Supabase (PostgreSQL + pgvector) for persistence and cloud access. The dashboard is a Next.js app deployed on Vercel that reads from Supabase. This approach minimizes cloud spend while maintaining production-grade data availability.

## Phase 1: Test / MVP

### Design Goals
- Fastest path to a working pipeline that ingests Sensor Tower data + Vietnamese gaming news
- Local processing for compute-heavy tasks (NLP, data transforms)
- Supabase as the single cloud dependency (Gold Layer + vector search)
- Internal Next.js dashboard on Vercel for team access

### Architecture Diagram

```mermaid
graph TB
    subgraph "Data Sources"
        ST["📊 Sensor Tower<br>API / CSV"]
        News["📰 f.gvn.co / GameK<br>Playwright Scraping"]
        Reports["📄 Market Reports<br>AppSamurai, Statista, etc."]
    end

    subgraph "Local Mac - 32GB"
        DLT["⚙️ dlt<br>Data Ingestion"]
        Bronze[("🥉 DuckDB<br>Bronze Layer")]
        Silver[("🥈 DuckDB<br>Silver Layer")]
        Polars["🔄 Polars<br>Transforms + Arrow"]
        PhoBERT["🧠 PhoBERT<br>VN Sentiment"]
        Gemini_Local["🤖 Gemini API<br>Complex NLP"]
    end

    subgraph "Supabase Cloud"
        Gold[("🥇 PostgreSQL<br>Gold Layer")]
        PGVector["🔍 pgvector<br>Vector Search"]
        Auth["🔐 Auth"]
    end

    subgraph "Vercel"
        Dashboard["🖥️ Next.js<br>Dashboard + RAG Chat"]
    end

    ST --> DLT
    News --> DLT
    Reports --> DLT
    DLT --> Bronze
    Bronze --> Polars
    Polars --> Silver
    Silver --> PhoBERT
    Silver --> Gemini_Local
    PhoBERT --> Polars
    Gemini_Local --> Polars
    Polars -->|".arrow()"| Gold
    Polars -->|"embeddings"| PGVector
    Gold --> Dashboard
    PGVector --> Dashboard
    Auth --> Dashboard
```

### Components
| Component | Role | Cost |
|---|---|---|
| dlt | Extract from Sensor Tower, scrape news sites, ingest reports | Free |
| DuckDB (Bronze/Silver) | Raw and cleaned data storage, SQL window functions | Free |
| Polars | Data transforms, zero-copy Arrow transfer | Free |
| PhoBERT | Vietnamese sentiment tagging for short text | Free |
| Gemini API | Complex NLP, embeddings, RAG synthesis | ~$5-50/mo |
| Supabase | Gold Layer (PostgreSQL), pgvector, Auth | Free → $25/mo |
| Next.js + Vercel | Internal dashboard, RAG chat interface | Free → $20/mo |

### Estimated Cost: $2K-3.3K/mo (with Sensor Tower) → $30-95/mo (infra only)

## Phase 2: Production

### Trigger to Transition
- Pipeline runs reliably for 30+ days with daily data updates
- Team uses dashboard regularly for report generation
- 3+ data sources integrated and producing insights
- Sensor Tower migration to AppMagic initiated

### Architecture Diagram

```mermaid
graph TB
    subgraph "Data Sources"
        ST2["📊 AppMagic API"]
        News2["📰 VN Gaming News"]
        Ads["📢 Meta / TikTok Ads"]
        Reviews["⭐ App Store Reviews"]
        Reports2["📄 Market Reports"]
        ABEI["🏛️ abei.gov.vn"]
    end

    subgraph "Local Mac - 32GB"
        Scheduler["⏰ Cron / Orchestrator"]
        DLT2["⚙️ dlt Pipeline"]
        Bronze2[("🥉 DuckDB Bronze")]
        Silver2[("🥈 DuckDB Silver")]
        Polars2["🔄 Polars"]
        PhoBERT2["🧠 PhoBERT"]
        Gemini2["🤖 Gemini API"]
        Backup["💾 Cloud Backup<br>R2 / S3"]
    end

    subgraph "Supabase Cloud"
        Gold2[("🥇 PostgreSQL Gold")]
        PGV2["🔍 pgvector"]
        Auth2["🔐 Auth + RLS"]
        RT["📡 Realtime"]
    end

    subgraph "Vercel"
        Dash2["🖥️ Next.js Dashboard"]
        API2["🔌 API Routes"]
        Edge2["⚡ Edge Functions"]
    end

    subgraph "Observability"
        Sentry2["🐛 Sentry"]
        Logs2["📋 Structured Logs"]
    end

    ST2 & News2 & Ads & Reviews & Reports2 & ABEI --> DLT2
    Scheduler --> DLT2
    DLT2 --> Bronze2
    Bronze2 --> Polars2 --> Silver2
    Silver2 --> PhoBERT2 & Gemini2
    PhoBERT2 & Gemini2 --> Polars2
    Polars2 -->|".arrow()"| Gold2
    Polars2 -->|"embeddings"| PGV2
    Bronze2 & Silver2 -->|"nightly"| Backup
    Gold2 & PGV2 --> Dash2
    Auth2 --> Dash2
    RT --> Dash2
    Dash2 --> Sentry2
    DLT2 --> Logs2
```

### New Components
| Addition | Purpose |
|---|---|
| Cron scheduler | Automated daily pipeline runs |
| Cloud backup (R2/S3) | Nightly DuckDB file backups for disaster recovery |
| Structured logging | Pipeline observability and debugging |
| Sentry | Error tracking across pipeline + dashboard |
| Supabase RLS | Row-level security for team access control |
| Realtime | Live dashboard updates when Gold Layer changes |
| Edge functions | Rate-limited API endpoints for RAG chat |

### Security Measures
- Supabase RLS policies for data access control
- API keys stored in Vercel environment variables
- Sentry for error alerting
- Git-versioned pipeline code
- Nightly DuckDB backups to object storage

### Estimated Cost: $475-1,120/mo (after AppMagic migration)

## Phase 3: Scale

### Trigger to Transition
- Team size exceeds 5 concurrent dashboard users
- Data volume exceeds 100GB Gold Layer
- Pipeline processing time exceeds 4 hours per run
- Client demand for programmatic API access to intelligence data

### Architecture Diagram

```mermaid
graph TB
    subgraph "Data Sources"
        Sources3["📊 All 7+ Sources"]
    end

    subgraph "Processing Cluster"
        Orchestrator["⏰ Dagster / Prefect<br>Orchestration"]
        Workers["⚙️ 2x Mac Workers<br>Pipeline Processing"]
        NLP3["🧠 PhoBERT + Gemini<br>NLP Cluster"]
    end

    subgraph "Supabase Cloud"
        Primary3[("🥇 Primary DB<br>Write")]
        Replica3[("📖 Read Replica")]
        PGV3["🔍 pgvector<br>HNSW Index"]
        Auth3["🔐 Auth + RLS"]
        Edge3["⚡ Edge Functions"]
    end

    subgraph "CDN + Delivery"
        Vercel3["🌐 Vercel<br>Next.js + Edge"]
        Cache3["💨 Vercel Edge Cache"]
        ClientAPI["🔌 Public API<br>Rate-Limited"]
    end

    subgraph "Observability"
        Sentry3["🐛 Sentry"]
        Grafana3["📊 Grafana Cloud"]
        Alerts3["🔔 PagerDuty / Slack"]
    end

    Sources3 --> Orchestrator --> Workers
    Workers --> NLP3
    Workers -->|".arrow()"| Primary3
    NLP3 -->|"embeddings"| PGV3
    Primary3 --> Replica3
    Replica3 --> Vercel3
    PGV3 --> Vercel3
    Auth3 --> Vercel3
    Vercel3 --> Cache3
    Vercel3 --> ClientAPI
    Workers --> Sentry3
    Vercel3 --> Grafana3
    Sentry3 --> Alerts3
```

### Scaling Strategy
| Component | Scaling Approach |
|---|---|
| Pipeline processing | Add second Mac worker, distribute data sources across workers |
| Database | Supabase read replicas for dashboard queries, primary for writes |
| Vector search | HNSW index optimization, filtered search for performance |
| Dashboard | Vercel Edge Cache for common queries, ISR for static pages |
| API | Rate-limited public API for client programmatic access |
| Orchestration | Migrate from cron to Dagster/Prefect for complex DAG management |

### Performance Optimizations
- Incremental pipeline runs (dlt handles this natively)
- Materialized views in PostgreSQL for common dashboard queries
- Vercel ISR for report pages (regenerate every hour)
- pgvector HNSW index with filtered search for targeted RAG
- Apache Arrow for all data transfer between components

### Estimated Cost: $1,500-3,000/mo (with AppMagic + Supabase Team + Vercel Pro)

## Data Architecture

### ERD

```mermaid
erDiagram
    GAMES {
        uuid id PK
        string app_id
        string name
        string publisher
        string platform
        string genre
        timestamp created_at
    }

    MARKET_DATA {
        uuid id PK
        uuid game_id FK
        date date
        float revenue_estimate
        int downloads_estimate
        int category_rank
        float revenue_7d_ma
        float revenue_dod_pct
        float revenue_wow_pct
        string source
    }

    SENTIMENT {
        uuid id PK
        uuid game_id FK
        date date
        string text
        string source
        string sentiment_label
        float sentiment_score
        string[] topics
        string model_used
    }

    NEWS {
        uuid id PK
        date published_date
        string title
        string source
        string url
        string summary
        string[] related_games
        string sentiment
    }

    AD_CREATIVES {
        uuid id PK
        uuid game_id FK
        string platform
        date start_date
        date end_date
        string creative_type
        string creative_url
        string ad_text
        int days_running
    }

    REGULATORY {
        uuid id PK
        string license_type
        string game_name
        string publisher
        date approval_date
        string status
        string source_url
    }

    EMBEDDINGS {
        uuid id PK
        uuid source_id FK
        string source_type
        vector embedding
        string text_chunk
        jsonb metadata
    }

    GAMES ||--o{ MARKET_DATA : "has"
    GAMES ||--o{ SENTIMENT : "has"
    GAMES ||--o{ AD_CREATIVES : "has"
    NEWS }o--o{ GAMES : "mentions"
    EMBEDDINGS }o--|| SENTIMENT : "embeds"
    EMBEDDINGS }o--|| NEWS : "embeds"
```

### Key Data Flows

1. **Sensor Tower → Bronze → Silver → Gold (MARKET_DATA)**
   - Daily automated ingestion via dlt
   - Silver layer: 7-day moving averages, DoD/WoW deltas via DuckDB SQL
   - Gold layer: Enriched records pushed to Supabase

2. **News/Reviews → Bronze → Silver (+ NLP) → Gold (SENTIMENT, NEWS)**
   - Playwright scrapes f.gvn.co, GameK
   - PhoBERT tags short text sentiment locally
   - Gemini processes complex text for topics + intent
   - Embeddings generated and stored in pgvector

3. **Ads → Bronze → Gold (AD_CREATIVES)**
   - Meta Ads Library API fetches competitor ad data
   - Stored with creative metadata and run duration

## API Design

### Key Endpoints (Next.js API Routes)

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/games` | List tracked games with latest metrics |
| GET | `/api/games/[id]/trends` | Time-series revenue/download data |
| GET | `/api/games/[id]/sentiment` | Sentiment analysis results |
| GET | `/api/news` | Latest gaming news with sentiment |
| GET | `/api/ads/[game_id]` | Ad creatives for a specific game |
| POST | `/api/chat` | RAG chat — query Gold Layer via pgvector |
| GET | `/api/alerts` | Triggered alert scenarios |
| POST | `/api/reports/generate` | Generate PDF/Markdown report |

### Authentication Flow

```mermaid
sequenceDiagram
    participant User as Team Member
    participant Dashboard as Next.js Dashboard
    participant Supabase as Supabase Auth
    participant DB as Supabase DB

    User->>Dashboard: Login (email/password)
    Dashboard->>Supabase: signInWithPassword()
    Supabase-->>Dashboard: JWT + session
    Dashboard->>DB: Query Gold Layer (with JWT)
    DB-->>DB: RLS policy check
    DB-->>Dashboard: Authorized data
    Dashboard-->>User: Render dashboard
```
