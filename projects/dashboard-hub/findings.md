# Findings: Dashboard Hub

> **Last Updated**: 2026-03-09  
> **Phase**: Research Complete

## Discovered Constraints

### Technical
- **Google OAuth Verification**: 
  - [!CAUTION] If the app requests broad access to a user's Google Drive, it triggers a mandatory, expensive CASA security assessment. 
  - **Decision:** We MUST limit the OAuth scope to `https://www.googleapis.com/auth/drive.file`. This restricts the app's access ONLY to sheets the user explicitly links to the app, avoiding the massive security audit.
- **API Rate Limits**: Google Sheets API strictly enforces 300 read requests per minute per project, and 100 requests per 100 seconds per user. The architecture MUST include aggressive caching (Redis or Next.js cache) to avoid hitting 429 errors.
- **Data Fragility**: Users control the source of truth. They will inevitably rename columns, delete rows, or break formulas. The frontend must have extremely defensive parsing and graceful error boundaries.

### Business
- The B2C market for "Life Trackers" is huge, but entirely fragmented between completely closed SaaS apps (Habitica) and locked ecosystem templates (Notion).
- There are no major competitors offering premium, standalone dashboards powered by personal Google Sheets. Existing tools (Glide, Softr) are focused heavily on B2B internal tooling and lack premium consumer aesthetics.

## Key Decisions Made
- **Socratic Refinement:** 
  1. The architecture will be built for Continuous AI Expansion, allowing the admin to easily drop in new AI-generated dashboard modules.
  2. Users will clone a Master Data Sheet, which acts as the database for both reading and writing from the dashboard.
  3. Payments will initially be manually approved to ensure transaction security and prevent data leaking.
- **Tech Stack:** Next.js 15 (App Router), Supabase (Auth + Accounts DB), Tailwind + Tremor (UI), Google Sheets API (Data), Auth.js (OAuth).
- **Architecture Flow:**
  - MVP relies on manual payment approval in Supabase and achieves a $0/mo fixed cost.
  - Vercel KV (Redis) caching is absolutely mandatory for Phase 2 to prevent strict Google API rate limit bans (100 req/100s).
  - The AI Generation loop will produce React code and commit it directly to GitHub via API to continuously expand the platform.

## Open Questions (Resolved)
1. **Authentication:** Resolved. We will use Google OAuth exclusively for user accounts and sign-in. This aligns perfectly with the need to request `drive.file` scope access to their sheets, eliminating the need for a separate email/password system.
2. **AI Generation Focus:** Resolved. The Antigravity AI generation pipeline will be responsible for creating **both** the Next.js React component UI code AND the Google Sheet schema/template (columns, structure/Apps Script if needed) that the user will clone. Essentially, the AI outputs a complete full-stack "Dashboard Applet".
