# Findings: Dashboard Hub

> **Last Updated**: 2026-03-11  
> **Phase**: v2 Optimization Complete

## Discovered Constraints

### Technical
- **Google OAuth Verification**: 
  - [!CAUTION] Broad Google Drive access triggers mandatory CASA security assessment.
  - **Decision:** We MUST limit OAuth scope to `https://www.googleapis.com/auth/drive.file`.
- **API Rate Limits**: Google Sheets API: 300 read reqs/min/project, 100 reqs/100sec/user.
  - **v2 Solution:** Supabase `sheet_cache` table eliminates direct API dependency on page loads. Background sync every 5-15 min.
- **Data Fragility**: Users WILL rename columns, delete rows, break formulas.
  - **v2 Solution:** `last_good_data` fallback in `sheet_cache`. Dashboard shows cached data with error warning instead of crashing.

### Business
- B2C "Life Tracker" market is fragmented between closed SaaS apps (Habitica) and locked ecosystem templates (Notion).
- No major competitors offering premium dashboards powered by personal Google Sheets.

## Key Decisions Made

### v1 Decisions (preserved)
- Architecture built for expandable dashboard modules
- Users clone Master Data Sheet as their personal database
- Google OAuth exclusively for user accounts (aligns with `drive.file` scope need)

### v2 Decisions (new)
| Decision | Rationale | Date |
|---|---|---|
| Replace Manual Purchase Approval with Stripe Webhooks | Manual approval kills conversion. Stripe Webhooks provide same security with instant access. | 2026-03-11 |
| Replace Tremor with shadcn/ui + Recharts | Zero vendor lock-in (copy-paste, not npm). Larger community. Same AI generation compatibility. | 2026-03-11 |
| Add free "Starter" dashboard (Habit Tracker) | Users need to try before buying. Free tier proves the value proposition. | 2026-03-11 |
| Add demo mode with sample data | Users can't decide to buy without seeing the dashboard in action. | 2026-03-11 |
| Add Supabase `sheet_cache` layer | Solves 3 problems in one: API rate limits, Sheet fragility, page load speed. | 2026-03-11 |
| Defer AI Generation to Milestone 5 | Premature optimization. Hand-craft first 2-3 dashboards to validate market before investing in AI pipeline. | 2026-03-11 |
| Update Tailwind 3.x → 4.x | Latest version for performance and feature improvements. | 2026-03-11 |
| Add SEO-optimized product pages | B2C products live and die by organic traffic. Each dashboard = landing page. | 2026-03-11 |
| One-click Sheet cloning via deep link | Reduces friction vs. "instructions" text. Google's `/spreadsheets/d/{ID}/copy` handles it natively. | 2026-03-11 |

## Open Questions (Resolved)
1. **Authentication:** ✅ Google OAuth with `drive.file` scope.
2. **AI Generation Focus:** ✅ Deferred to Phase 2. Hand-craft first dashboards.
3. **Payment Flow:** ✅ Stripe Webhooks auto-approval from Day 1.
4. **Caching Strategy:** ✅ Supabase `sheet_cache` with `last_good_data` fallback.
