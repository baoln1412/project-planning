# Research: Dashboard Hub

## Executive Summary
The market for digital wellness, life planning, and health tracking is experiencing explosive growth, projected to surpass $600 billion by 2026. While there are many generic "no-code" platforms that turn Google Sheets into web apps (like Glide or Softr), there is no dedicated, premium marketplace specifically for life/habit tracker dashboards powered by a user's own Google Sheet. The technical path is highly feasible using Next.js and the Google Sheets API, but the primary hurdle will be the rigorous Google OAuth Verification process required for a public SaaS application accessing user drives.

## Market Analysis
### Market Size & Trends
- **Digital Health & Wellness:** Expected to grow from ~$563B (2025) to $697B (2026), at a 23.7% CAGR.
- **Life Planning Apps:** A specialized subset valued around $342M, growing steadily.
- **Key Trends:** 
  - **AI-Powered Insights:** Users want proactive alerts and predictive analytics, not just static data.
  - **Data Ownership:** Growing privacy concerns push users toward platforms where they control the underlying data (like local files or personal Google Sheets).
  - **Premium UX/UI:** Clutter-free, beautiful data storytelling is replacing ugly spreadsheet templates.

### Target Demographics
- Productivity enthusiasts (the "Notion template" crowd).
- "Quantified Self" advocates who track habits, reading, fitness, and finances.
- Individuals who want privacy and full ownership of their data database (Google Sheets) but lack the design skills to make it look good.

## Competitor Analysis

### Direct Competitors
Unlike the massive market for "Notion Templates" (which lock users into Notion's ecosystem), there are very few direct platforms selling premium, standalone, read/write web dashboards connected to personal Google Sheets.

### Indirect Competitors (No-Code Sheet Builders)
These platforms turn Google Sheets into websites, but they target B2B internal tools rather than B2C life planners.

| Name | Pricing | Strengths | Weaknesses |
|---|---|---|---|
| **Glide** | High (mostly B2B) | Extremely fast to build mobile-first apps | Generic UI, expensive for consumers, focused on internal company tools. |
| **Softr** | Freemium | Great for B2B client portals | UI is blocky; not designed for beautiful, custom consumer dashboards. |
| **AppSheet** | Included w/ Google | Deep Google integration, powerful logic | Utilitarian UI, complex learning curve, B2B focused. |
| **SpreadSimple**| Freemium | E-commerce focus from Sheets | Not designed for charts, calendars, or habit tracking. |

### Substitutes
- **Notion / Obsidian Templates:** Highly popular, but users are locked into those specific note-taking ecosystems.
- **Dedicated SaaS (e.g., Habitica, Strides):** Beautiful UX, but the company owns the data. Hard to export or run custom queries.

## Technical Feasibility
- **APIs Available:** Google Sheets API v4 provides comprehensive read/write access.
- **Recommended Libraries:** `googleapis` (Node.js client), `next-auth` (for Google OAuth login), Tremor or Recharts (for beautiful dashboard components in React).
- **Architecture:** Next.js deployed on Vercel is ideal. The app will authenticate the user via Google OAuth, request access to their specific cloned Master Sheet, and fetch data server-side or via SWR on the client.

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| **Google OAuth Verification** | Legal / Tech | High | High | Accessing a user's Google Drive/Sheets requires sensitive scopes. We MUST use `https://www.googleapis.com/auth/drive.file` (only gives access to files created or explicitly opened by the app) to avoid the expensive CASA security assessment required for full Drive access. Requires a privacy policy, demo video, and domain verification. |
| **Google API Rate Limits** | Technical | High | Med | Google limits Sheets API to 300 requests/minute/project and 100 reqs/100sec/user. Mitigation: Implement robust caching (SWR/Redis) and exponential backoff for retries. Do not fetch on every render. |
| **User Modifying Master Sheet** | Usage | High | High | Because the user owns the Google Sheet, they might accidentally delete a column or break a format, crashing the dashboard. Mitigation: The dashboard must have extremely defensive parsing and elegant error states ("Missing column 'A' in Sheet"). |
| **Market Education** | Business | Med | Med | Users are used to buying SaaS or Notion templates. Buying a "Dashboard" and connecting a Sheet is a new flow. Mitigation: Excellent onboarding UX and simple 1-click clone links for the Master Sheets. |

## Key Insights
- The market opportunity is massive. Capturing even a fraction of the "Notion Template" market by offering standalone, data-private dashboards is highly viable.
- **The biggest technical hurdle is Google OAuth.** Do NOT request full Google Drive access. We must structure the app to only request access to the specific spreadsheet the user clones (`drive.file` scope).
- Defensive programming is paramount. The web app must handle messy, user-altered spreadsheet data gracefully without crashing.

## Sources
1. The Business Research Company: Digital Health Market
2. Fortune Business Insights: Health and Wellness Market
3. Softr, Glide, AppSheet product documentation
4. Google Cloud Documentation: OAuth Verification FAQ & Sheets API Quotas
