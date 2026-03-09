# Handoff: Project Viewer

> **Generated**: 2026-03-09  
> **Status**: 🟢 Ready for Development

---

## Project Overview

A live web dashboard for browsing and reviewing all projects in the `project-planning` workspace. A lightweight **Next.js 15** server reads project Markdown files at runtime from the `projects/` directory and renders them as an interactive UI with project cards, drill-down document views, version comparison, Mermaid diagram rendering, and status tracking. Designed for a small team (~5 people) with a clean, light-themed aesthetic optimized for document readability.

> See full details: [overview.md](file:///Users/lap15230/project-planning/projects/project-viewer/overview.md)

---

## Repository Structure

```
project-viewer/
├── src/
│   ├── app/                          # Next.js App Router pages
│   │   ├── layout.tsx                # Root layout (header, global styles)
│   │   ├── page.tsx                  # Dashboard home (project cards grid)
│   │   ├── projects/
│   │   │   └── [slug]/
│   │   │       ├── page.tsx          # Project detail view
│   │   │       └── diff/
│   │   │           └── page.tsx      # Side-by-side diff comparison
│   │   └── globals.css               # Global styles + light theme variables
│   ├── components/
│   │   ├── ProjectCard.tsx           # Card component for dashboard grid
│   │   ├── StatusBadge.tsx           # Emoji status badge (🟡🟢🔵)
│   │   ├── SearchBar.tsx             # Fuzzy search with fuse.js
│   │   ├── StatusFilter.tsx          # Filter bar by planning status
│   │   ├── MarkdownRenderer.tsx      # react-markdown wrapper with custom components
│   │   ├── MermaidDiagram.tsx        # Client-side Mermaid code block renderer
│   │   ├── DocumentTabs.tsx          # Tab navigation for project docs
│   │   ├── VersionTimeline.tsx       # Version history list
│   │   └── DiffView.tsx             # Side-by-side diff display
│   ├── lib/
│   │   ├── types.ts                  # TypeScript interfaces (shared types)
│   │   ├── projectScanner.ts         # Scan projects/ dir, extract metadata
│   │   ├── documentReader.ts         # Read .md files, parse frontmatter
│   │   ├── cache.ts                  # In-memory cache with chokidar invalidation
│   │   ├── versionParser.ts          # Parse CHANGELOG.md / version tables
│   │   └── watcher.ts               # chokidar file watcher setup
│   └── api/                          # (if using route handlers outside app/)
├── public/                           # Static assets (favicon, etc.)
├── package.json
├── tsconfig.json
├── next.config.ts
├── README.md                         # Setup instructions & usage guide
└── .env.local                        # Environment variables
```

| Directory | Purpose |
|---|---|
| `src/app/` | Next.js App Router pages and layouts |
| `src/components/` | Reusable React components |
| `src/lib/` | Core services — file reading, caching, parsing |
| `public/` | Static assets |

---

## Tech Stack

| Layer | Technology | Version | npm Package |
|---|---|---|---|
| Language | TypeScript | 5.x | `typescript` |
| Framework | Next.js | 15.x | `next` |
| Runtime | Node.js | 20+ | — |
| React | React | 19.x | `react`, `react-dom` |
| Markdown | react-markdown | 9.x | `react-markdown` |
| GFM Tables | remark-gfm | 4.x | `remark-gfm` |
| Diagrams | mermaid | 11.x | `mermaid` |
| File Watcher | chokidar | 5.x | `chokidar` |
| Search | fuse.js | 7.x | `fuse.js` |
| Diff Engine | diff | 7.x | `diff` |
| Frontmatter | gray-matter | 4.x | `gray-matter` |
| Date Formatting | date-fns | 4.x | `date-fns` |
| Syntax Highlighting | rehype-highlight | 7.x | `rehype-highlight` |
| HTML Sanitization | rehype-sanitize | 6.x | `rehype-sanitize` |

> See full tech evaluation: [tech-stack.md](file:///Users/lap15230/project-planning/projects/project-viewer/tech-stack.md)

---

## Data Contracts

### ProjectSummary — `GET /api/projects`

```json
[
  {
    "slug": "project-viewer",
    "name": "Project Viewer",
    "status": "planning",
    "summary": "A web application that provides a visual dashboard...",
    "lastUpdated": "2026-03-08T22:43:36+07:00",
    "documentCount": 6
  }
]
```

### ProjectDetail — `GET /api/projects/:slug`

```json
{
  "slug": "project-viewer",
  "name": "Project Viewer",
  "status": "planning",
  "documents": [
    {
      "type": "overview",
      "content": "# Project Viewer\n...",
      "frontmatter": { "status": "Planning", "created": "2026-03-08" },
      "lastModified": "2026-03-08T22:43:36+07:00"
    }
  ],
  "versions": [
    { "version": "Initial", "date": "2026-03-08", "summary": "Project created" }
  ]
}
```

### DiffResult — `GET /api/projects/:slug/diff?from=v1&to=v2`

```json
{
  "fromVersion": "v1",
  "toVersion": "v2",
  "changes": [
    { "type": "added", "lineNumber": 15, "content": "+ new line" },
    { "type": "removed", "lineNumber": 10, "content": "- old line" },
    { "type": "unchanged", "lineNumber": 9, "content": "  context line" }
  ]
}
```

### SearchResult — `GET /api/search?q=query`

```json
[
  {
    "slug": "project-viewer",
    "name": "Project Viewer",
    "matchedField": "summary",
    "matchedText": "...visual dashboard for browsing...",
    "score": 0.85
  }
]
```

### HealthCheck — `GET /api/health`

```json
{
  "status": "ok",
  "uptime": 3600,
  "projectCount": 5,
  "lastScanTime": "2026-03-09T14:00:00+07:00"
}
```

### TypeScript Interfaces

```typescript
type ProjectStatus = 'planning' | 'ready' | 'in-development';

interface ProjectSummary {
  slug: string;
  name: string;
  status: ProjectStatus;
  summary: string;
  lastUpdated: string;       // ISO 8601
  documentCount: number;
}

interface ProjectDetail {
  slug: string;
  name: string;
  status: ProjectStatus;
  documents: Document[];
  versions: VersionEntry[];
}

interface Document {
  type: 'overview' | 'research' | 'tech-stack' | 'architecture' | 'implementation-plan' | 'handoff';
  content: string;           // Raw Markdown
  frontmatter: Record<string, unknown>;
  lastModified: string;      // ISO 8601
}

interface VersionEntry {
  version: string;
  date: string;
  summary: string;
}

interface DiffResult {
  fromVersion: string;
  toVersion: string;
  changes: DiffChange[];
}

interface DiffChange {
  type: 'added' | 'removed' | 'unchanged';
  lineNumber: number;
  content: string;
}

interface SearchResult {
  slug: string;
  name: string;
  matchedField: string;
  matchedText: string;
  score: number;
}

interface HealthResponse {
  status: 'ok' | 'error';
  uptime: number;            // seconds
  projectCount: number;
  lastScanTime: string;      // ISO 8601
}
```

---

## Environment Setup

### Prerequisites
- Node.js 20+ (required by chokidar v5 and Next.js 15)
- npm or pnpm

### Setup Commands

```bash
# 1. Clone/create the repository
mkdir project-viewer && cd project-viewer

# 2. Initialize Next.js 15 with TypeScript
npx -y create-next-app@latest ./ --typescript --app --eslint --no-tailwind --no-src-dir --import-alias "@/*"

# 3. Install dependencies
npm install react-markdown remark-gfm mermaid chokidar fuse.js diff gray-matter date-fns rehype-highlight rehype-sanitize

# 4. Install dev dependencies
npm install -D @types/diff

# 5. Set up environment variables
cp .env.example .env.local

# 6. Start development server
npm run dev
```

### Environment Variables

| Variable | Description | Default |
|---|---|---|
| `PROJECTS_DIR` | Absolute path to the `projects/` directory to scan | `../project-planning/projects` |
| `HOST` | Host to bind to (`0.0.0.0` for LAN access) | `localhost` |
| `PORT` | Port number | `3000` |

### `.env.example`

```env
# Path to the project-planning workspace's projects folder
PROJECTS_DIR=/Users/lap15230/project-planning/projects

# Bind to all interfaces for team LAN access
HOST=0.0.0.0
PORT=3000
```

---

## Architecture Reference

### Key Architectural Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Architecture | Monolithic Next.js 15 | Single app serves frontend + API; simplest for ~5 users |
| Data access | File-system at runtime | No database needed; reads Markdown directly from disk |
| Caching | In-memory (Node.js process) | Invalidated by chokidar file watcher on changes |
| Rendering | React Server Components + client Mermaid | SSR for Markdown, client-side for Mermaid (needs DOM) |
| Network access | Bind to `0.0.0.0` | Makes dashboard accessible to teammates on LAN |
| Design | Light theme | User preference for document readability |
| Scope | Read-only viewer | No editing — Markdown files are the source of truth |

### Constraints
- **Read-only**: The dashboard must never modify the source Markdown files
- **Mermaid is client-side only**: Requires DOM access; use loading skeletons during init
- **chokidar v5 is ESM-only**: Project must use ES modules (Next.js 15 supports this natively)
- **No database**: All data comes from the file system

> See full architecture with Mermaid diagrams: [architecture.md](file:///Users/lap15230/project-planning/projects/project-viewer/architecture.md)

---

## First Sprint / MVP Scope

Build **Milestones 1–3** first (scaffolding, dashboard, Markdown rendering). This gives a usable product in ~2 weeks.

### MVP Acceptance Criteria
- [ ] `npm run dev` starts a live server
- [ ] Dashboard shows all projects as cards with name, status, summary, last updated
- [ ] Clicking a card opens the full project detail view with tabbed documents
- [ ] Markdown renders with GFM tables, code blocks, and Mermaid diagrams
- [ ] Search bar finds projects by name or keyword
- [ ] Status filter shows/hides projects by planning status
- [ ] Server auto-detects file changes (no restart needed)

### Post-MVP (Milestones 4–5)
- Version history and side-by-side diff comparison
- Keyboard shortcuts, cross-browser polish, team deployment

> See full milestone breakdown: [implementation-plan.md](file:///Users/lap15230/project-planning/projects/project-viewer/implementation-plan.md)

---

## CI/CD Recommendations

For a local team tool, a lightweight pipeline is sufficient:

| Stage | Tool | Purpose |
|---|---|---|
| Lint | ESLint (built-in with Next.js) | Catch code issues |
| Type Check | `tsc --noEmit` | Verify TypeScript correctness |
| Build | `next build` | Verify production build succeeds |
| Deploy | PM2 or systemd | Auto-restart on crash, cluster mode for Phase 2 |

### Suggested `package.json` scripts

```json
{
  "scripts": {
    "dev": "next dev --turbopack -H 0.0.0.0",
    "build": "next build",
    "start": "next start -H 0.0.0.0",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  }
}
```

---

## References

| Document | Path |
|---|---|
| Project Overview | [overview.md](file:///Users/lap15230/project-planning/projects/project-viewer/overview.md) |
| Market Research | [research.md](file:///Users/lap15230/project-planning/projects/project-viewer/research.md) |
| Persistent Findings | [findings.md](file:///Users/lap15230/project-planning/projects/project-viewer/findings.md) |
| Tech Stack Evaluation | [tech-stack.md](file:///Users/lap15230/project-planning/projects/project-viewer/tech-stack.md) |
| System Architecture | [architecture.md](file:///Users/lap15230/project-planning/projects/project-viewer/architecture.md) |
| Implementation Plan | [implementation-plan.md](file:///Users/lap15230/project-planning/projects/project-viewer/implementation-plan.md) |
