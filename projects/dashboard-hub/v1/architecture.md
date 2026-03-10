# Architecture: Dashboard Hub

## Overview
Dashboard Hub uses a modern, serverless architecture centered around Next.js on Vercel. The system serves dual purposes: a B2C storefront for selling tracking dashboards, and a B2B admin portal where Antigravity AI generates new dashboard components. User tracking data never touches our database; our Next.js backend securely proxies requests from the user's browser to their personal Google Sheet using OAuth credentials stored in Supabase.

## Phase 1: MVP & Storefront Validation

### Design Goals
- **Zero Fixed Cost:** Use Vercel and Supabase free tiers.
- **Data Privacy:** Read/write directly to Google Sheets; do not store tracking data in Supabase.
- **Manual Payment Security:** Stripe checkout links, but admin manually flips the `has_access` flag in Supabase to prevent unauthorized scraping or data leaks.

### Architecture Diagram

```mermaid
graph LR
    subgraph "External Providers"
        Google[Google Sheets API]
        Stripe[Stripe Checkout]
    end

    subgraph "Vercel Edge"
        Next[Next.js 15 App]
        NextAuth[Auth.js<br>Manages Tokens]
    end

    subgraph "Supabase"
        Auth[User Accounts]
        DB[(PostgreSQL<br>Catalog & Purchases)]
    end
    
    subgraph "Users"
        User[👤 Customer]
        Admin[👤 Admin (You)]
    end

    User -->|Views Store| Next
    User -->|Pays| Stripe
    User -->|OAuth Login| NextAuth
    NextAuth <--> Auth
    Next <--> DB
    Next -->|Reads/Writes Data| Google
    
    Admin -->|Manual Approval| DB
```

### Components
- **Next.js App Router**: Handles storefront rendering, user dashboard views, and secured API routes.
- **Auth.js**: Manages "Sign in with Google," requesting the `drive.file` scope, and storing the short-lived access tokens.
- **Supabase DB**: Holds user profiles, the catalog of available dashboards, and a `purchases` join table mapping users to dashboards.
- **Google Sheets API**: The actual backend for the life trackers. Next.js fetches data from here on behalf of the logged-in user.

### Estimated Cost: $0/mo

---

## Phase 2: Automation & Performance

### Trigger to Transition
- You have 3+ successful dashboard sales.
- Manual payment approvals become a bottleneck.
- Google Sheets API limits are causing slow load times for users.

### Architecture Diagram

```mermaid
graph TB
    subgraph "Users"
        User[👤 Customer]
    end

    subgraph "Vercel Managed Infrastructure"
        Next[Next.js App Server]
        KV[(Redis Cache)]
    end

    subgraph "Third Party Services"
        Stripe[Stripe Webhooks]
        Google[Google Sheets API]
    end

    subgraph "Supabase"
        DB[(PostgreSQL Primary)]
    end

    User --> Next
    Next --> KV
    KV -->|Cache Miss| Google
    
    Stripe -->|Webhook Auto-Approve| Next
    Next -->|Update Access| DB
```

### New Components & Upgrades
- **Stripe Webhooks**: Replaces manual approval. When a user buys, a webhook automatically updates the `purchases` table in Supabase.
- **Redis Caching (Vercel KV)**: Google Sheets APIs rate limit strictly (100 reqs/100sec). We must cache API responses for 10-30 seconds to prevent the dashboard from crashing if the user refreshes rapidly.

### Security Measures
- Stripe Webhook signature verification to prevent spoofed purchases.
- SWR (Stale-While-Revalidate) caching strategy to hide Google API latency.

### Estimated Cost: $45/mo 
(Vercel Pro $20 + Supabase Pro $25 for higher database performance/backups).

---

## Phase 3: Scale & Multi-Tenancy

### Trigger to Transition
- Thousands of users. 
- You want to allow *other* creators to design and sell their own dashboards on your platform (converting from a first-party store to a marketplace).

### Architecture Diagram

```mermaid
graph TB
    subgraph "Global Edge Network"
        CDN[Vercel Edge Network]
        WAF[Web Application Firewall]
    end

    subgraph "Application Cluster"
        API[Next.js Serverless Functions]
        Workers[Background Job Queue]
    end

    subgraph "Data & Storage Layer"
        Primary[(Supabase Primary DB)]
        Replica[(Supabase Read Replica)]
        Cache[(Redis Enterprise)]
    end

    subgraph "External Integrations"
        Google[Google Sheets APIs]
        Stripe[Stripe Connect]
    end

    CDN --> WAF --> API
    API --> Cache
    API --> Primary
    API --> Replica
    API --> Stripe
    API -.->|Offload Sync| Workers
    Workers --> Google
```

### Scaling Strategy
- **Stripe Connect**: Instead of standard Checkout, use Connect to route payouts to third-party dashboard creators while taking a platform fee.
- **Background Sync Workers**: Instead of fetching from Google Sheets while the user waits for the page to load, background workers keep the cache warm.
- **Database Read Replicas**: Supabase read replicas handle the heavy load of listing the catalog and validating user permissions.

### Estimated Cost: $150+/mo

---

## Data Architecture

Because user tracking data lives in Google Sheets, our database schema is incredibly lightweight.

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
        string master_sheet_url "Link to the Google Sheet template"
        integer price_cents
        boolean is_active
    }

    purchases {
        uuid id PK
        uuid user_id FK
        uuid dashboard_id FK
        boolean is_approved "Manual gate for MVP"
        datetime purchased_at
    }

    users ||--o{ purchases : "makes"
    dashboards ||--o{ purchases : "bought in"
```

## IDE-Based AI Generation Flow

*Note: The AI generator is NOT hosted on the web app for security and simplicity. All dashboard generation happens locally in the creator's IDE.*

```mermaid
sequenceDiagram
    actor Admin
    participant IDE as Local Antigravity IDE
    participant Repo as Local Filesystem
    
    Admin->>IDE: Submit description/mockup for new dashboard
    IDE->>Repo: Write `src/app/dashboard/[new-slug]/page.tsx`
    IDE->>Repo: Create JSON schema doc for Google Sheet
    IDE-->>Admin: "Code generated. Please create the Master Google Sheet."
    
    Admin->>Admin: Manually create Master Sheet from schema
    Admin->>Repo: Commit code & push
```
