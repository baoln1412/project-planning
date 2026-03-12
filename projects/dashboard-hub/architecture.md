# Architecture: Dashboard Hub

## Overview
Dashboard Hub uses a modern, serverless architecture centered around Next.js on Vercel. The system serves dual purposes: a B2C storefront for selling tracking dashboards, and an admin workspace for generating new dashboard components. User tracking data lives in their personal Google Sheets — the platform caches Sheet data in Supabase for performance and reads/writes through a secure proxy.

## Phase 1: MVP & Storefront Validation

### Design Goals
- **Zero Fixed Cost:** Use Vercel and Supabase free tiers.
- **Data Privacy:** Cache Google Sheets data in Supabase for performance, but user's Sheet remains source of truth.
- **Instant Access:** Stripe Webhooks auto-approve purchases — no manual admin gate.
- **Free Tier:** 1 free dashboard (Habit Tracker) to prove value before asking for payment.

### Architecture Diagram

```mermaid
graph TB
    subgraph "Users"
        User["👤 Customer"]
    end

    subgraph "Vercel"
        Next["Next.js 15<br>App Router"]
        API["API Routes<br>Stripe Webhook"]
        ISR["ISR<br>SEO Pages"]
    end

    subgraph "Supabase"
        Auth["Auth<br>Google OAuth"]
        DB[("PostgreSQL<br>Accounts + Purchases")]
        Cache[("sheet_cache<br>Cached Sheet Data")]
    end

    subgraph "External"
        Google["Google Sheets API"]
        Stripe["Stripe Checkout"]
    end

    User -->|Browse/Buy| Next
    User -->|OAuth Login| Auth
    User -->|Pays| Stripe
    Stripe -->|Webhook: auto-approve| API
    API --> DB
    Next -->|Reads cached data| Cache
    Next -->|Sync every 5-15 min| Google
    Google -->|Fresh data| Cache
    ISR -->|SEO product pages| Next
```

### Components
- **Next.js App Router**: Storefront rendering, dashboard views, secured API routes, ISR for SEO pages.
- **Auth.js**: Google OAuth with `drive.file` scope. Only accesses Sheets the user explicitly grants.
- **Supabase DB**: User profiles, dashboard catalog, purchases table, AND `sheet_cache` table.
- **sheet_cache**: Cached Google Sheets data. Refreshed every 5-15 minutes or on user demand. Eliminates Google API rate limit issues.
- **Stripe Webhooks**: Listens for `checkout.session.completed` — auto-toggles purchase access. No manual approval.
- **Google Sheets API**: Source of truth for user tracking data. Accessed via background sync, not on every page load.

### Key Data Flows

**Sheet Caching Flow:**
```mermaid
sequenceDiagram
    participant User
    participant Dashboard as Next.js Dashboard
    participant Cache as Supabase sheet_cache
    participant Sheets as Google Sheets API

    User->>Dashboard: Opens dashboard
    Dashboard->>Cache: Read cached data
    Cache-->>Dashboard: Return cached rows
    Dashboard-->>User: Render dashboard (fast)
    
    Note over Dashboard,Sheets: Background sync (every 5-15 min)
    Dashboard->>Sheets: Fetch latest data
    alt Sheet is valid
        Sheets-->>Cache: Update cache
    else Sheet is broken
        Note over Cache: Keep last-good data
        Dashboard-->>User: Show warning + cached data
    end
```

**Purchase Flow:**
```mermaid
sequenceDiagram
    participant User
    participant Store as Next.js Store
    participant Stripe
    participant Webhook as API Route
    participant DB as Supabase

    User->>Store: Click "Buy" on dashboard
    Store->>Stripe: Redirect to Checkout
    User->>Stripe: Complete payment
    Stripe->>Webhook: checkout.session.completed
    Webhook->>DB: Set is_approved = true
    Webhook-->>User: Redirect to dashboard (instant access)
```

### Estimated Cost: $0/mo

---

## Phase 2: Automation & Performance

### Trigger to Transition
- 3+ successful dashboard sales
- Google Sheets sync needs optimization for multiple concurrent users
- SEO traffic warrants dedicated content strategy

### Architecture Diagram

```mermaid
graph TB
    subgraph "Users"
        User["👤 Customers"]
    end

    subgraph "Vercel"
        Next["Next.js App"]
        Edge["Edge Functions<br>Rate Limiting"]
        ISR2["ISR + Blog<br>Content Marketing"]
    end

    subgraph "Supabase"
        DB[("PostgreSQL Primary")]
        Cache2[("sheet_cache<br>+ sync queue")]
        Auth2["Auth + RLS"]
    end

    subgraph "External"
        Google["Google Sheets API"]
        Stripe2["Stripe Webhooks"]
    end

    subgraph "AI Generation"
        IDE["Antigravity IDE<br>Local Generation"]
        Design["_design/ + _dashboards/"]
    end

    User --> Next
    Next --> Cache2
    Cache2 -->|"Background sync"| Google
    Stripe2 --> Next --> DB
    Auth2 --> Next
    IDE --> Design
    ISR2 -->|"SEO blog"| Next
```

### New Components
- **Google Stitch MCP** (`@_davideast/stitch-mcp`): Visual design layer — generate UI screens from prompts, extract "Design DNA" (fonts, colors, layouts), export React/HTML code. This replaces text-only prompting for dashboard generation.
- **AI Dashboard Generation**: Stitch designs → Design DNA → Antigravity adapts exported React to shadcn/ui + Recharts + Next.js. Google Sheets MCP auto-creates Master Sheet templates.
- **Sync Queue**: Instead of simple cron, a queue-based sync handles multiple users' Sheets efficiently.
- **Blog / Content Marketing**: SEO content pages built with ISR for organic traffic.
- **Edge Functions**: Rate limiting on API routes to prevent abuse.

### Estimated Cost: $45/mo
(Vercel Pro $20 + Supabase Pro $25)

---

## Phase 3: Scale & Multi-Tenancy

### Trigger to Transition
- Thousands of users
- You want to allow other creators to sell dashboards (marketplace model)

### Architecture Diagram

```mermaid
graph TB
    subgraph "Global Edge"
        CDN["Vercel Edge Network"]
        WAF["Web Application Firewall"]
    end

    subgraph "Application"
        API3["Next.js Serverless"]
        Workers["Background Sync Workers"]
    end

    subgraph "Data Layer"
        Primary[("Supabase Primary")]
        Replica[("Read Replica")]
        Cache3[("sheet_cache")]
    end

    subgraph "External"
        Google3["Google Sheets APIs"]
        StripeConnect["Stripe Connect"]
    end

    CDN --> WAF --> API3
    API3 --> Primary
    API3 --> Replica
    API3 --> Cache3
    Workers -->|"Batch sync"| Google3
    Workers --> Cache3
    StripeConnect -->|"Platform fee"| API3
```

### Scaling Strategy
- **Stripe Connect**: Route payouts to third-party dashboard creators while taking a platform fee.
- **Background Sync Workers**: Batch Sheet sync for all users — keep cache warm proactively.
- **Database Read Replicas**: Handle catalog listings and purchase validation at scale.

### Estimated Cost: $150+/mo

---

## Data Architecture

### ERD (Supabase)

```mermaid
erDiagram
    users {
        uuid id PK
        string email
        string google_refresh_token "Encrypted"
        datetime created_at
    }

    dashboards {
        uuid id PK
        string slug "e.g., reading-tracker"
        string title
        string description
        string master_sheet_url "Google Sheet template link"
        integer price_cents "0 = free tier"
        boolean is_active
        string seo_title
        string seo_description
    }

    purchases {
        uuid id PK
        uuid user_id FK
        uuid dashboard_id FK
        boolean is_approved "Auto-set by Stripe Webhook"
        string stripe_session_id
        datetime purchased_at
    }

    sheet_connections {
        uuid id PK
        uuid user_id FK
        uuid dashboard_id FK
        string sheet_url "User's cloned Sheet URL"
        datetime last_synced_at
        string sync_status "ok | error | stale"
    }

    sheet_cache {
        uuid id PK
        uuid connection_id FK
        jsonb data "Cached Sheet rows as JSON"
        jsonb last_good_data "Fallback if current sync fails"
        datetime cached_at
    }

    users ||--o{ purchases : "makes"
    dashboards ||--o{ purchases : "bought in"
    users ||--o{ sheet_connections : "connects"
    dashboards ||--o{ sheet_connections : "linked to"
    sheet_connections ||--|| sheet_cache : "cached in"
```

### Key Schema Changes from v1
- **`sheet_connections` table** (NEW): Tracks which user connected which Sheet to which dashboard. Stores sync status.
- **`sheet_cache` table** (NEW): Stores cached Sheet data as JSONB. Includes `last_good_data` for fallback when the user's Sheet is broken/modified.
- **`dashboards.price_cents = 0`**: Enables free tier dashboards (Habit Tracker).
- **`dashboards.seo_title/seo_description`**: SEO metadata per dashboard product page.
- **`purchases.stripe_session_id`**: Links purchase to Stripe session for audit trail.
- **Removed**: manual `is_approved` toggle is now auto-set by Stripe Webhook.
