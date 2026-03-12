# Handoff: Dashboard Hub

> **Status**: 🟢 Ready for Development  
> **Last Updated**: 2026-03-12 (v2.1)

---

## Project Overview

A marketplace for premium life/habit tracker dashboards powered by users' own Google Sheets. Users buy dashboards, clone a Master Sheet, connect it, and get a beautiful web dashboard. Data caching in Supabase eliminates Google API rate limits. Cache is **cleared and refreshed on every write-back**.

> See full details: [overview.md](overview.md)

---

## Repository Structure

```
dashboard-hub/
├── src/
│   ├── app/                          # Next.js 15 App Router
│   │   ├── (marketing)/              # Public pages
│   │   │   ├── page.tsx              # Homepage / storefront
│   │   │   └── product/[slug]/       # SEO product pages (ISR)
│   │   ├── (auth)/                   # Auth routes
│   │   ├── dashboard/                # Protected dashboard area
│   │   │   ├── page.tsx              # User's purchased dashboards
│   │   │   └── reading-tracker/      # First premium product
│   │   │       ├── library/
│   │   │       ├── progress/
│   │   │       ├── schedule/
│   │   │       └── challenges/
│   │   └── api/
│   │       ├── webhooks/stripe/
│   │       ├── sheets/               # Sync + write-back routes
│   │       └── reading-tracker/      # Product-specific API
│   ├── components/
│   │   ├── ui/                       # shadcn/ui (copy-paste, not npm)
│   │   ├── dashboard/                # Shared dashboard components
│   │   └── marketing/                # Storefront components
│   ├── lib/
│   │   ├── supabase.ts
│   │   ├── sheets.ts                 # Google Sheets API + write-back
│   │   ├── cache.ts                  # sheet_cache CRUD + invalidation
│   │   ├── stripe.ts
│   │   └── auth.ts                   # Auth.js (Google OAuth, drive.file scope)
│   └── db/
│       ├── schema.ts                 # Drizzle schema
│       └── migrations/
├── _design/
│   └── brand/                        # Design DNA exports from Stitch
├── _dashboards/
│   └── reading-tracker/              # Stitch exports + idea files
├── public/
├── .env.local
└── package.json
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 15.x |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | 4.x |
| UI Components | shadcn/ui | latest |
| Charts | Recharts | 2.x |
| Calendar | react-day-picker (via shadcn) | latest |
| Database | Supabase (PostgreSQL) | — |
| ORM | Drizzle | latest |
| Auth | Auth.js | 5.x |
| Payments | Stripe | latest |
| Design | Google Stitch MCP | `@_davideast/stitch-mcp` |
| Hosting | Vercel | — |

> Full evaluation: [tech-stack.md](tech-stack.md)

---

## Data Contracts

### Supabase Tables (Confirmed)

```sql
create table public.profiles (
  id text not null primary key,
  email text not null unique,
  role text not null default 'user',
  created_at timestamp not null default now()
);

create table public.dashboards (
  id text not null primary key,
  slug text not null unique,
  title text not null,
  description text not null,
  price_cents integer not null default 0,
  master_sheet_url text not null,
  is_active boolean not null default true,
  created_at timestamp not null default now(),
  discount_cents integer not null default 0,
  thumbnail_url text null,
  demo_images text null,
  demo_video_url text null
);

create table public.purchases (
  id text not null primary key,
  user_id text not null references profiles(id),
  dashboard_id text not null references dashboards(id),
  is_approved boolean not null default false,
  purchased_at timestamp not null default now(),
  user_sheet_url text null,
  payment_submitted boolean not null default false
);

-- Cache layer: cleared and refreshed on every write-back
create table public.sheet_cache (
  id text not null primary key,
  purchase_id text not null references purchases(id),
  tab_name text not null,
  data jsonb not null default '[]'::jsonb,
  last_good_data jsonb null,
  cached_at timestamp not null default now(),
  constraint sheet_cache_unique unique (purchase_id, tab_name)
);
```

### Core API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/sheets/sync?purchaseId=` | Sync all tabs from Sheet → cache |
| POST | `/api/sheets/write-back` | Write to Sheet → clear cache → re-fetch |
| POST | `/api/reading-tracker/add-book` | Add book to Library tab |
| POST | `/api/reading-tracker/log-session` | Log timer session to Reading Log |
| GET | `/api/reading-tracker/book-search?q=` | Search Vietnamese books (P1) |

### Cache Invalidation Rule

```
Write to Google Sheet → DELETE sheet_cache row → Re-fetch fresh data → INSERT
```

### Google Sheets Security Rules

- Only `drive.file` scope (avoids $10K+ CASA audit)
- Google token NEVER sent to client browser
- All Sheet data treated as untrusted input (defensive parsing)

---

## Environment Variables

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-side service role key |
| `AUTH_SECRET` | Auth.js session encryption |
| `AUTH_GOOGLE_ID` | Google OAuth client ID |
| `AUTH_GOOGLE_SECRET` | Google OAuth client secret |
| `STRIPE_SECRET_KEY` | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe publishable key |

### Setup

```bash
npx -y create-next-app@latest ./ --typescript --app --eslint --tailwind
npx shadcn@latest init
npx shadcn@latest add button card dialog table input tabs avatar badge calendar
npm install recharts drizzle-orm postgres googleapis stripe next-auth@beta
npm install -D drizzle-kit
```

---

## Architecture Reference

Key decisions:
- Google Sheets = source of truth (user owns data)
- Supabase = ephemeral cache (cleared on write-back)
- `user_sheet_url` on purchases (no separate sheet_connections table)
- `payment_submitted` + `is_approved` = manual flow for MVP
- Google Stitch MCP = visual design layer for dashboard generation

> Full diagrams: [architecture.md](architecture.md)

---

## First Sprint

**M1**: Foundation + Auth → **M2**: Free Dashboard + Sheets + Caching

Acceptance criteria:
- [ ] Google OAuth login with `drive.file` scope
- [ ] Supabase schema migrated (4 tables)
- [ ] 1 free dashboard: clone Sheet → connect → see data → cache works
- [ ] Cache invalidation on write-back functioning
- [ ] Demo mode with sample data

> Full milestones: [implementation-plan.md](implementation-plan.md)

---

## CI/CD

```
GitHub Push → Vercel Auto-Deploy (preview on PR, production on main)
Local: tsc --noEmit → eslint . → next build
```

---

## References

| Doc | Content |
|---|---|
| [overview.md](overview.md) | Features, version history |
| [tech-stack.md](tech-stack.md) | Technology evaluations |
| [architecture.md](architecture.md) | 3-phase architecture, Mermaid diagrams |
| [implementation-plan.md](implementation-plan.md) | 5 milestones, task breakdown |
| [findings.md](findings.md) | Decisions and constraints |
| [research.md](research.md) | Market analysis |
| [v2/CHANGELOG.md](v2/CHANGELOG.md) | shadcn/ui + Stitch MCP guides |
