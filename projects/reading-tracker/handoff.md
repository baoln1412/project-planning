# Handoff: Reading Tracker

> **Status**: 🟢 Ready for Development  
> **Parent**: [Dashboard Hub](../dashboard-hub/handoff.md)  
> **Price**: $9.99 one-time  
> **Last Updated**: 2026-03-12

---

## Project Overview

Reading Tracker is the **first premium product** on Dashboard Hub. It provides avid readers with a bookshelf view, progress analytics, reading timer, schedule calendar, and gamified challenges — all powered by the user's Google Sheet (5 tabs, 21+ columns). Designed for Vietnamese readers first.

**Theme**: Bright, warm, and inviting — light backgrounds with amber/brown accents.

> Full context: [overview.md](overview.md)

---

## Dependency

> [!IMPORTANT]
> Reading Tracker is a route within the Dashboard Hub Next.js app. **Dashboard Hub M1-M2 must be built first** (auth, Supabase, sheet_cache, storefront).

---

## Route Structure

```
/product/reading-tracker         → SEO product page ($9.99)
/dashboard/reading-tracker/
  ├── library/                   → Bookshelf grid (Server Component)
  ├── progress/                  → Analytics dashboard (Server Component)
  ├── schedule/                  → Calendar + live timer (Client Component)
  └── challenges/                → Badges + goals (Server Component)
```

---

## Tech Stack (Additions to Dashboard Hub)

| Layer | Technology | Why |
|---|---|---|
| Timer | Client-side `useState` + `useEffect` | Ephemeral state, no server needed |
| Calendar | shadcn/ui Calendar (react-day-picker) | Already in shadcn, Tailwind-styled |
| Gamification | Server-side calculation on sync | Streaks, badges computed and written back to Sheet |
| Book search (P1) | Tiki.vn scraping / Google Books API | Vietnamese book coverage |
| Design | Google Stitch MCP | Visual-first design workflow |

---

## Data Contracts

### Google Sheet Master Template (5 Tabs)

**Library Tab (21 columns):**

| Column | Type | Required | Notes |
|---|---|---|---|
| Cover URL | URL | — | Book cover image |
| Title | String | **Yes** | — |
| Author | String | **Yes** | — |
| Release Year | Number | — | — |
| Country | String | — | — |
| Publisher | String | — | — |
| Author Gender | String | — | — |
| Main Genre | String | — | For genre breakdown chart |
| Sub-Genre(s) | String | — | Comma-separated |
| Non-fiction Category | String | — | — |
| Price | Number | — | For spending analytics (VND) |
| Status | Enum | — | Read / To Read / Reading / DNF / On Hold |
| Bought? | Enum | — | Yes / No |
| Rating | 1-5 | — | Star rating |
| Date Added | Date | — | For spending per month |
| Start Date | Date | — | When started reading |
| Date Finished | Date | — | For "books this month", streaks |
| Total Pages | Number | — | For total pages stat |
| Current Page | Number | — | Progress % = Current / Total |
| Review/Notes | Text | — | Personal thoughts |
| Reread? | Enum | — | Yes / No |

**Reading Log Tab:** Date, Book Title, Start Time, End Time, Pages Read, Duration (min), Notes

**Goals Tab:** Year, Month, Target Books, Target Hours, Actual Books, Actual Hours

**Challenges Tab:** Name, Type, Target, Current Progress, Status, Start Date, End Date

**Badges Tab:** Badge Name, Earned Date, Description, Icon Emoji

### API Endpoints

```typescript
// POST /api/reading-tracker/add-book
// Input:
{ title: string, author: string, genre?: string, coverUrl?: string, status: "To Read" }
// Effect: Appends to Library tab → clears cache → re-fetches

// POST /api/reading-tracker/log-session
// Input:
{ bookTitle: string, startTime: ISO, endTime: ISO, pagesRead: number, durationMin: number, notes?: string }
// Effect: Appends to Reading Log tab → clears cache → re-fetches

// GET /api/reading-tracker/book-search?q=tô+hoài (P1)
// Output:
{ results: [{ title, author, coverUrl, genre, source: "tiki.vn" }] }
```

### Cache Invalidation

Every write-back follows: **Write to Sheet → DELETE cache → Re-fetch → INSERT fresh cache.**

---

## Analytics → Data Mapping

| Analytic | Formula |
|---|---|
| Books read this month | `COUNT WHERE Status="Read" AND Date Finished in month` |
| Total pages | `SUM(Total Pages) WHERE Status="Read"` |
| Total reading time | `SUM(Duration) FROM Reading Log` |
| Completion rate | `Read / (Read + DNF) × 100` |
| Genre breakdown | `GROUP BY Main Genre, COUNT DESC` |
| Reread/DNF per quarter | `COUNT Reread?="Yes" OR Status="DNF", by quarter` |
| Monthly spending | `SUM(Price) WHERE Bought?="Yes", by month` |

---

## Component Tree

```
src/app/dashboard/reading-tracker/
├── layout.tsx              → Sidebar + access check
├── library/
│   ├── page.tsx            → SSR: fetch cached Library
│   ├── bookshelf.tsx       → CSR: cover grid + hover effects
│   └── book-modal.tsx      → CSR: add/edit dialog
├── progress/
│   ├── page.tsx            → SSR: fetch all data
│   ├── stat-cards.tsx      → 6 stat cards
│   ├── charts.tsx          → Recharts: bar, donut, line, stacked
│   └── activity-log.tsx    → Recent books list
├── schedule/
│   ├── page.tsx
│   ├── calendar.tsx        → Monthly view + session markers
│   └── timer.tsx           → Live timer (client state)
└── challenges/
    ├── page.tsx
    ├── challenge-list.tsx   → Progress bars
    └── badge-gallery.tsx    → Earned badges grid
```

---

## Gamification

### 10 Badges

| Badge | Trigger | Emoji |
|---|---|---|
| First Book | Finish 1 book | 📖 |
| Bookworm | Finish 5 books | 📚 |
| 10 Books Club | Finish 10 books | 🏆 |
| Speed Reader | Finish book in < 3 days | ⚡ |
| Night Owl | Session after 22:00 | 🦉 |
| Early Bird | Session before 07:00 | 🐦 |
| Marathon | 3+ hours in one session | 🏃 |
| Streak 7 | 7-day reading streak | 🔥 |
| Streak 30 | 30-day reading streak | 💎 |
| Genre Explorer | 5+ different genres | 🌍 |

### 5 Challenges

| Challenge | Type | Target |
|---|---|---|
| Monthly Reader | books | Read 3 books this month |
| Weekly Hours | hours | Read 10 hours this week |
| Streak Builder | streak | Maintain 7-day streak |
| Genre Sampler | books | 2 books in new genre |
| Weekend Warrior | hours | 5 hours on weekends |

---

## Google Stitch Prompts (Bright Theme)

> ⚠️ All 4 prompts use: **bright warm theme — light cream/off-white background, warm amber and soft brown accents, clean typography (Inter), cozy bookstore feel.**

### Screen 1 — My Library

> Design a premium digital bookshelf dashboard for a reading tracker web app. Bright, warm theme — light cream/off-white background with warm amber and soft brown accents. Clean and cozy. Full-width header with "My Library" title and "+ Add Book" button (warm amber). Filter tabs: All, Reading, To Read, Read, DNF. Responsive grid of book cards (3-4 columns desktop). Each card: tall book cover image, bold title, muted author, colored status badge (Reading=blue, Read=green, To Read=amber, DNF=red), star rating. Hover: subtle lift/shadow. Left sidebar with icons: Library, Progress, Schedule, Challenges. Premium, modern, warm — like a cozy bookstore.

### Screen 2 — Progress Dashboard

> Analytics dashboard for reading tracker. Same bright warm theme, sidebar with Progress highlighted. Top row: 6 stat cards (white, subtle shadow) — Books Read 📚, Reading Streak 🔥, Total Pages 📄, Total Hours ⏱️, Completion Rate ✅%, Monthly Spending 💰. Charts 2x2 grid: 1) Bar "Books per Month" 2) Donut "Genre Breakdown" 3) Line "Spending Trend" 4) Stacked bar "Quarterly Read vs DNF vs Reread". Bottom: "Recent Activity" list with cover thumbnail, title, date finished, rating.

### Screen 3 — Reading Schedule

> Schedule page with timer. Bright warm theme. Left 60%: monthly calendar grid (white day cards, green dots=completed, amber=scheduled). Right 40%: "Start Reading" — large circular timer 00:00:00 on white card with amber border. Large amber Start button. Book dropdown, Pages/Notes fields on stop, "Log Session" button. Below calendar: "Upcoming Sessions" list. Timer should feel premium and satisfying.

### Screen 4 — Challenges & Badges

> Gamification page. Bright warm theme. Top: "Active Challenges" — horizontal white cards with progress bars (amber fill), current/target, days remaining. Completed: green check. Bottom: "Badge Gallery" — 4-5 column grid of badge cards. Each: large emoji, badge name, earned date. Unearned: gray + lock. Earned: warm gold border glow. Rewarding, warm feel.

---

## Milestones

| # | Name | Effort | Deliverable |
|---|---|---|---|
| M1 | Library + Progress | 2-3 weeks | **Product is sellable** after this |
| M2 | Calendar + Timer + VN Book Search | 2-3 weeks | Engagement features |
| M3 | Challenges + Badges | 1-2 weeks | Retention/gamification |

> Full task breakdown: [implementation-plan.md](implementation-plan.md)

---

## References

| Doc | Content |
|---|---|
| [overview.md](overview.md) | Features, pricing, scope |
| [tech-stack.md](tech-stack.md) | Stack additions |
| [architecture.md](architecture.md) | Data flow, timer, gamification, Stitch prompts |
| [implementation-plan.md](implementation-plan.md) | 3 milestones, 36 tasks, Gantt chart |
| [../dashboard-hub/handoff.md](../dashboard-hub/handoff.md) | Parent project handoff |
