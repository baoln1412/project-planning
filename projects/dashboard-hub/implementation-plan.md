# Implementation Plan: Dashboard Hub

This plan outlines the build process from project initialization to a functioning MVP with validated revenue, followed by AI-powered expansion.

## Milestone 1: Foundation & Authentication
**Goal:** Scaffold the app, set up the database, and implement Google OAuth for `drive.file` scope.
**Estimated Effort:** M/L

- [ ] Initialize Next.js 15 (App Router) with Tailwind CSS 4.x
- [ ] Install shadcn/ui components (Button, Card, Dialog, Table, Input, etc.)
- [ ] Set up the Supabase project and run Drizzle migrations for `users`, `dashboards`, `purchases`, `sheet_connections`, `sheet_cache` tables
- [ ] Configure Auth.js with Google Provider, requesting `drive.file` scope
- [ ] Create a secure server utility to retrieve Google Access Token from session
- [ ] Build the basic homepage / navigation shell
- [ ] Set up GitHub repo + CI (linting, type checks)

## Milestone 2: Free Dashboard + Sheets Integration + Caching
**Goal:** Prove the core value proposition — 1 free Habit Tracker dashboard that reads from a user's cloned Google Sheet via Supabase cache.
**Estimated Effort:** L (Core Technical Challenge)

- [ ] Create a Master Sheet template for the free Habit Tracker (specific column schema)
- [ ] Build one-click Sheet cloning flow (deep link: `/spreadsheets/d/{ID}/copy`)
- [ ] Build Sheet connection UI: user pastes their cloned Sheet URL → stored in `sheet_connections`
- [ ] Implement Google Sheets API server-side fetcher using user's OAuth token
- [ ] Build the caching layer: fetch Sheet data → store in `sheet_cache` as JSONB
- [ ] Implement background sync: refresh cache every 5-15 min (or on-demand "Refresh" button)
- [ ] Build the Habit Tracker dashboard page with Recharts + shadcn/ui components
- [ ] Build demo mode: show dashboard with pre-loaded sample data for non-connected users
- [ ] **Critical:** Implement defensive parsing + `last_good_data` fallback when Sheet is broken
- [ ] Add error boundaries with user-friendly messages ("Column 'A' missing in your Sheet")

## Milestone 3: Storefront + Premium Dashboards + Stripe
**Goal:** Launch the paid storefront with 2 premium dashboards and instant-access Stripe payments.
**Estimated Effort:** M

- [ ] Build `/store` page listing all dashboards (free and premium)
- [ ] Build `/product/[slug]` SEO-optimized landing pages with meta tags, Open Graph, structured data
- [ ] Hand-craft 2 premium dashboards (e.g., Reading Tracker, Fitness Tracker) with dedicated Sheet templates
- [ ] Integrate Stripe Checkout: "Buy" button → Stripe payment link
- [ ] Build Stripe Webhook endpoint (`/api/webhooks/stripe`):
  - [ ] Verify webhook signature
  - [ ] Listen for `checkout.session.completed`
  - [ ] Auto-set `is_approved = true` in `purchases` table
  - [ ] Redirect user to their new dashboard (instant access)
- [ ] Build `/dashboard` index page showing user's purchased + free dashboards
- [ ] Seed `dashboards` table with all 3 products (1 free + 2 premium)

## Milestone 4: SEO + Onboarding Polish
**Goal:** Drive organic traffic and reduce friction for new users.
**Estimated Effort:** S/M

- [ ] Create a blog layout + 2-3 SEO posts ("Best habit tracking methods 2026", "Why you should own your data")
- [ ] Add sitemap.xml and robots.txt
- [ ] Build step-by-step onboarding flow:
  1. Sign in with Google
  2. Clone the Master Sheet (one click)
  3. Paste your Sheet URL
  4. See your data in the dashboard
- [ ] Add loading states, empty states, and responsive design polish
- [ ] Set up Vercel Analytics for traffic monitoring
- [ ] Performance optimization: Server Components, dynamic imports, image optimization

## Milestone 5: AI Dashboard Generation (Post-Revenue Validation)
**Goal:** After proving the market with hand-crafted dashboards, enable AI-powered expansion.
**Estimated Effort:** S

- [ ] Create `_design/brand/` directory for global assets (logos, colors, theme variables, UI preferences)
- [ ] Create `_dashboards/` directory with subfolders for each dashboard idea (mockups, descriptions)
- [ ] Build Antigravity Skill (`.agents/skills/generate-dashboard/SKILL.md`) that:
  1. Reads brand guidelines from `_design/brand/`
  2. Reads specific dashboard idea from `_dashboards/[target]/`
  3. Researches and implements shadcn/ui + Recharts components
  4. Writes Next.js page route + Google Sheet schema
- [ ] Document the workflow in README.md

---

## Key Differences from v1

| Area | v1 (Previous) | v2 (Current) |
|---|---|---|
| Purchase flow | Manual admin approval | Stripe Webhooks auto-approval |
| UI library | Tremor | shadcn/ui + Recharts |
| Tailwind | 3.x | 4.x |
| Free tier | None | 1 free Habit Tracker dashboard |
| Demo mode | None | Preview with sample data |
| Sheet data | Direct API calls every page load | Supabase `sheet_cache` with background sync |
| Sheet cloning | "Instructions" text | One-click deep link |
| SEO | None | SEO-optimized product pages + blog |
| AI Generation | Milestone 4 (MVP) | Milestone 5 (post-revenue validation) |

## Next Steps
Run `/export-project` to compile these documents into a final `handoff.md` for the development workspace.
