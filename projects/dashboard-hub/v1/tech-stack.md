# Tech Stack: Dashboard Hub

## Summary

The Dashboard Hub utilizes a **Next.js 15 (App Router)** full-stack architecture, optimized for rapid AI generation and seamless Google Sheets integration. **Supabase (PostgreSQL)** handles user accounts and purchase data securely, while **Auth.js** manages the complex Google OAuth flow required to request narrow `drive.file` access. The UI is built with **Tailwind CSS and Tremor**, providing beautiful, premium dashboard components out-of-the-box for free.

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | Essential for safely typing complex Google Sheets API responses and AI JSON outputs. |
| Framework | Next.js | 15.x | App Router provides secure server components (protecting API keys from clients) and powerful caching to mitigate Google API rate limits. |
| Authentication | Auth.js (NextAuth) | 5.x | Best-in-class for Google OAuth. Securely handles requesting and storing the specific Google Drive access tokens needed for Sheets read/write. |
| Database | Supabase (PostgreSQL) | — | Free tier is very generous. Stores the dashboard catalog, user accounts, and tracks which users have purchased which dashboards. |
| ORM | Drizzle | 0.3x | Lighter and faster than Prisma. Excellent TypeScript inference and seamless Supabase integration. |
| UI Framework | Tailwind CSS | 3.x | Industry standard, highly flexible, perfect for AI to generate UI code against. |
| Dashboard UI | Tremor | 3.x | Open-source (free) React library specifically built for dashboards. Provides charts, metric cards, and sleek layouts out-of-the-box. |
| Payment Gateway| Stripe | — | Industry standard for selling digital products. Will generate checkout links that we manually verify initially before fully automating. |
| AI Generation | Antigravity AI | — | Plugs into the admin portal to generate both the Next.js React component for the dashboard and the Google Apps Script/Schema for the master sheet. |
| Hosting | Vercel | — | Zero-config deployment for Next.js. The free tier will easily support the MVP launch. |

## Detailed Evaluation

### Database & Auth

| Criterion | Supabase (PostgreSQL) | Firebase (NoSQL) | Vercel Postgres |
|---|---|---|---|
| Relational Data | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Auth Integration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Cost (Free Tier) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| DX / Tooling | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.80** | **4.10** | **4.00** |

**Recommendation**: **Supabase** — We need relational data (User -> Purchases -> Dashboard Catalog). Supabase provides a powerful Postgres database, but also includes an excellent free tier and a robust Auth system that pairs perfectly with Auth.js for Google OAuth.

### UI / Styling

| Criterion | Tailwind + Tremor | Material UI (MUI) | Chakra UI |
|---|---|---|---|
| Dashboard Focus | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Aesthetics | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Customizability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| AI Generation Fit| ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.90** | **3.20** | **3.80** |

**Recommendation**: **Tailwind + Tremor** — Tremor is specifically designed for building analytics dashboards. It is free, looks incredibly premium, and uses Tailwind underneath. Because Tailwind is utility-first, it is exceptionally easy for the Antigravity AI to generate new dashboard layouts using it.

## Cost Estimate

| Service | Free Tier | Production Est. (100+ Sales/mo) |
|---|---|---|
| Hosting (Vercel) | $0/mo (Hobby) | $20/mo (Pro) |
| Database (Supabase)| $0/mo (up to 500MB, 50K MAU) | $25/mo (Pro) |
| Payments (Stripe) | No monthly fee, flat 2.9% + 30¢ per successful transaction | 2.9% + 30¢ / tx |
| **Total Flat Cost**| **$0/mo** | **$45/mo** |

> [!TIP]  
> The MVP can be launched and scaled to your first 50–100 customers with exactly **$0/month** in fixed infrastructure costs. You only pay Stripe when you make a sale. 

## Alternatives Considered

| Technology | Why Rejected |
|---|---|
| **Prisma (ORM)** | Slightly slower cold starts on serverless edge functions compared to Drizzle. Drizzle's zero-dependency nature fits Next.js 15 App Router perfectly. |
| **MongoDB** | We have distinct, relational entities (Users, Dashboards, Purchases). A relational database (PostgreSQL via Supabase) enforces data integrity better than document stores for e-commerce. |
| **Custom CSS/SCSS** | Too slow to write manually, and AI models struggle to write decoupled CSS files securely compared to inline Tailwind utility classes. |
