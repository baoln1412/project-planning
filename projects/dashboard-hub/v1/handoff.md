# Handoff: Dashboard Hub

> **Generated**: 2026-03-09  
> **Status**: 🟢 Ready for Development

---

## Project Overview

A web application marketplace where users can purchase premium life/habit tracker dashboards. The unique value proposition is that the dashboards connect directly to the user's own Google Sheets, which act as their personal database. The platform uses Next.js, Supabase, and Auth.js (requesting strict `drive.file` OAuth scope). The creator uses the Antigravity IDE locally to generate new dashboard code and Master Sheet schemas.

> See full details: [overview.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/overview.md)

---

## Repository Structure

```
dashboard-hub/
├── src/
│   ├── app/                          # Next.js App Router pages
│   │   ├── (auth)/                   # Login routes
│   │   ├── admin/                    # Admin portal for manual purchase approvals
│   │   ├── store/                    # Public storefront listing dashboards
│   │   ├── dashboard/                # Protected user dashboards
│   │   │   ├── page.tsx              # Index showing purchased dashboards
│   │   │   └── [slug]/               # Specific tracker views (e.g., reading-tracker)
│   │   ├── api/                      # Protected API handlers
│   │   │   └── webhooks/stripe/      # Stripe webhook handler (Phase 2)
│   │   ├── layout.tsx                # Root layout (header, Tailwind/Tremor global styles)
│   │   └── page.tsx                  # Home/Landing page
│   ├── components/                   # Reusable React/Tremor components
│   │   ├── DashboardCard.tsx
│   │   ├── MasterSheetConnectForm.tsx
│   │   └── charts/                   # Tremor wrapper components
│   ├── lib/
│   │   ├── db/                       # Drizzle ORM schema & client
│   │   ├── auth.ts                   # Auth.js configuration (Google Provider)
│   │   ├── google.ts                 # Google Sheets API client (using stored access token)
│   │   └── stripe.ts                 # Stripe server-side client
│   └── types/                        # Global TypeScript interfaces
├── _templates/                       # GUIDES: Local IDE AI Prompt Templates
│   ├── dashboard-prompt.md           # Instructions for IDE to generate new Next.js Dashboards
│   └── schema-generator.md           # Instructions for IDE to generate Master Sheets
├── public/                           # Static assets
├── supabase/
│   └── migrations/                   # Drizzle SQL migrations
├── package.json
├── tsconfig.json
├── next.config.ts
├── README.md                         
└── .env.local                        # Environment variables
```

---

## Tech Stack

| Layer | Technology | Version | npm Package |
|---|---|---|---|
| Language | TypeScript | 5.x | `typescript` |
| Framework | Next.js (App Router) | 15.x | `next`, `react`, `react-dom` |
| Auth | Auth.js (NextAuth) | 5.x | `next-auth` |
| CSS | Tailwind CSS | 3.x | `tailwindcss` |
| UI Framework| Tremor | 3.x | `@tremor/react` |
| Icons | Lucide React | latest | `lucide-react` |
| ORM | Drizzle | 0.3x | `drizzle-orm`, `drizzle-kit` |
| DB Driver | Postgres.js | latest | `postgres` |
| API | Google APIs | 14x | `googleapis` |
| Payments | Stripe Node | 16x | `stripe` |

> See full evaluation: [tech-stack.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/tech-stack.md)

---

## Data Contracts (Data-First Rule)

Because user data lives in Google Sheets, the database schema is very small, focusing solely on the catalog and access control.

### 1. Database Entities (Drizzle / Supabase)

**`users`**
Managed heavily by Auth.js, but we intercept the Google OAuth payload to extract the tokens.
```typescript
{
  id: uuid,             // PK
  email: string,
  name: string,
  image: string,
  created_at: timestamp
}
```

**`dashboards` (The Product Catalog)**
```typescript
{
  id: uuid,             // PK
  slug: string,         // e.g., 'reading-tracker'
  title: string,
  description: string,
  price_cents: integer,
  master_sheet_url: string, // URL to the public template users clone
  is_active: boolean
}
```

**`purchases` (The Access Control Join Table)**
```typescript
{
  id: uuid,             // PK
  user_id: uuid,        // FK to users.id
  dashboard_id: uuid,   // FK to dashboards.id
  is_approved: boolean, // Must be true for user to view the dashboard
  user_sheet_id: string,// The ID of the cloned sheet the user connected
  purchased_at: timestamp
}
```

### 2. Core API Responses

**`GET /api/store`** (Public: List available dashboards)
```json
[
  {
    "id": "uuid-123",
    "slug": "reading-tracker",
    "title": "Premium Reading Tracker",
    "price_cents": 1500,
    "description": "Track your books...",
    "master_sheet_url": "https://docs.google.com/..."
  }
]
```

**`GET /api/user/purchases`** (Protected: List what the user bought)
```json
[
  {
    "purchase_id": "uuid-999",
    "dashboard_id": "uuid-123",
    "dashboard_slug": "reading-tracker",
    "is_approved": true,
    "user_sheet_id": "1BxiMVs0XRYFgCE9..."
  }
]
```

### 3. Google Sheets Integration Rules
- **Token Handling:** The Next.js server MUST extract the Google Access Token from the Auth.js session (or refresh it if expired) to make calls to the Sheets API. This token should **never** be sent to the client browser.
- **Defensive Parsing:** You cannot trust the shape of the data returned by `googleapis` because the user can freely edit their spreadsheet. Every API route fetching Sheet data must wrap the logic in `try/catch` and use Zod (or manual validation) to ensure the row array matches the expected component schema before returning it to the React frontend.

---

## Environment Setup

### Setup Commands

```bash
# 1. Initialize Next.js
npx -y create-next-app@latest dashboard-hub --typescript --app --eslint --tailwind --no-src-dir --import-alias "@/*"
cd dashboard-hub

# 2. Install Core Dependencies
npm install next-auth@beta drizzle-orm postgres @tremor/react lucide-react googleapis stripe 

# 3. Install Dev Dependencies
npm install -D drizzle-kit 

# 4. Copy environment variables
cp .env.example .env.local

# 5. Run Database Migrations
npx drizzle-kit push
```

### `.env.example`

```env
# Next.js App URL
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Auth.js required secret (generate with `openssl rand -hex 32`)
AUTH_SECRET=

# Google OAuth Credentials (Ensure drive.file scope is configured in GCP Console)
AUTH_GOOGLE_ID=
AUTH_GOOGLE_SECRET=

# Supabase Postgres Connection String
DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[YOUR-PROJECT].supabase.co:5432/postgres

# Stripe Keys (Phase 2)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

---

## Architecture Reference

### Key Constraints & Decisions
- **Security Constraint (The Golden Rule):** We only request `https://www.googleapis.com/auth/drive.file`. This prevents triggering a $10,000+ Google CASA security audit. The app will *only* be able to read/write Google Sheets that the user explicitly creates via our app, or opens with our app via the Google Drive UI picker.
- **XSS Prevention:** ALL string data read from Google Sheets must be treated as untrusted user input. Never use `dangerouslySetInnerHTML`.
- **Admin AI Generation:** The Antigravity AI generation process does NOT happen on the web server. It is handled entirely locally in the admin's IDE (generating code + schemas) and pushed to git.

> See full architecture & ERD: [architecture.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/architecture.md)

---

## First Sprint / MVP Scope

Build **Milestones 1–3** first (Foundation, Storefront, Auth, Dashboard Engine). Phase 2 (Stripe Webhooks & Web Caching) can wait until you have actual paying users.

### MVP Acceptance Criteria
- [ ] Users can log in via Google OAuth.
- [ ] The app successfully requests and stores the `drive.file` access token.
- [ ] Users can view the `/store` page (data seeded in Supabase).
- [ ] Admin can manually toggle `is_approved` in Supabase for a user's purchase.
- [ ] Approved users can access their purchased dashboard page.
- [ ] The dashboard securely fetches their cloned Google Sheet data server-side and renders a Tremor chart without crashing.
- [ ] The `_templates/` directory exists with instructions for the IDE on how to generate new dashboards.

> See detailed milestones: [implementation-plan.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/implementation-plan.md)

---

## CI/CD Recommendations

| Stage | Tool | Purpose |
|---|---|---|
| Hosting | Vercel | Zero-config deployments for Next.js App Router |
| Database | Supabase | Managed Postgres; handle migrations via GitHub Actions or locally |
| Repo | GitHub | Push to `main` triggers Vercel deploy |

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "db:generate": "drizzle-kit generate",
    "db:push": "drizzle-kit push"
  }
}
```

---

## References

| Document | Path |
|---|---|
| Project Overview | [overview.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/overview.md) |
| Research | [research.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/research.md) |
| Persistent Findings | [findings.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/findings.md) |
| Tech Stack | [tech-stack.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/tech-stack.md) |
| Security Review | [security-review.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/security-review.md) |
| Architecture | [architecture.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/architecture.md) |
| Implementation Plan | [implementation-plan.md](file:///Users/lap15230/project-planning/projects/dashboard-hub/implementation-plan.md) |
