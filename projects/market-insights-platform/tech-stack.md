# Tech Stack: Market Insights Platform

## Summary

The Market Insights Platform uses a **Next.js 15 (App Router)** full-stack architecture with **Vercel AI SDK** for streaming LLM integration. The existing **PostgreSQL** database (with Sensor Tower data) is extended via **Drizzle ORM** for new tables. **Tiptap** powers the collaborative report editor with LLM content insertion. **Apify SDK** handles community data scraping, and a custom scraper collects Vietnamese regulation data from abei.gov.vn. LLM routing uses direct API calls to **Claude** (MVP) and later **LiteLLM** for multi-model dispatch (Production phase).

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | Type-safe across frontend, API, and LLM schemas |
| Framework | Next.js | 15.x | App Router + Server Components + API Routes + Vercel deployment |
| UI Components | shadcn/ui | latest | Copy-paste components, zero vendor lock-in, dashboard-ready |
| Rich Text Editor | Tiptap | 2.x | ProseMirror-based, AI Agent toolkit, collaborative editing via Yjs |
| Charts | Recharts | 2.x | Composable React charts for dashboard metrics |
| LLM SDK | Vercel AI SDK | 6.x | Streaming-first, `useChat` hooks, multi-provider, type-safe |
| LLM Gateway (Phase 2+) | LiteLLM | latest | Open-source routing proxy for Minimax + Claude dispatch |
| Database | PostgreSQL | 15+ | Already has Sensor Tower data; extend with new tables |
| ORM | Drizzle | 0.3x | Lightweight, TypeScript-first, great migration tooling |
| Scraping | Apify SDK | 3.x | Pre-built actors for Reddit, Discord, Facebook, App Store |
| Regulation Scraper | Custom (Cheerio) | — | Simple HTML parser for abei.gov.vn; no Apify actor exists |
| Auth | NextAuth.js (Auth.js) | 5.x | Simple email/password for 5 users, extensible |
| Background Jobs | Inngest | latest | Event-driven, serverless-compatible, scheduling |
| Styling | Tailwind CSS | 4.x | Utility-first, AI code generation friendly |
| Hosting | Vercel | — | Zero-config Next.js, preview URLs, edge functions |
| CI/CD | Vercel Git Integration | — | Auto-deploy on push, preview per branch |

## Detailed Evaluation

### Rich Text Editor (Critical for Report Generation)

| Criterion | Tiptap | Plate | Draft.js |
|---|---|---|---|
| Foundation | ⭐⭐⭐⭐⭐ (ProseMirror) | ⭐⭐⭐⭐ (Slate) | ⭐⭐⭐ (Legacy) |
| Collaboration | ⭐⭐⭐⭐⭐ (Yjs + Hocuspocus) | ⭐⭐⭐⭐ (Yjs plugin) | ⭐⭐ (No built-in) |
| LLM Integration | ⭐⭐⭐⭐⭐ (AI Agent toolkit) | ⭐⭐⭐⭐ (AI SDK plugin) | ⭐⭐ (Manual) |
| Community | ⭐⭐⭐⭐⭐ (34K+ GitHub) | ⭐⭐⭐⭐ (15K+ GitHub) | ⭐⭐⭐ (Declining) |
| Extensibility | ⭐⭐⭐⭐⭐ (Node/Mark system) | ⭐⭐⭐⭐⭐ (Plugin system) | ⭐⭐⭐ (Limited) |
| Documentation | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.9** | **4.2** | **2.7** |

**Recommendation**: **Tiptap** — Best-in-class for our use case. The AI Agent toolkit provides built-in support for streaming LLM content into the editor, accept/reject suggestions, and context-aware editing. Yjs + Hocuspocus enables real-time collaboration for 5 users with minimal setup. ProseMirror foundation ensures structured document output for report formatting.

### LLM Integration Layer

| Criterion | Vercel AI SDK v6 | LangChain.js | Direct API Calls |
|---|---|---|---|
| Streaming UI | ⭐⭐⭐⭐⭐ (useChat hooks) | ⭐⭐⭐ (Manual setup) | ⭐⭐ (Raw SSE) |
| Multi-Provider | ⭐⭐⭐⭐⭐ (25+ providers) | ⭐⭐⭐⭐⭐ (200+ integrations) | ⭐⭐ (One at a time) |
| Type Safety | ⭐⭐⭐⭐⭐ (Zod schemas) | ⭐⭐⭐⭐ (TypeScript) | ⭐⭐⭐ (Manual types) |
| Bundle Size | ⭐⭐⭐⭐⭐ (34-60kB) | ⭐⭐⭐ (Heavy) | ⭐⭐⭐⭐⭐ (Minimal) |
| Next.js Fit | ⭐⭐⭐⭐⭐ (React Server Actions) | ⭐⭐⭐⭐ (API routes) | ⭐⭐⭐⭐ (API routes) |
| Agent/RAG | ⭐⭐⭐⭐ (Built-in tools) | ⭐⭐⭐⭐⭐ (LangGraph) | ⭐⭐ (Manual) |
| **Weighted Total** | **4.7** | **4.0** | **3.0** |

**Recommendation**: **Vercel AI SDK v6** — Purpose-built for streaming LLM outputs into React UIs. The `useChat` hook, structured outputs with Zod, and multi-provider support make it ideal for our report generation workflow. For MVP, we don't need LangChain's complex agent orchestration. If needed later, the `@ai-sdk/langchain` interop package bridges both ecosystems.

### Database ORM

| Criterion | Drizzle | Prisma | Raw SQL |
|---|---|---|---|
| Cold Start (Serverless) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ (Slower) | ⭐⭐⭐⭐⭐ |
| TypeScript DX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Migration Tooling | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ (Manual) |
| Existing DB Support | ⭐⭐⭐⭐⭐ (Pull schema) | ⭐⭐⭐⭐⭐ (Introspect) | ⭐⭐⭐⭐⭐ |
| SQL-like API | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ (Custom DSL) | ⭐⭐⭐⭐⭐ |
| Bundle Size | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Weighted Total** | **4.9** | **4.0** | **3.7** |

**Recommendation**: **Drizzle** — Lightweight, SQL-like API that works naturally with the existing Postgres sensor tower tables. Excellent cold start performance on Vercel serverless functions. Schema introspection can pull existing table definitions without migration conflicts.

### Background Job Processing

| Criterion | Inngest | BullMQ | Vercel Cron |
|---|---|---|---|
| Serverless Compatible | ⭐⭐⭐⭐⭐ | ⭐⭐ (Needs Redis) | ⭐⭐⭐⭐⭐ |
| Event-Driven | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ (Cron only) |
| Retry/Failure Handling | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Long-Running Tasks | ⭐⭐⭐⭐⭐ (Step functions) | ⭐⭐⭐⭐ | ⭐⭐ (10s limit) |
| Cost | ⭐⭐⭐⭐⭐ (Free tier) | ⭐⭐⭐ (Redis cost) | ⭐⭐⭐⭐⭐ (Free) |
| **Weighted Total** | **4.8** | **3.8** | **3.2** |

**Recommendation**: **Inngest** — Serverless-native, event-driven, with step functions for long-running report generation (which can take minutes). Free tier is sufficient for MVP. No Redis dependency. Built-in retry and failure handling critical for scraping reliability.

## Cost Estimate

| Service | Free Tier | Production (~$/mo) | Scale (~$/mo) |
|---|---|---|---|
| Hosting (Vercel) | $0/mo (Hobby) | $20/mo (Pro) | $20/mo (Pro) |
| Database (Supabase or existing Postgres) | $0/mo | $25/mo (Supabase Pro) | $25/mo |
| LLM APIs (Claude MVP) | ~$30-50/mo (5 users) | ~$50-100/mo | ~$200-500/mo |
| LLM APIs (Minimax, Phase 2+) | — | ~$10-30/mo (routing saves) | ~$30-80/mo |
| Apify | $0/mo (free tier) | $49/mo (Starter) | $149/mo |
| Inngest | $0/mo (free tier) | $0/mo (within limits) | $50/mo |
| Tiptap (Collaboration) | $0/mo (open-source core) | $0/mo (self-host Yjs) | $99/mo (Cloud) |
| **Total** | **~$30-50/mo** | **~$155-235/mo** | **~$475-825/mo** |

> [!TIP]
> MVP launches at **~$30-50/mo** — primarily Claude API costs. Adding Minimax routing in Phase 2 reduces LLM costs by ~60%, bringing total down despite adding more data sources.

## Alternatives Considered

| Technology | Why Rejected |
|---|---|
| **Plate (Slate)** | Tiptap has stronger AI Agent toolkit and more mature collaboration (Yjs + Hocuspocus) |
| **LangChain.js** | Overkill for MVP; Vercel AI SDK provides simpler streaming with better Next.js integration. Interop package available if needed later |
| **Prisma** | Slower cold starts on serverless; Drizzle is lighter and more SQL-native for working with existing tables |
| **BullMQ** | Requires Redis infrastructure; Inngest is serverless-native with no additional services |
| **Vercel Cron** | 10s execution limit too short for report generation and scraping tasks |
| **Firebase** | NoSQL not ideal for relational market data; existing Postgres already has Sensor Tower tables |
| **Portkey AI Gateway** | More enterprise features than needed for 5-user MVP; LiteLLM is simpler and fully open-source |
| **Custom LLM Router** | LiteLLM provides proven routing with cost tracking; building custom wastes development time |
| **Tailwind 3.x** | v4.x is latest with improved performance and simpler configuration |
