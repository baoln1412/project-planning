# Research: Project Viewer

> **Researched**: 2026-03-08  
> **Status**: Complete

## Executive Summary

The Project Viewer concept — a web dashboard for browsing Markdown-based project plans — sits at the intersection of two well-established markets: **developer documentation tools** (~$2.55B technical writing tool market, growing ~8% YoY) and **project management dashboards** (part of the broader $7.5B+ software dev tools market). The demand for structured, visual interfaces over raw files is validated by tools like Docusaurus, MkDocs, GitBook, and Backstage, all of which have achieved significant adoption.

However, **no existing tool directly solves the exact problem**: providing a visual, read-only dashboard specifically designed for browsing a local directory of Markdown project plan files with version history, status tracking, and Mermaid diagram rendering. The closest alternatives either require a complex setup (Backstage), focus on public documentation sites (Docusaurus/MkDocs/VitePress), or are lightweight CLI tools without a polished UI (md-fileserver/mdts).

This creates a genuine **differentiation opportunity**: a purpose-built, zero-config viewer that reads from a `projects/` folder and renders a premium dashboard experience. The technical feasibility is high — mature libraries exist for every required feature (Markdown rendering, Mermaid diagrams, file system access, search).

## Market Analysis

### Market Size & Trends

| Segment | 2025 Value | Growth |
|---|---|---|
| Software Dev Tools | $7.57B | ~10% CAGR |
| Technical Writing Tools | $2.55B | ~8% CAGR |
| API Management | $6.89B | ~15% CAGR |
| AI Developer Tools | $4.5B | ~30% CAGR |

**Key trends fueling demand:**
- **Docs-as-code** philosophy: managing docs alongside code using Markdown, Git, and CI/CD
- **AI integration**: generative AI for writing, editing, and publishing documentation
- **Cloud-based collaboration**: real-time editing and sharing are now expected
- **Version control**: seamless Git integration and built-in versioning
- **Interactive content**: embedding diagrams, widgets, and components in docs

### Target Demographics

- **Solo developers / indie hackers** managing multiple project ideas
- **Small teams** coordinating project planning before development
- **Technical leads** who maintain a portfolio of planned projects
- **Architects & planners** who use Markdown-heavy planning workflows

### Demand Signals

- GitHub has become the most desired tool for code documentation (surpassing Jira)
- 92% of developers use AI tools, contributing to a 25% productivity boost
- Growing community around static site generators (Docusaurus 50K+ GitHub stars, MkDocs/Material 20K+)
- Reddit/forum discussions frequently request lightweight Markdown viewers with folder browsing

## Competitor Analysis

### Direct Competitors

No tool directly replaces the Project Viewer concept. The closest are documentation platforms repurposed for this use case:

| Tool | Type | Pricing | Strengths | Weaknesses | Tech Stack |
|---|---|---|---|---|---|
| **Docusaurus** | Static site generator | Free (OSS) | Rich plugin ecosystem, React/MDX, versioning, search | Heavy setup for simple use case, needs build step, not a dashboard | React, MDX |
| **MkDocs** | Static site generator | Free (OSS) | Python-based, simple YAML config, Material theme | Material theme entering maintenance mode (late 2025), no dashboard view | Python, Jinja2 |
| **VitePress** | Static site generator | Free (OSS) | Blazing fast (Vite), Vue components, minimal config | Vue-centric, no file-system browser dashboard | Vue 3, Vite |
| **GitBook** | Documentation platform | Freemium ($8/mo+) | Polished UI, GitHub integration, real-time editing | SaaS dependency, not local-first, no project status tracking | Proprietary |
| **Notion** | All-in-one workspace | Freemium ($10/mo+) | Flexible, customizable, collaborative | Not Markdown-native, requires import/export, no Mermaid | Proprietary |
| **Confluence** | Enterprise wiki | $6.05/user/mo+ | Enterprise features, Atlassian integration | Heavy, expensive, not developer-friendly for Markdown | Java |

### Indirect Competitors

| Tool | Type | Relevance |
|---|---|---|
| **Backstage** (Spotify) | Developer portal framework | Closest conceptually — centralized hub for docs and projects. But extremely heavy, requires Kubernetes knowledge, designed for large orgs |
| **GitHub Web UI** | Repository browser | Can browse Markdown files natively with rendering. But no dashboard view, no project cards, no status tracking |
| **VS Code** | Code editor | Markdown preview built-in, extensions for Mermaid. But no dashboard — it's a code editor, not a viewer |

### Open Source Alternatives

| Tool | Description | Strengths | Weaknesses |
|---|---|---|---|
| **md-fileserver** | Local server for rendering Markdown in browser | File-system reading, live reload | Basic UI, no dashboard view, no project structure awareness |
| **mdts** (Markdown Tree Server) | CLI tool for browsing Markdown folder | File tree sidebar, offline-first | CLI-only, minimal UI, no cards or status |
| **simple-md-viewer** | React library for Markdown folder display | React component, folder navigation | Library only (not standalone), basic rendering |
| **MarkdownManager** | Self-hosted Markdown viewer/editor | Direct file system access, PHP | Dated UI, PHP stack, no modern features |
| **Typemill** | Lightweight Markdown CMS | File-based storage, YAML config | CMS not viewer, editing-focused, PHP-based |

## Technical Feasibility

### Available APIs & Services

Since this is a **local-first** application reading from the file system, no external APIs are strictly required. Key technical capabilities:

- **File System Access**: Node.js `fs` module for server-side, or static build-time reading
- **Markdown Frontmatter Parsing**: `gray-matter` npm package for extracting metadata from Markdown files
- **Git Integration** (optional): `simple-git` npm package for reading commit history and diffs

### Recommended Libraries

| Need | Library | Why |
|---|---|---|
| **Markdown → HTML** | `react-markdown` + `remark-gfm` | Industry standard, safe (no dangerouslySetInnerHTML), supports GFM tables |
| **Mermaid Diagrams** | `mermaid` (official) | 70K+ npm weekly downloads, supports all diagram types, active development |
| **Alternative Mermaid** | `beautiful-mermaid` | Newer, better themes, synchronous rendering, zero-DOM — worth evaluating |
| **Syntax Highlighting** | `rehype-highlight` or `shiki` | Code block rendering in Markdown |
| **Search** | `flexsearch` or `fuse.js` | Client-side full-text search, lightweight |
| **Date/Time** | `date-fns` | Lightweight date formatting for "last updated" |
| **Markdown Frontmatter** | `gray-matter` | Parse YAML frontmatter from Markdown files |

### Infrastructure Requirements

**Approach A — Static Site (recommended for v1):**
- Build at compile time using Next.js, Astro, or VitePress
- Read `projects/` folder during build, generate static pages
- Deploy to Vercel, Netlify, or GitHub Pages (free)
- **Pro**: Zero runtime dependencies, fast, free hosting
- **Con**: Requires rebuild to see changes

**Approach B — Lightweight Node.js Server:**
- Express/Fastify server reading `projects/` at runtime
- WebSocket or polling for live updates
- **Pro**: Real-time, no rebuild needed
- **Con**: Requires running server process

**Approach C — Hybrid (recommended for v2+):**
- Next.js with ISR (Incremental Static Regeneration) or Astro with server endpoints
- Static pages with on-demand revalidation via file-system watcher
- **Pro**: Best of both worlds — fast static serving with live updates
- **Con**: Slightly more complex setup

### Build Complexity Assessment

| Feature | Complexity | Notes |
|---|---|---|
| Project card grid | Low | Standard UI component |
| Markdown rendering | Low | `react-markdown` handles most cases |
| Mermaid diagrams | Medium | Requires async initialization, SSR considerations |
| Version history (CHANGELOG) | Medium | Parsing CHANGELOG.md or Git history |
| Status filtering | Low | Simple state management |
| Search | Low-Medium | `fuse.js` for fuzzy search across project data |
| Dark mode | Low | CSS variables + toggle |
| Side-by-side version diff | High | Requires diff algorithm and split-pane UI |

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Mermaid SSR issues | Technical | Medium | Medium | Use client-side rendering with loading states; consider `beautiful-mermaid` for sync rendering |
| Scope creep beyond viewer into editor/PM tool | Business | High | High | Enforce read-only scope in v1; resist adding editing features |
| Limited user base (personal/small teams) | Business | Medium | Low | Accept niche scope — this is a personal productivity tool, not a SaaS product |
| Markdown rendering edge cases | Technical | Medium | Low | Use well-tested `react-markdown` + `remark-gfm`; define supported Markdown subset |
| Performance with many projects | Technical | Low | Medium | Static build handles scale well; lazy-load project details |
| Dependency on Mermaid.js maintenance | Technical | Low | Medium | Mermaid.js is actively maintained with 70K+ weekly downloads; low risk |
| Material for MkDocs entering maintenance mode | Technical | Low | Low | Not a direct dependency but relevant trend — validates the need for alternatives |
| Security (XSS from rendered Markdown) | Technical | Medium | High | `react-markdown` escapes HTML by default; use `rehype-sanitize` for extra safety |

## Key Insights

- **No direct competitor exists** — this is a greenfield opportunity in a niche (Markdown project plan viewer/dashboard)
- **All required libraries are mature** — Markdown rendering, Mermaid, search, and file-system access are solved problems
- **Static site approach is ideal for v1** — simplest deployment, free hosting, good enough for a solo/small team use case
- **Differentiation is built-in** — the app is purpose-built for `project-planning` workspace structure, not a generic docs site
- **Biggest risk is scope creep** — the temptation to add editing, task management, or collaboration features should be resisted in v1
- **Mermaid diagram support is critical** — architecture docs in this workspace rely heavily on them
- **VitePress or Next.js are the strongest technical choices** — VitePress for simplicity, Next.js for flexibility

## Sources

1. [Developer documentation tools market analysis](https://mordorintelligence.com) — Software Development Tools Market, 2025
2. [Technical Writing Tool Market](https://wiseguyreports.com) — $2.55B valuation, 2025
3. [Stack Overflow Developer Survey 2025](https://stackoverflow.co) — GitHub adoption trends
4. [Daily.dev Developer Trends](https://daily.dev) — AI tool adoption statistics
5. [Docusaurus](https://docusaurus.io) — Meta's documentation framework
6. [MkDocs](https://mkdocs.org) — Python static site generator
7. [VitePress](https://vitepress.dev) — Vite-powered static site generator
8. [GitBook](https://gitbook.com) — Documentation platform
9. [Mermaid.js](https://mermaid.js.org) — JavaScript diagramming library
10. [react-markdown](https://github.com/remarkjs/react-markdown) — Safe Markdown rendering for React
11. [beautiful-mermaid](https://npmjs.com/package/beautiful-mermaid) — Advanced Mermaid renderer
12. [Backstage](https://backstage.io) — Spotify's developer portal framework
13. [md-fileserver](https://github.com/nicerloop/md-fileserver) — Local Markdown server
14. [mdts](https://github.com/nicholasgasior/mdts) — Markdown Tree Server CLI
15. [simple-md-viewer](https://github.com/nicholasgasior/simple-md-viewer) — React Markdown viewer component
16. [Typemill](https://typemill.net) — Lightweight Markdown CMS
17. [Material for MkDocs maintenance announcement](https://squidfunk.github.io/mkdocs-material) — Transitioning to Zensical
18. [ferndesk.com](https://ferndesk.com) — Docusaurus pricing analysis
