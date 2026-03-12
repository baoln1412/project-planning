# Reading Tracker

> **Parent Project**: [Dashboard Hub](../dashboard-hub/overview.md)  
> **Type**: First Premium Product  
> **Status**: 🟢 Ready for Development  
> **Created**: 2026-03-12

## Summary

Reading Tracker is the **first premium product** on the Dashboard Hub marketplace. It provides avid readers with a tactile digital bookshelf, reading progress tracking, session scheduling with a live timer, and gamified challenges with badges — all powered by the user's own Google Sheets as the database.

The app is designed for **Vietnamese readers first**, with Vietnamese book search integration planned for P1. It follows the dashboard-hub architecture: Google Sheets as source of truth, Supabase as performance cache, Next.js + shadcn/ui + Recharts on Vercel.

## Problem Statement

Readers want to track their reading habits, set goals, and feel rewarded for consistency — but existing solutions either lock data into proprietary apps (Goodreads, StoryGraph) or offer ugly spreadsheet templates. Reading Tracker provides a premium, gamified dashboard that connects to the user's own Google Sheet, so they own every byte of their data.

## Target Users

- **Primary**: Vietnamese avid readers who track reading habits and want a beautiful dashboard
- **Secondary**: Productivity/quantified-self enthusiasts who already use Google Sheets for personal tracking

## Discovery Answers

| # | Question | Answer |
|---|---|---|
| 🎯 | **North Star** | Be the first attractive premium product on Dashboard Hub — prove users will pay for a Google Sheets-powered dashboard |
| 🔌 | **Integrations** | Google Sheets API (user data), Supabase (cache), Dashboard Hub ecosystem (Next.js, shadcn/ui, Recharts) |
| 💾 | **Source of Truth** | User's Google Sheets — ALL data including books, reading logs, goals, challenges, and badges live in the user's Sheet |
| 📦 | **Delivery Payload** | A premium dashboard product within Dashboard Hub's storefront, generated via `/generate-dashboard` skill |
| 🚫 | **Behavioral Rules** | Write ALL data back to user's Sheet. Supabase cache is ephemeral only. Follow dashboard-hub `_design/brand/` guidelines |

## Google Sheets Data Model (Master Sheet Tabs)

The user clones a Master Sheet with these pre-configured tabs:

| Tab | Columns | Purpose |
|---|---|---|
| **Library** | Title, Author, Genre, Status (Want to Read / Reading / Finished / Dropped), Cover URL, Rating (1-5), Review, Date Added | Book collection |
| **Reading Log** | Date, Book Title, Start Time, End Time, Pages Read, Duration (min), Notes | Session history |
| **Goals** | Year, Month, Target Books, Target Hours, Actual Books, Actual Hours | Monthly/annual targets |
| **Challenges** | Name, Type (books/hours/streak), Target, Current Progress, Status (active/completed/expired), Start Date, End Date | Active challenges |
| **Badges** | Badge Name, Earned Date, Description, Icon Emoji | Gamification rewards |

> [!IMPORTANT]
> **$0 extra cost at 100 users.** Google Sheets API handles 300 req/min. With sheet_cache syncing every 15 min for 100 users = ~7 reads/min. Session write-backs are occasional (~20/day peak). All within free tier limits for Google API, Supabase, and Vercel.

## Key Features

### Milestone 1: My Library + Progress Dashboard (P0)

**My Library (Bookshelf View)**
- Visual grid displaying book covers on a stylized digital bookshelf
- Add books: manual entry (Title + Author), edit to add cover URL, genre, etc.
- Book status management: Want to Read → Reading → Finished → Dropped
- Book detail modal: edit metadata, add rating and review
- Responsive bookshelf layout (desktop grid, mobile list)

**Progress Dashboard**
- High-level stat cards: Total Books Finished, Current Reading Streak, Total Hours Read
- Goal progress: annual and monthly book/hour targets with progress bars (Recharts)
- Activity history: recent finished books and logged sessions
- Reading trends: books per month bar chart (Recharts)

### Milestone 2: Reading Schedule & Timer (P1)

**Reading Calendar**
- Monthly calendar view showing reading sessions and scheduled future sessions
- Activity list alongside calendar for detailed session info
- Scheduling tool: plan future reading sessions

**Live Reading Timer**
- "Start Reading" button with live timer (client-side state)
- Select which book you're reading from Library
- Auto-stop after idle timeout (configurable)
- On stop: writes session entry back to user's Google Sheet (Reading Log tab)
- Session summary: pages read, duration, notes

**Vietnamese Book Search** (P1 enhancement)
- Search endpoint to find book info from Vietnamese sources
- MVP approach: manual entry first, search as enhancement
- Sources to evaluate: Tiki.vn, Fahasa.com, Google Books API
- Auto-fills: Title, Author, Cover URL, Genre when user selects a search result

### Milestone 3: Challenges & Badges (P2)

**Challenges**
- Pre-configured challenges: "Read 3 books this month", "Read 10 hours this week", "7-day streak"
- Progress tracking against challenge targets
- Auto-complete detection: when a challenge target is met, mark as completed
- All challenge state stored in user's Google Sheet (Challenges tab)

**Badge Gallery**
- Badge milestones: First Book Finished, 10 Books Club, 30-Day Streak, Night Owl (late reading), etc.
- Earned badges with dates and descriptions
- Visual badge display with emoji icons
- Badge unlock triggers: calculated from Reading Log + Library data
- All badge state stored in user's Google Sheet (Badges tab)

## Architecture Fit

```
┌─────────────────────────────────────────────┐
│  Dashboard Hub (parent project)             │
│                                             │
│  /store                                     │
│    └── /product/reading-tracker  (SEO page) │
│                                             │
│  /dashboard                                 │
│    └── /dashboard/reading-tracker            │
│         ├── /library     (Bookshelf view)   │
│         ├── /progress    (Stats + charts)   │
│         ├── /schedule    (Calendar + timer)  │
│         └── /challenges  (Badges + goals)   │
│                                             │
│  Google Sheet (user's clone)                │
│    ├── Library tab                          │
│    ├── Reading Log tab                      │
│    ├── Goals tab                            │
│    ├── Challenges tab                       │
│    └── Badges tab                           │
│                                             │
│  Supabase (sheet_cache — ephemeral)         │
│    └── Cached copy of user's Sheet data     │
└─────────────────────────────────────────────┘
```

## Design Principles

- **Cohesive with Dashboard Hub**: Uses `_design/brand/` guidelines, shadcn/ui components, Recharts charts
- **Tactile Bookshelf UI**: Digital bookshelf feel — book covers on wooden shelves, hover effects, satisfying interactions
- **Gamification**: Streaks, badges, and challenges encourage consistent reading habits
- **Vietnamese-first**: UI supports Vietnamese text, book search targets Vietnamese book catalog
- **Data ownership**: Every piece of data lives in the user's Google Sheet — if they leave the app, they keep everything

## Development Workflow

This product will be generated using the Dashboard Hub's `/generate-dashboard` Antigravity skill:

1. Create idea folder: `_dashboards/reading-tracker/`
2. Add mockup descriptions, screen specs, and Sheet schema to the folder
3. Run `/generate-dashboard` skill → generates Next.js page routes + Master Sheet template
4. Review, customize, and polish generated code
5. Create Master Sheet on Google Drive, seed with sample data
6. Deploy and add to storefront catalog

## Vietnamese Book Search — Research Notes

**Potential Sources (to evaluate in P1):**

| Source | Type | Cover Art | VN Coverage | API? |
|---|---|---|---|---|
| Tiki.vn | E-commerce | ✅ High quality | ⭐⭐⭐⭐⭐ | No (scrape) |
| Fahasa.com | Bookstore | ✅ Good | ⭐⭐⭐⭐ | No (scrape) |
| Google Books API | API | ✅ Variable | ⭐⭐ (limited VN) | ✅ Free |
| Open Library | API | ⚠️ Some | ⭐ (minimal VN) | ✅ Free |

**MVP recommendation**: Start with manual entry (Option B: Title + Author, edit later to add cover URL). Add Tiki.vn or Google Books search as a P1 enhancement — it's a delighter, not a blocker.

## Open Questions

- [x] Pricing: **$9.99 one-time purchase**
- [ ] What specific badges/challenges should ship in M3? (Need a curated list)
- [ ] Timer UX: Should the timer work when the browser tab is in background? (Requires service worker)

## Created

2026-03-12
