# Handoff: Market Insights Platform

> This document contains everything needed to start development in a separate workspace. An AI coding agent should be able to read this file and begin building the MVP immediately.

## Project Overview

Build an **internal web application** for a team of 5 market analysts that combines Sensor Tower mobile app data (already in PostgreSQL), community sentiment data (via Apify scraping of App Store reviews), and Vietnamese gaming regulation data (scraped from abei.gov.vn) into a unified platform. Analysts can generate market insight reports using Claude API, edit them in a collaborative rich text editor, and publish finalized reports. The goal is to reduce report generation from days to under 30 minutes.

**Planning Documents**: See the full project folder at `projects/market-insights-platform/`

---

## Repository Structure

```
market-insights-platform/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/
│   │   │   ├── page.tsx              # Main dashboard
│   │   │   ├── apps/page.tsx         # App rankings browser
│   │   │   ├── community/page.tsx    # Community sentiment view
│   │   │   ├── regulations/page.tsx  # Regulation tracker
│   │   │   ├── reports/
│   │   │   │   ├── page.tsx          # Report list
│   │   │   │   ├── new/page.tsx      # New report wizard
│   │   │   │   └── [id]/page.tsx     # Report editor (Tiptap)
│   │   │   └── settings/page.tsx     # Scrape config, job history
│   │   ├── api/
│   │   │   ├── ai/chat/route.ts      # Streaming AI chat
│   │   │   ├── reports/
│   │   │   │   ├── route.ts          # List + Create
│   │   │   │   └── [id]/
│   │   │   │       ├── route.ts      # Get + Update
│   │   │   │       └── generate/route.ts  # LLM report generation
│   │   │   ├── dashboard/route.ts
│   │   │   ├── apps/
│   │   │   │   ├── route.ts
│   │   │   │   └── [id]/metrics/route.ts
│   │   │   ├── community/route.ts
│   │   │   ├── regulations/route.ts
│   │   │   ├── scrape/
│   │   │   │   ├── trigger/route.ts
│   │   │   │   └── jobs/route.ts
│   │   │   └── inngest/route.ts      # Inngest webhook
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/                       # shadcn/ui components
│   │   ├── dashboard/                # KPI cards, trend charts
│   │   ├── editor/                   # Tiptap editor + AI extensions
│   │   ├── reports/                  # Report list, wizard, status
│   │   └── layout/                   # Sidebar, header, nav
│   ├── lib/
│   │   ├── db/
│   │   │   ├── schema.ts            # Drizzle schema
│   │   │   ├── index.ts             # DB connection
│   │   │   └── migrations/
│   │   ├── ai/
│   │   │   ├── prompts/             # Report generation prompts
│   │   │   │   ├── market-overview.ts
│   │   │   │   └── competitive-brief.ts
│   │   │   ├── router.ts            # Task complexity classifier
│   │   │   └── providers.ts         # LLM provider config
│   │   ├── scraping/
│   │   │   ├── apify.ts             # Apify SDK wrapper
│   │   │   ├── regulation.ts        # abei.gov.vn scraper
│   │   │   └── sentiment.ts         # Sentiment classification
│   │   ├── inngest/
│   │   │   ├── client.ts
│   │   │   └── functions.ts         # Scrape + generation jobs
│   │   └── auth.ts                  # NextAuth config
│   └── types/
│       └── index.ts
├── drizzle.config.ts
├── next.config.ts
├── package.json
├── tsconfig.json
├── .env.example
└── .gitignore
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Language | TypeScript | 5.x |
| Framework | Next.js (App Router) | 15.x |
| UI Components | shadcn/ui | latest |
| Rich Text Editor | Tiptap | 2.x |
| Charts | Recharts | 2.x |
| LLM SDK | Vercel AI SDK | 6.x |
| Database | PostgreSQL | 15+ |
| ORM | Drizzle | 0.3x |
| Scraping | Apify SDK | 3.x |
| Auth | NextAuth.js (Auth.js) | 5.x |
| Background Jobs | Inngest | latest |
| Styling | Tailwind CSS | 4.x |
| Hosting | Vercel | — |

See [tech-stack.md](./tech-stack.md) for full evaluation matrices and rationale.

---

## Data Contracts

> ⛔ These schemas were confirmed by the user via the Data-First Rule before handoff.

### Database Schema (Drizzle)

```typescript
// Existing tables (introspect from Postgres, do NOT create migrations for these)
// sensor_tower_apps: { id, app_id, app_name, publisher, market, platform, category, last_updated }
// sensor_tower_metrics: { id, app_id, period, downloads, revenue, rank, rating, review_count, ingested_at }

// NEW tables — create via Drizzle migrations

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  email: varchar("email", { length: 255 }).notNull().unique(),
  name: varchar("name", { length: 255 }).notNull(),
  role: varchar("role", { length: 20 }).notNull().default("analyst"), // "analyst" | "lead"
  createdAt: timestamp("created_at").defaultNow(),
});

export const communityPosts = pgTable("community_posts", {
  id: uuid("id").primaryKey().defaultRandom(),
  source: varchar("source", { length: 50 }).notNull(), // "reddit" | "discord" | "facebook" | "appstore"
  sourceId: varchar("source_id", { length: 255 }),
  gameRef: varchar("game_ref", { length: 255 }),  // nullable reference to app
  author: varchar("author", { length: 255 }),
  content: text("content").notNull(),
  sentimentScore: real("sentiment_score"),           // 0.0-1.0, LLM-computed
  sentimentLabel: varchar("sentiment_label", { length: 20 }), // "positive" | "neutral" | "negative"
  metadata: jsonb("metadata"),                       // { subreddit, channel, rating, etc. }
  postedAt: timestamp("posted_at"),
  scrapedAt: timestamp("scraped_at").defaultNow(),
});

export const regulationEntries = pgTable("regulation_entries", {
  id: uuid("id").primaryKey().defaultRandom(),
  gameName: varchar("game_name", { length: 500 }).notNull(),
  licenseCategory: varchar("license_category", { length: 10 }).notNull(), // "G1" | "G2" | "G3" | "G4"
  publisher: varchar("publisher", { length: 500 }),
  decisionNumber: varchar("decision_number", { length: 100 }),
  approvalDate: date("approval_date"),
  expiryDate: date("expiry_date"),
  status: varchar("status", { length: 20 }).default("approved"), // "approved" | "expired" | "revoked"
  notes: text("notes"),
  scrapedAt: timestamp("scraped_at").defaultNow(),
});

export const reports = pgTable("reports", {
  id: uuid("id").primaryKey().defaultRandom(),
  authorId: uuid("author_id").references(() => users.id),
  title: varchar("title", { length: 500 }).notNull(),
  templateType: varchar("template_type", { length: 50 }).notNull(), // "market_overview" | "competitive_brief"
  status: varchar("status", { length: 20 }).default("draft"),      // "draft" | "review" | "published"
  tiptapContent: jsonb("tiptap_content"),                           // Tiptap JSON document
  dataSnapshot: jsonb("data_snapshot"),                             // Frozen data used for generation
  llmMetadata: jsonb("llm_metadata"),                               // { totalCost, totalTokens, models }
  createdAt: timestamp("created_at").defaultNow(),
  updatedAt: timestamp("updated_at").defaultNow(),
  publishedAt: timestamp("published_at"),
});

export const reportSections = pgTable("report_sections", {
  id: uuid("id").primaryKey().defaultRandom(),
  reportId: uuid("report_id").references(() => reports.id),
  sectionType: varchar("section_type", { length: 50 }).notNull(), // "summary" | "market_data" | "sentiment" | "regulation" | "recommendations"
  sortOrder: integer("sort_order").notNull(),
  generatedContent: text("generated_content"),   // LLM output
  editedContent: text("edited_content"),         // Human-edited version
  llmModel: varchar("llm_model", { length: 100 }),
  inputTokens: integer("input_tokens"),
  outputTokens: integer("output_tokens"),
  costUsd: real("cost_usd"),
  generatedAt: timestamp("generated_at").defaultNow(),
});

export const scrapeJobs = pgTable("scrape_jobs", {
  id: uuid("id").primaryKey().defaultRandom(),
  source: varchar("source", { length: 50 }).notNull(), // "apify_appstore" | "abei_scraper"
  status: varchar("status", { length: 20 }).default("pending"), // "pending" | "running" | "completed" | "failed"
  config: jsonb("config"),                              // actor params, URLs, etc.
  itemsScraped: integer("items_scraped"),
  errorMessage: text("error_message"),
  startedAt: timestamp("started_at"),
  completedAt: timestamp("completed_at"),
});
```

### Key API Contracts

**POST `/api/reports/[id]/generate`** — Trigger LLM report generation
```typescript
// Request
interface GenerateRequest {
  templateType: "market_overview" | "competitive_brief";
  params: {
    market: "VN" | "SEA" | "Global";
    period: string;              // "2026-03"
    appIds?: number[];           // specific apps to focus on
    includeRegulation?: boolean;
    includeSentiment?: boolean;
  };
}

// Response — streamed via Vercel AI SDK
// Each section streams progressively to the Tiptap editor
interface GenerateResponse {
  sections: Array<{
    type: "summary" | "market_data" | "sentiment" | "regulation" | "recommendations";
    content: string;             // markdown
    model: string;
    tokens: { input: number; output: number };
    cost: number;
  }>;
  totalCost: number;
  dataSnapshot: Record<string, unknown>;
}
```

**GET `/api/dashboard`** — Dashboard KPIs
```typescript
interface DashboardResponse {
  topMovers: Array<{ appName: string; downloads: number; changePct: number }>;
  revenueLeaders: Array<{ appName: string; revenue: number; changePct: number }>;
  sentimentSummary: {
    positive: number; neutral: number; negative: number;
    trendingTopics: string[];
  };
  recentRegulations: Array<{ gameName: string; category: string; approvalDate: string }>;
  reportActivity: { drafts: number; published: number; thisWeek: number };
}
```

---

## Environment Setup

### Prerequisites

- Node.js 20+ and npm/pnpm
- Access to existing PostgreSQL database with Sensor Tower data
- Anthropic API key (Claude)
- Apify API token
- Vercel account for deployment

### Setup Commands

```bash
# Initialize project
npx -y create-next-app@latest ./ --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --turbopack

# Install core dependencies
npm install drizzle-orm postgres @auth/core next-auth@beta
npm install @ai-sdk/anthropic ai
npm install @tiptap/react @tiptap/starter-kit @tiptap/extension-placeholder
npm install recharts apify-client inngest
npm install -D drizzle-kit @types/node

# Install shadcn/ui
npx shadcn@latest init
npx shadcn@latest add sidebar button card table dialog tabs input textarea badge avatar dropdown-menu skeleton toast sheet select

# Configure environment
cp .env.example .env.local
# Edit .env.local with your API keys

# Run Drizzle introspection for existing tables
npx drizzle-kit introspect

# Run migrations for new tables
npx drizzle-kit push

# Start dev server
npm run dev
```

### Environment Variables

| Variable | Description | Required |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string (existing DB with Sensor Tower data) | Yes |
| `NEXTAUTH_SECRET` | Random secret for NextAuth session encryption | Yes |
| `NEXTAUTH_URL` | App URL (http://localhost:3000 for dev) | Yes |
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude | Yes |
| `APIFY_API_TOKEN` | Apify platform API token | Yes |
| `INNGEST_EVENT_KEY` | Inngest event key for background jobs | Yes |
| `INNGEST_SIGNING_KEY` | Inngest signing key for webhook verification | Yes |

### External Services

| Service | Purpose | Setup Link |
|---|---|---|
| Anthropic | Claude API for report generation and sentiment analysis | https://console.anthropic.com |
| Apify | App Store review scraping (pre-built actor) | https://console.apify.com |
| Inngest | Background job scheduling (scraping, generation) | https://app.inngest.com |
| Vercel | Hosting and deployment | https://vercel.com |

---

## Architecture Reference

See [architecture.md](./architecture.md) for full Mermaid diagrams and ERD.

**Key Architectural Decisions**:
- **Monolithic Next.js app** — API routes co-located with frontend for simpler deployment; no microservices needed at 5-user scale
- **Tiptap for editor** — ProseMirror-based with AI Agent toolkit for streaming LLM content directly into the editor
- **Vercel AI SDK v6** — Streaming-first with `useChat` hooks, eliminates need for LangChain at MVP scale
- **Drizzle ORM** — Introspects existing Sensor Tower tables without migration conflicts; lightweight serverless cold starts
- **Inngest for background jobs** — Serverless-native step functions handle long-running scraping and report generation (minutes); Vercel Cron limited to 10s
- **Data snapshots** — Each report stores a frozen copy of the data used for generation, ensuring reproducibility
- **Section-level cost tracking** — Every LLM call logs model, tokens, and USD cost for monitoring

**Data Flow (Report Generation)**:
1. Analyst selects template + parameters (market, period, apps)
2. System queries Sensor Tower metrics + community posts + regulation entries
3. Data assembled into structured snapshot JSON (frozen with report)
4. Template prompt + data sent to Claude via Vercel AI SDK
5. Response streams section-by-section into Tiptap editor
6. Analyst reviews, edits, publishes — all changes saved to `reports.tiptap_content`

---

## First Sprint / MVP Scope

### Goals

Validate the end-to-end workflow: an analyst can generate a market insight report that combines Sensor Tower data with community reviews and regulation info, edit it in a rich text editor, and publish it — all in under 30 minutes.

### Features to Build (6 Milestones, 6 Weeks)

**Milestone 1 — Foundation & Auth (Week 1)**
- [ ] Next.js 15 + TypeScript + Tailwind 4 + shadcn/ui setup
- [ ] Drizzle ORM connected to existing Postgres + Sensor Tower table introspection
- [ ] New table migrations (users, reports, report_sections, community_posts, regulation_entries, scrape_jobs)
- [ ] NextAuth email/password for 5 users
- [ ] App shell with sidebar navigation

**Milestone 2 — Data Layer & Dashboard (Week 2)**
- [ ] Dashboard page with KPI cards (top movers, revenue leaders, report activity)
- [ ] Recharts trend charts (downloads, revenue over time)
- [ ] Apps browser page (paginated, searchable, filterable)
- [ ] App detail page with metrics charts

**Milestone 3 — Scraping Pipeline (Week 3)**
- [ ] Apify SDK integration + App Store Reviews actor configuration
- [ ] Custom abei.gov.vn regulation scraper (cheerio + fetch)
- [ ] Inngest scheduled jobs (weekly App Store, monthly regulations)
- [ ] Community page with posts and sentiment labels
- [ ] Regulations page with G1-G4 filters
- [ ] Settings page with manual scrape trigger + job history

**Milestone 4 — LLM Integration (Week 4)**
- [ ] Vercel AI SDK v6 + Anthropic provider setup
- [ ] Report generation prompts for 2 templates (Market Overview, Competitive Brief)
- [ ] Data fetcher assembling Sensor Tower + community + regulation data into snapshot
- [ ] Streaming report generation endpoint
- [ ] Community post sentiment classification via Claude
- [ ] LLM cost tracking (model, tokens, USD per section)

**Milestone 5 — Report Editor (Week 5)**
- [ ] Tiptap editor with StarterKit, Placeholder, and custom AI content insertion extension
- [ ] "Generate Report" button streaming sections into editor
- [ ] New Report wizard with template selection and market/period parameters
- [ ] Report list page with status badges (draft/review/published)
- [ ] In-editor AI chat panel for follow-up questions
- [ ] Report export (Markdown, HTML)

**Milestone 6 — Polish & Deploy (Week 6)**
- [ ] Loading skeletons and error boundaries on all pages
- [ ] Toast notifications for user actions
- [ ] Dashboard sentiment and regulation summary widgets
- [ ] API rate limiting on AI routes
- [ ] UAT with 2-3 team members → bug fixes
- [ ] Production deployment on Vercel

### Acceptance Criteria

- [ ] Analyst can log in, see dashboard with real Sensor Tower data
- [ ] Analyst can browse apps with search/filter and view per-app metrics
- [ ] Apify scrapes App Store reviews on schedule and displays them in Community page
- [ ] Regulation scraper extracts game names + G1-G4 categories from abei.gov.vn
- [ ] Analyst can create report, select template + params, click "Generate" and see Claude output stream into editor
- [ ] Analyst can freely edit generated content and change report status (draft → review → published)
- [ ] LLM cost is tracked per section and visible in report metadata
- [ ] Report is exportable as Markdown or HTML
- [ ] All pages load in <3 seconds with loading states

See [implementation-plan.md](./implementation-plan.md) for the full task breakdown with effort estimates and dependencies.

---

## CI/CD Recommendations

| Stage | Tool | Purpose |
|---|---|---|
| Build | Vercel Build | Automatic on git push, Next.js optimized |
| Preview | Vercel Preview Deployments | Per-branch preview URLs for testing |
| Deploy | Vercel Production | Auto-deploy from main branch |
| DB Migrations | Drizzle Kit | `npx drizzle-kit push` before deploy or in build script |
| Monitoring | Vercel Analytics + Inngest Dashboard | Page performance + background job status |

---

## Important Notes for the Implementing Agent

1. **Existing Postgres**: The Sensor Tower tables (`sensor_tower_apps`, `sensor_tower_metrics`) already exist with data. Use `drizzle-kit introspect` to pull their schema — do NOT create migrations for these tables.

2. **abei.gov.vn scraping**: The regulation data is at `https://abei.gov.vn/danh-sach-cap-phep/69`. The HTML structure contains tables with game names, license categories (G1/G2/G3/G4), publisher names, and decision numbers. Use cheerio to parse. The site is in Vietnamese.

3. **Apify App Store actor**: Use the `apify/app-store-scraper` actor. Configure to scrape reviews for the top 50 games in the Vietnamese market. Store results in `community_posts` with `source = "appstore"`.

4. **Report generation prompts**: The two MVP templates are:
   - **Market Overview**: Summarizes top movers, revenue trends, notable entries/exits, and regulatory context for a specific market and month.
   - **Competitive Brief**: Deep-dive comparison of 3-5 specific apps — their metrics, community sentiment, and regulatory status.

5. **Streaming to Tiptap**: Use Vercel AI SDK's `streamText()` on the server and `useChat()` on the client. Insert streamed content as Tiptap JSON nodes section-by-section.

6. **MVP is Claude-only**: Do not implement multi-LLM routing (LiteLLM/Minimax) in MVP. That's planned for Production phase (Weeks 7-12).

7. **Design should be premium**: Dark mode, modern typography (Inter or similar), smooth animations, gradient accents. This is an internal tool but analysts should enjoy using it.

---

## References

- [Project Overview](./overview.md) — Problem, users, NFRs, scope
- [Research](./research.md) — Market analysis, competitor intelligence, API feasibility, risks
- [Findings (Persistent Memory)](./findings.md) — All decisions, ADRs, session notes
- [Tech Stack](./tech-stack.md) — Evaluated alternatives with scored matrices
- [Architecture](./architecture.md) — System diagrams, ERD, data flows, API contracts, folder structure
- [Implementation Plan](./implementation-plan.md) — 6 milestones, 56 tasks, Gantt chart, definitions of done
