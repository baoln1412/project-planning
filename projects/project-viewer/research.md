# Research: Project Viewer

## Executive Summary

The Project Viewer is a web application designed to visualize and review Markdown-based project plans from the `project-planning` workspace. Research reveals a **strong technical foundation** for building this — mature libraries exist for Markdown rendering (`react-markdown`, `marked`), Mermaid diagram support (`mermaid.js`), and file system access (Node.js `fs` API). The concept sits at the intersection of **documentation viewers** and **project portfolio dashboards**, a space with many general-purpose tools but no purpose-built solution for browsing structured planning folders.

The competitive landscape shows that while tools like Notion, Obsidian, Docusaurus, and MkDocs handle Markdown rendering well, none are designed specifically as a read-only visual dashboard for browsing a structured project planning workspace with versioning. This represents a **clear differentiation opportunity** — a lightweight, focused tool rather than a full knowledge base or documentation platform.

Technical feasibility is **high**. All required capabilities (Markdown parsing, Mermaid rendering, file system access, search) have well-supported open-source libraries. A modern framework like Next.js or Astro can deliver a premium experience with minimal infrastructure.

## Market Analysis

### Market Size & Trends

The broader project management software market was valued at ~$6.68B in 2024 and is projected to grow at a CAGR of ~15.7% through 2030. Key trends driving growth:

- **AI-powered insights** — dashboards increasingly use AI for predictive analytics and risk identification
- **Markdown-first workflows** — developer tools are moving toward Markdown-based documentation (GitHub, Obsidian, Notion)
- **Static site generators** — Astro, Docusaurus, and MkDocs are seeing rapid adoption for documentation sites
- **Low-code dashboards** — tools like Metabase, Appsmith, and ToolJet are making custom dashboards accessible

### Target Demographics

| Segment | Description |
|---|---|
| Solo developers / founders | Planning projects before building, want visual overview |
| Small teams (2–10) | Collaborating on project plans, need shared visibility |
| Technical project managers | Reviewing architecture and implementation plans |
| AI workspace users | Using Antigravity or similar tools, need a way to review generated plans |

### Demand Signals

- Growing adoption of Markdown-based planning tools (Obsidian, Logseq, Notion)
- Increasing demand for "docs-as-code" workflows
- Reddit/HN discussions about better ways to visualize Markdown documentation
- No existing tool specifically designed for browsing structured project planning folders

## Competitor Analysis

### Direct Competitors

There are **no direct competitors** that match this exact use case — a read-only visual dashboard for browsing structured Markdown project plans with versioning. However, several tools overlap in functionality:

| Name | Type | Pricing | Strengths | Weaknesses | Differentiation Gap |
|---|---|---|---|---|---|
| **Notion** | Knowledge base | Free–$10/user/mo | Rich Markdown editing, databases, views | Heavy, not file-based, no Mermaid native | Not focused on read-only project review |
| **Obsidian** | Markdown editor | Free / $50 commercial | File-based, graph view, plugins | Desktop-only (no web), editing-focused | No project dashboard, no web access |
| **GitHub** | Code platform | Free–$21/user/mo | Renders Markdown, version history | Not a dashboard, requires repo navigation | No card-based project overview |
| **Outline** | Wiki | Free (self-host) | Real-time collab, modern UI, Markdown | Requires server, database, not file-based | Overkill for read-only plan viewing |

### Indirect Competitors

| Name | Type | Overlap |
|---|---|---|
| **Docusaurus** | Docs site generator | Renders Markdown with sidebars, but requires build step and is docs-focused |
| **MkDocs** (Material) | Docs site generator | Beautiful Markdown rendering, but linear docs structure, not dashboard |
| **Astro Starlight** | Docs theme | Modern, fast Markdown docs, but not designed for project portfolio browsing |
| **BookStack** | Documentation platform | Organized content, but wiki-style not project-planning-style |

### Open Source Alternatives

| Tool | URL | Relevance |
|---|---|---|
| **Glow** | github.com/charmbracelet/glow | CLI Markdown viewer — terminal only, no web UI |
| **Grip** | github.com/joeyespo/grip | Renders GitHub-flavored Markdown locally — single file, no dashboard |
| **Markserv** | github.com/markserv/markserv | Live Markdown server — basic file listing, no cards/dashboard |
| **Focalboard** | github.com/mattermost/focalboard | Project management boards — Kanban style, not Markdown-plan focused |

## Technical Feasibility

### Available APIs & Services

| Need | Solution | Maturity |
|---|---|---|
| Read Markdown files | Node.js `fs` module | ⭐⭐⭐⭐⭐ Built-in |
| Parse Markdown → HTML | `marked`, `react-markdown`, `unified`/`remark` | ⭐⭐⭐⭐⭐ Very mature |
| Render Mermaid diagrams | `mermaid.js` (client-side SVG rendering) | ⭐⭐⭐⭐⭐ Used by GitHub, Notion |
| Syntax highlighting | `highlight.js`, `Prism.js`, `shiki` | ⭐⭐⭐⭐⭐ Very mature |
| Full-text search | `fuse.js` (client-side fuzzy search) | ⭐⭐⭐⭐ Lightweight |
| Frontend framework | Next.js, Astro, Vite + React | ⭐⭐⭐⭐⭐ Production-ready |
| Table support | GFM (GitHub Flavored Markdown) support in all major parsers | ⭐⭐⭐⭐⭐ |

### Recommended Libraries

| Library | Purpose | npm Weekly Downloads |
|---|---|---|
| `react-markdown` | React component for rendering Markdown | ~3M |
| `mermaid` | Diagram rendering from text | ~1.5M |
| `gray-matter` | Parse YAML frontmatter from Markdown files | ~4M |
| `fuse.js` | Lightweight fuzzy search | ~1.5M |
| `shiki` | Syntax highlighting (used by Astro, VitePress) | ~3M |
| `date-fns` | Date formatting | ~20M |

### Infrastructure Requirements

- **Minimal**: This is a read-only app that reads local files
- **No database needed** — data comes from the filesystem
- **No auth needed** (v1) — single-user local tool
- **Hosting**: Can run locally (`npm run dev`) or deploy as a static site
- **Build time**: Can use Next.js static export or Astro for zero-JS pages

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Mermaid diagrams render inconsistently | Technical | Low | Med | Use official `mermaid.js`, test across diagram types |
| Large number of projects slows load time | Technical | Low | Low | Lazy loading, pagination, static generation |
| Markdown parsing edge cases (tables, nested lists) | Technical | Med | Low | Use `remark-gfm` plugin for GFM support |
| Scope creep into full project management tool | Business | Med | High | Keep scope strict: read-only viewer, no editing |
| No demand beyond personal use | Business | Med | Low | Build for personal use first; open-source later if useful |
| File system access not available in browser | Technical | Low | Med | Use SSR (Next.js) or build-time generation (Astro) for file access |

## Key Insights

- **No direct competitor exists** for this exact use case — a read-only Markdown project plan viewer with versioning and Mermaid support
- **Technical feasibility is very high** — all needed libraries are mature and well-maintained
- **The biggest risk is scope creep** — resist adding editing capabilities; this is a viewer
- **Two viable architecture paths**: (1) Next.js with server-side file reading, or (2) Astro with build-time static generation
- **Mermaid.js is the critical dependency** — it's well-supported but rendering must be client-side
- **Astro Starlight** is the closest existing solution, but it's a docs framework not a project dashboard — differentiating through card-based dashboard UI is key

## Sources

1. [Mermaid.js GitHub](https://github.com/mermaid-js/mermaid) — Diagram rendering library
2. [react-markdown GitHub](https://github.com/remarkjs/react-markdown) — React Markdown renderer
3. [Astro Docs](https://astro.build) — Static site generator with Starlight theme
4. [Docusaurus](https://docusaurus.io) — Meta's documentation framework
5. [MkDocs](https://www.mkdocs.org) — Python-based docs generator
6. [Outline](https://getoutline.com) — Open source knowledge base
7. [Obsidian](https://obsidian.md) — Markdown knowledge management
8. [BookStack](https://www.bookstackapp.com) — Open source documentation platform
9. [Fuse.js](https://fusejs.io) — Lightweight fuzzy search library
10. [OpenProject](https://www.openproject.org) — Open source project management
