# Tech Stack: Project Viewer

## Summary

The Project Viewer uses a **TypeScript + Next.js 15** full-stack approach with a lightweight Node.js backend that reads the `projects/` directory at runtime. Next.js API routes serve project data while React Server Components handle the frontend with `react-markdown` for rendering and `mermaid` for diagrams. **A local SQLite database via Prisma ORM** stores project management data (tasks, sprints, comments, reviews, issues). The stack is optimized for a small team (~5 users), developer simplicity, and real-time file updates via `chokidar`.

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | Type safety across full stack; catches errors early; team-standard for modern JS |
| Framework | Next.js | 15.x | Full-stack React framework; API routes + SSR + file-system reading in one package |
| Runtime | Node.js | 20+ | LTS required by dependencies (chokidar 5, Next.js 15); stable and well-supported |
| Markdown Rendering | react-markdown + remark-gfm | 9.x + 4.x | Safe rendering (no dangerouslySetInnerHTML), GFM tables, extensible plugin system |
| Diagram Rendering | mermaid | 11.x | Industry standard for text-based diagrams; 70K+ weekly npm downloads; active development |
| File Watching | chokidar | 5.x | Efficient cross-platform file watcher; ESM-only; minimal dependencies |
| Search | fuse.js | 7.x | Lightweight client-side fuzzy search; no backend search infrastructure needed |
| Diff Engine | diff | 7.x | Text diffing for side-by-side version comparison; well-tested, small footprint |
| Frontmatter Parsing | gray-matter | 4.x | Parse YAML frontmatter from Markdown files for metadata extraction |
| Date Formatting | date-fns | 4.x | Lightweight alternative to Moment.js; tree-shakeable |
| **Database** | **SQLite** | **3.x** | **Zero-config embedded database; single file; sufficient for ~5 concurrent writers with WAL mode** |
| **ORM** | **Prisma** | **6.x** | **Type-safe queries, auto-generated client, declarative migrations, excellent Next.js support** |
| **Drag & Drop** | **@hello-pangea/dnd** | **17.x** | **React 19 compatible fork of react-beautiful-dnd; smooth Kanban drag-and-drop** |
| **Charts** | **recharts** | **2.x** | **React-native, composable charts; lightweight; perfect for burndown and progress charts** |
| Hosting | Self-hosted (local network) | — | Runs on the team's local network or a shared dev server; no cloud dependency |

## Detailed Evaluation

### Frontend Framework

| Criterion | Next.js 15 | Vite + React | VitePress |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Community | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| DX | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Weighted Total** | **4.50** | **4.25** | **3.65** |

**Recommendation**: **Next.js 15** — provides built-in API routes for the backend (file reading, search endpoints), React Server Components for efficient Markdown rendering, and file-system routing. Eliminates the need for a separate backend framework. Turbopack makes dev startup fast.

*Why not Vite + React?* Would require a separate Express/Fastify server for file-system access, adding complexity.  
*Why not VitePress?* Designed for static documentation sites, not dynamic file-reading dashboards.

### Backend API Layer

| Criterion | Next.js API Routes | Express.js | Fastify |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Community | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.45** | **3.65** | **4.20** |

**Recommendation**: **Next.js API Routes** — zero additional setup; co-located with frontend; automatically typed with TypeScript. For ~5 concurrent users, performance difference from Fastify is negligible.

### Markdown Rendering

| Criterion | react-markdown | markdown-to-jsx | MDX |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Safety (XSS) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Plugin Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Mermaid Compat | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.75** | **3.50** | **4.25** |

**Recommendation**: **react-markdown** — built on the unified/remark ecosystem; escapes HTML by default; `remark-gfm` plugin for tables/task lists; custom components for Mermaid code blocks.

### Search

| Criterion | fuse.js | flexsearch | Server-side grep |
|---|---|---|---|
| Setup Complexity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Fuzzy Match | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Bundle Size | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Relevance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.75** | **4.00** | **3.00** |

**Recommendation**: **fuse.js** — excellent fuzzy search for short text datasets (project names, summaries); ~15KB gzipped; runs entirely client-side.

### ORM / Database *(New)*

| Criterion | Prisma + SQLite | better-sqlite3 (direct) | Drizzle + SQLite |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Type Safety | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Migration System | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Next.js Integration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.70** | **3.15** | **4.35** |

**Recommendation**: **Prisma + SQLite** — auto-generated TypeScript client with full type safety, declarative schema with migrations, excellent Next.js integration. SQLite provides zero-config embedded storage.

*Why not better-sqlite3 directly?* Raw SQL queries are error-prone; no migration system; manual type definitions needed.  
*Why not Drizzle?* Excellent alternative, but Prisma has stronger migration tooling and wider community support.

### Drag-and-Drop Library *(New)*

| Criterion | @hello-pangea/dnd | dnd-kit | react-dnd |
|---|---|---|---|
| React 19 Support | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Kanban UX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Setup Simplicity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Accessibility | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Bundle Size | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.80** | **4.20** | **2.80** |

**Recommendation**: **@hello-pangea/dnd** — maintained fork of react-beautiful-dnd (Atlassian) with React 19 support. Best-in-class Kanban drag-and-drop experience out of the box.

*Why not dnd-kit?* More flexible but requires significantly more setup for Kanban boards; hello-pangea gives Kanban UX for free.  
*Why not react-dnd?* Legacy library; worse accessibility; complex setup.

### Charting Library *(New)*

| Criterion | recharts | chart.js + react-chartjs-2 | visx |
|---|---|---|---|
| React-Native | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Simplicity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Bundle Size | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Customization | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Responsive | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Weighted Total** | **4.60** | **3.80** | **4.20** |

**Recommendation**: **recharts** — composable React components; built on D3; responsive by default; simple API for line charts (burndown) and bar charts (progress).

*Why not chart.js?* Canvas-based, not React-native; less composable with React components.  
*Why not visx?* More powerful but more complex setup; overkill for burndown and progress charts.

## Cost Estimate

| Service | Free Tier | Production Est. | Scale Est. |
|---|---|---|---|
| Self-hosted Node.js | $0 (local machine) | $0 (team server) | $5-10/mo (VPS) |
| SQLite database | $0 (embedded) | $0 (embedded) | $0 (or PostgreSQL ~$10/mo) |
| Domain (optional) | — | $12/yr | $12/yr |
| **Total** | **$0/mo** | **$0-1/mo** | **$5-21/mo** |

> [!TIP]
> This project is essentially **free to run** — it's a local Node.js server reading files off disk with an embedded SQLite database. No external database, no cloud services, no subscriptions.

## Alternatives Considered

| Technology | Why Rejected |
|---|---|
| **VitePress** | Static site generator — doesn't support runtime file reading; requires rebuild on every change |
| **Docusaurus** | Too heavy for this use case; designed for public-facing docs, not internal dashboards |
| **Fastify (standalone)** | Would need a separate frontend build; Next.js API routes provide equivalent capability with less setup |
| **Hono** | Excellent for edge/serverless, but this is a local server; Hono's strengths aren't leveraged here |
| **Express.js** | Legacy patterns; slower than alternatives; no native TypeScript; would need separate frontend |
| **beautiful-mermaid** | Newer, fewer diagram types (6 vs all); standard `mermaid` is more battle-tested for now |
| **flexsearch** | Slightly faster but less intuitive fuzzy matching than fuse.js for this use case |
| **PostgreSQL** | Overkill for ~5 users; adds external dependency and setup complexity; reserved for Phase 3 |
| **MongoDB** | Document DB not needed — PM data is relational (tasks → sprints, tasks → milestones) |
| **react-beautiful-dnd** | Unmaintained; does not support React 19; forked as @hello-pangea/dnd |

