# Tech Stack: Reading Tracker

> **Inherits from**: [Dashboard Hub tech-stack.md](../dashboard-hub/tech-stack.md)

## Summary

Reading Tracker inherits Dashboard Hub's full stack (Next.js 15, Tailwind 4.x, shadcn/ui, Recharts, Supabase, Drizzle, Auth.js). This document covers only the **additions specific to the Reading Tracker product**.

## Inherited Stack (from Dashboard Hub)

| Layer | Technology | Notes |
|---|---|---|
| Framework | Next.js 15 (App Router) | Shared app — Reading Tracker is a route under `/dashboard/reading-tracker` |
| Styling | Tailwind CSS 4.x | Shared theme |
| Components | shadcn/ui | Shared component library at `src/components/ui/` |
| Charts | Recharts 2.x | Composable charts for progress visualization |
| Database | Supabase (PostgreSQL) | Shared DB — `sheet_cache`, `purchases`, `dashboards` tables |
| ORM | Drizzle | Shared schema |
| Auth | Auth.js (Google OAuth) | Shared auth with `drive.file` scope |
| Payments | Stripe Webhooks | Shared purchase flow, $9.99 one-time |
| Hosting | Vercel | Shared deployment |

## Reading Tracker Additions

| Layer | Technology | Rationale |
|---|---|---|
| Timer | Client-side `useState` + `useEffect` | Live reading timer. No server dependency — timer state is ephemeral. On stop, writes session to Google Sheet. |
| Calendar | `react-day-picker` (shadcn/ui Calendar) | shadcn/ui includes a Calendar component built on react-day-picker. Free, accessible, Tailwind-styled. |
| Gamification | Server-side calculation (Next.js API route) | Streak/badge calculations run on sync, not client-side. Results written back to Sheet (Challenges/Badges tabs). |
| Book Covers | Manual entry (MVP) → Tiki.vn search (P1) | MVP: user adds cover URL. P1: search endpoint returns cover from Vietnamese book sites. |
| Design | Google Stitch MCP | Design screens in Stitch → extract Design DNA → export React → adapt for shadcn/ui. |

## Cost Impact

| Item | Cost | Notes |
|---|---|---|
| Reading Tracker product | $0 additional | Runs on Dashboard Hub's existing infrastructure |
| Google Sheets API (100 users) | $0 | ~7 reads/min + occasional writes — within free limits |
| Book search API (P1) | $0 | Tiki.vn scraping or Google Books API (free) |
| **Revenue per sale** | **$9.99** | One-time purchase — $9.69 net after Stripe (2.9% + 30¢) |
| **Breakeven** | **~5 sales** | Covers Vercel Pro + Supabase Pro ($45/mo) |
