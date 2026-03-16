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

---

### 2026-03-16 — Domain Exploration: Operations & Project Management

**Skills Used**: `project-tracker`, `scope-of-work`, `status-update-template`, `retrospective`, `task-prioritization`

#### Key Findings

1. **Data persistence strategy**: SQLite + Prisma ORM is the best fit — maintains zero-external-dependency philosophy, single-file database, easily portable, sufficient for ~5 users with WAL mode
2. **Hybrid architecture**: Keep Markdown files as read-only source of truth for planning docs; add SQLite for PM data (tasks, sprints, comments, reviews)
3. **Task seeding**: Parse task tables from `implementation-plan.md` to auto-bootstrap tasks into the database — bridges existing Markdown plans to new PM system
4. **Project size upgrade**: This feature expansion changes the project from 🟢 Small (viewer) to 🟡 Medium (multi-component web app with read-write capabilities)

#### Data Model Design (from project-tracker skill)

| Entity | Key Properties | Source Skill |
|---|---|---|
| **Task** | title, status (5 stages), priority (4 levels), owner, dueDate, sprintId, milestoneId, dependencies, tags | `project-tracker` |
| **Sprint** | name, goal, startDate, endDate, status (planning/active/completed/cancelled) | `retrospective` |
| **Milestone** | name, description, dueDate, status | `scope-of-work` |
| **Comment** | taskId/documentType, author, content, createdAt | New |
| **Review** | documentType, reviewer, status (pending/approved/changes-requested), comments | New |

#### Feature Design (from skill synthesis)

| Feature | Skill Inspiration | Description |
|---|---|---|
| **Task Board** | `project-tracker` | Kanban with drag-and-drop: Not Started → In Progress → In Review → Complete → Blocked |
| **Sprint Management** | `retrospective` | Sprint list, sprint board, planning, burndown chart, retro integration |
| **Progress Dashboard** | `status-update-template` | Traffic-light status (🟢🟡🔴), milestone timeline, activity feed, weekly snapshot |
| **Comments** | New | Thread-based on tasks and documents, @-mentions, timestamps |
| **Review Workflow** | New | Request review, approve/request-changes, status badges, review history |
| **Enhanced Cards** | `status-update-template` | Progress bar, active sprint name, task count (done/total), review badges |

#### Tech Stack Additions

| Package | Purpose | Rationale |
|---|---|---|
| `prisma` + `@prisma/client` | ORM for SQLite | Type-safe queries, migrations, excellent Next.js integration |
| `@hello-pangea/dnd` | Drag-and-drop | React 19 compatible fork of react-beautiful-dnd |
| `recharts` | Charts | Lightweight, React-native, for burndown and progress charts |

#### New Milestones (expanding from 5 to 8)

| Milestone | Goal | Est. Effort |
|---|---|---|
| M6 | Database Layer & Task CRUD API | 4-5 days |
| M7 | Task Board, Sprint Management, & Progress Dashboard UI | 6-7 days |
| M8 | Comments, Reviews, & Activity Feed | 4-5 days |

#### Architecture Decision Records

**ADR-PM-001: SQLite over PostgreSQL for PM data**
- **Status**: Proposed
- **Context**: Need writable storage for tasks, sprints, comments, reviews. Current app is zero-dependency file-system based.
- **Decision**: Use SQLite via Prisma ORM
- **Consequences**: ✅ Zero external deps, portable, sufficient for ~5 users. ⚠️ Limited concurrent write scalability. Phase 3 (Scale) would need migration to PostgreSQL.

**ADR-PM-002: Hybrid Markdown + Database architecture**
- **Status**: Proposed
- **Context**: Planning documents (overview, architecture, etc.) are Markdown files. PM data (tasks, comments) needs CRUD.
- **Decision**: Keep Markdown read-only, store PM data in SQLite, link via project slug.
- **Consequences**: ✅ Markdown files remain source of truth for planning. ✅ No risk of corrupting project docs. ⚠️ Two data sources to keep in sync.

**ADR-PM-003: Task seeding from implementation-plan.md**
- **Status**: Proposed
- **Context**: Projects already have task tables in `implementation-plan.md`. Users shouldn't have to re-enter them.
- **Decision**: Build a parser that reads Markdown task tables and creates corresponding Task records in SQLite.
- **Consequences**: ✅ Zero manual data entry. ⚠️ Parser must handle varying table formats gracefully.

#### Discovered Constraints (New)

- **Read-only boundary removed**: The original constraint "read-only viewer" is being relaxed. The system will now write to SQLite but still never modify source Markdown files.
- **SQLite concurrency**: WAL mode handles ~5 concurrent writers. Phase 3 scaling would require PostgreSQL migration.
- **React 19 DnD compatibility**: `react-beautiful-dnd` is unmaintained; `@hello-pangea/dnd` is the maintained fork for React 19.

---

### 2026-03-16 — Domain Exploration: Process & Quality (Round 2)

**Skills Used**: `workflow-mapper`, `quality-assurance-checklist`

#### Key Findings

1. **From workflow-mapper — Bottleneck Detection**: The automation scoring concept (rule-based scoring to detect issues) can be adapted as a "task health score" for the PM dashboard. Tasks scored by: overdue days (+3), blocked status (+3), missing owner (+2), dependency-stalled (+2). Total 7+ = 🔴, 4-6 = 🟡, 0-3 = 🟢.

2. **From workflow-mapper — Dependency Visualization**: Task dependencies should be shown as a simple DAG (directed acyclic graph) within the sprint/milestone view. This reuses the Mermaid rendering already in the project-viewer.

3. **From workflow-mapper — Handoff Tracking**: When a task changes owner (e.g., developer → reviewer), this is a "handoff" that should be logged in the activity feed and highlighted in notifications.

4. **From quality-assurance-checklist — 3-Tier Review System**: Reviews should use a tiered checklist: Critical (blocks approval), Important (flag if failed), Nice-to-Have. This replaces a simple approve/reject flow with a more structured quality gate.

5. **From quality-assurance-checklist — Acceptance Criteria**: Each milestone's "Definition of Done" from `implementation-plan.md` should be parsed into checkable acceptance criteria items that must pass before the milestone can be marked complete.

6. **From quality-assurance-checklist — Defect/Issue Log**: Add a simple issue tracking system per project — date, description, severity, root cause, fix. Enhances the comment system with structured issue tracking.

#### Feature Enhancements (from Round 2)

| Enhancement | Applies To | Source Skill |
|---|---|---|
| **Task health scoring** (auto-detect 🟢🟡🔴) | Task Board, Dashboard | `workflow-mapper` |
| **Dependency DAG visualization** | Sprint view, Milestone view | `workflow-mapper` |
| **Handoff alerts** in activity feed | Activity Feed | `workflow-mapper` |
| **3-tier review checklists** (critical/important/nice-to-have) | Review Workflow | `quality-assurance-checklist` |
| **Acceptance criteria checklist** per milestone | Milestone tracking | `quality-assurance-checklist` |
| **Issue tracker** (structured defect log) | Comments / Issues tab | `quality-assurance-checklist` |
| **Formal sign-off** (reviewer + date + status) | Review Workflow | `quality-assurance-checklist` |

