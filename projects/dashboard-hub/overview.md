# Dashboard Hub
*(Temporary Name)*

> **Status**: 🟢 Ready for Development  
> **Created**: 2026-03-09  
> **Last Updated**: 2026-03-11

## Summary

A web application marketplace where users can purchase premium life/habit tracker dashboards. The unique value proposition is that the dashboards connect directly to the user's own Google Sheets, which act as their personal database. Users own their data — the platform only reads/writes to Sheets they explicitly grant access to.

## Problem Statement

Users want beautiful, functional dashboards to track their lives (reading, habits, goals), but they don't want their personal tracking data locked into a proprietary system. Standard spreadsheet templates are ugly, while dedicated SaaS apps are rigid and own the data.

## Target Users

- **Consumers**: People who want premium tracking dashboards but prefer to own their data via Google Sheets.
- **Admin/Creator**: The workspace owner (you) — who needs to rapidly build and sell new dashboards.

## Discovery Answers

1. **🎯 North Star**: To generate passive income by selling premium life tracker dashboards to interested users.
2. **🔌 Integrations**: Google Sheets API (user data), Stripe Webhooks (auto-payments), Supabase (data cache + accounts).
3. **💾 Source of Truth**: User's Google Sheets (tracking data) + Supabase (accounts, purchases, sheet data cache).
4. **📦 Delivery Payload**: A Web Application.
5. **🚫 Behavioral Rules**: Do not store user tracking data permanently — cache for performance only, user's Sheet remains the source of truth.

## Key Features

### Core (MVP)
- **Free Starter Dashboard**: 1 free Habit Tracker dashboard to let users experience the value before paying
- **Demo Mode**: Preview any dashboard with pre-loaded sample data before connecting a real Sheet
- **One-Click Sheet Cloning**: Deep link (`/spreadsheets/d/{ID}/copy`) — one click to clone a Master Sheet template
- **Supabase Data Cache**: Sheet data cached in Supabase, served from cache, synced every 5-15 min — solves rate limits, fragility, and speed
- **Storefront & Product Pages**: SEO-optimized landing page per dashboard, browse available dashboards
- **Stripe Webhooks Auto-Approval**: Instant access after payment — no manual admin gate
- **Read & Write Integration**: Web app reads/writes to the user's connected Google Sheet via cached proxy

### Growth
- **SEO & Content Marketing**: Blog posts ("Best habit tracking methods"), each dashboard = its own landing page
- **Premium Dashboard Catalog**: Hand-crafted 2-3 premium dashboards (Reading Tracker, Fitness Tracker, etc.)
- **Onboarding Polish**: Step-by-step guided setup flow for connecting Sheets

### Phase 2 (Post-Revenue Validation)
- **Google Stitch MCP Integration**: Design dashboard screens visually in [Google Stitch](https://stitch.withgoogle.com) → extract "Design DNA" → export React → Antigravity adapts to shadcn/ui + Recharts. See `v2/CHANGELOG.md` for full workflow.
- **Continuous AI Generation**: Stitch designs + Antigravity code adaptation + Google Sheets MCP for template creation → fully automated dashboard pipeline
- **Dedicated Design System**: `_design/brand/` folder with Design DNA exports, brand assets, styling rules, and component summaries

## Version History

| Version | Date | Summary |
|---|---|---|
| Initial | 2026-03-09 | Project created and Socratic Refinement complete |
| v1 | 2026-03-10 | Added `_design/` system and AI generation prompts for brand consistency |
| v2 | 2026-03-11 | Feature optimization: replaced manual approval with Stripe Webhooks, replaced Tremor with shadcn/ui + Recharts, added free tier + demo mode + caching layer + SEO, deferred AI generation to Phase 2 |
| v2.1 | 2026-03-12 | Added Google Stitch MCP as visual design layer for Phase 2 AI dashboard generation workflow |
