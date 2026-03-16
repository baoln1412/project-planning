# Findings: Project Viewer

> **Last Updated**: 2026-03-08  
> **Phase**: Planning Complete

## Discovered Constraints

### Technical
- **Mermaid.js SSR**: Requires DOM — must render client-side with loading skeleton
- **Markdown XSS**: `react-markdown` escapes by default; add `rehype-sanitize` for belt-and-suspenders
- **chokidar v5**: ESM-only, requires Node.js 20+
- **Version diffing**: CHANGELOG.md or overview.md version table is the data source; not all projects may have this

### Business
- **Small team tool**: ~5 users, personal/team productivity — no monetization needed
- **Read-only viewer**: Source of truth stays in Markdown files, dashboard is view-only

### Regulatory
- None — local-first, no user data collection

## User Decisions (from research Q&A)

| Question | Answer |
|---|---|
| Deployment | Live Node.js server (not static) |
| v1 scope | Includes version comparison / side-by-side diffs |
| Audience | Shared with ~4 teammates (~5 total users) |
| Design | Light / white mode for document readability |

## Key Market Insights

- ~$2.55B technical writing tools market, ~8% YoY growth
- No direct competitor for "Markdown project plan viewer dashboard"
- Material for MkDocs entering maintenance mode — validates need for new tools

## Differentiation Opportunities

1. Purpose-built for `project-planning` workspace structure
2. Zero-config — point at `projects/` and go
3. Status tracking dashboard with emoji badges
4. Side-by-side version diffs from CHANGELOG
5. Mermaid-first rendering for architecture docs

## Tech Stack Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Framework | Next.js 15 | Full-stack (API + frontend), App Router, Turbopack |
| Language | TypeScript | Type safety across entire stack |
| Markdown | react-markdown + remark-gfm | Safe, extensible, GFM tables |
| Diagrams | mermaid (client-side) | Industry standard, comprehensive diagram types |
| Search | fuse.js | Lightweight fuzzy search, client-side |
| File watching | chokidar v5 | ESM-only, efficient cross-platform watcher |
| Diff | diff (npm) | Text diffing for version comparison |

## Architecture Decisions

- **Monolithic MVP**: Single Next.js app, no external services
- **In-memory cache**: Project data cached in Node.js process, invalidated by chokidar
- **LAN access**: Bind to `0.0.0.0` for team access
- **3-phase evolution**: Local → VPS with SSL → Scaled with Redis/CDN

## Risk Items & Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| Scope creep into editing | High | v1 is read-only — hard boundary |
| Mermaid flash of unstyled content | Medium | Loading skeleton while Mermaid initializes |
| Not all projects have CHANGELOG | Medium | Graceful "no version history" state |
| More than 5 users | Low | Phase 2 architecture with PM2 cluster mode |

## Open Questions (Resolved)

All initial questions have been answered. No blockers remain.
