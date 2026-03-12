# Tech Stack: Dashboard Hub

## Summary

The Dashboard Hub utilizes a **Next.js 15 (App Router)** full-stack architecture, optimized for rapid development and seamless Google Sheets integration. **Supabase (PostgreSQL)** handles user accounts, purchase records, and Sheet data caching. **Auth.js** manages the Google OAuth flow to request the narrow `drive.file` scope. The UI is built with **Tailwind CSS 4.x + shadcn/ui + Recharts**, providing premium, zero-dependency-risk dashboard components.

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | Essential for safely typing Google Sheets API responses and component props |
| Framework | Next.js | 15.x | App Router with Server Components protects API keys, ISR for SEO pages |
| Authentication | Auth.js (NextAuth) | 5.x | Best-in-class for Google OAuth. Handles `drive.file` scope token management |
| Database | Supabase (PostgreSQL) | — | User accounts, dashboard catalog, purchases, AND **Sheet data cache** |
| ORM | Drizzle | 0.3x | Lighter than Prisma, excellent TypeScript inference, Supabase-native |
| UI Framework | Tailwind CSS | 4.x | Latest version, utility-first, perfect for AI code generation |
| Component Library | shadcn/ui | latest | Copy-paste components (zero npm dependency), fully customizable, massive community |
| Charts | Recharts | 2.x | 20K+ GitHub stars, composable React charts, well-maintained |
| Payment Gateway | Stripe (Webhooks) | — | Auto-approve purchases via `checkout.session.completed` webhook. No manual gate |
| Hosting | Vercel | — | Zero-config Next.js deployment, free hobby tier, ISR for SEO |

## Detailed Evaluation

### UI / Component Library

| Criterion | shadcn/ui + Recharts | Tremor | Material UI |
|---|---|---|---|
| Vendor Lock-in Risk | ⭐⭐⭐⭐⭐ (none — copy-paste) | ⭐⭐⭐ (npm dependency) | ⭐⭐⭐ (heavy dep) |
| Customizability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Community Size | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Dashboard Focus | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| AI Generation Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Long-term Stability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.85** | **3.70** | **3.50** |

**Recommendation**: **shadcn/ui + Recharts** — Zero vendor lock-in (components are copy-pasted into your codebase, not installed as npm packages). If shadcn/ui stops maintenance, your code still works. Recharts has 20K+ GitHub stars with consistent maintenance. Combined with Tailwind 4.x, AI can generate dashboard layouts just as easily as with Tremor.

### Database & Auth (unchanged from v1)

| Criterion | Supabase (PostgreSQL) | Firebase (NoSQL) | Vercel Postgres |
|---|---|---|---|
| Relational Data | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Auth Integration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Caching Support | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Cost (Free Tier) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.80** | **4.10** | **4.00** |

**Recommendation**: **Supabase** — Now even stronger with the addition of `sheet_cache` table. Single database for accounts, purchases, AND cached Sheet data.

## Cost Estimate

| Service | Free Tier | Production Est. (100+ Sales/mo) |
|---|---|---|
| Hosting (Vercel) | $0/mo (Hobby) | $20/mo (Pro) |
| Database (Supabase) | $0/mo (up to 500MB, 50K MAU) | $25/mo (Pro) |
| Payments (Stripe) | No monthly fee, 2.9% + 30¢ per tx | 2.9% + 30¢ / tx |
| **Total Flat Cost** | **$0/mo** | **$45/mo** |

> [!TIP]
> The MVP still launches with **$0/month** in fixed costs. Stripe Webhooks auto-approval replaces the manual gate with zero additional infrastructure cost.

## Alternatives Considered

| Technology | Why Rejected |
|---|---|
| **Tremor** | Vendor lock-in risk — if Tremor is abandoned, all dashboard UIs break. shadcn/ui components are copy-pasted into your codebase (zero dependency). |
| **Prisma** | Slower cold starts on serverless compared to Drizzle. |
| **Manual Purchase Approval** | Anti-pattern that kills conversion. Stripe Webhooks provide the same security guarantee with instant access. |
| **Tailwind 3.x** | Replaced with 4.x for latest features and performance improvements. |
| **Redis / Vercel KV** | Unnecessary for MVP. Supabase `sheet_cache` table provides the same caching benefit without an additional service. |
