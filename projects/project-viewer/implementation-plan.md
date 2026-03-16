# Implementation Plan: Project Viewer

> **Team Size**: 1 developer (solo build)  
> **Estimated Timeline**: 3–4 weeks  
> **Milestones**: 5

---

## Milestone 1: Project Scaffolding & Core Data Layer

**Goal**: Set up the Next.js project, implement file-system reading, and build the project data API.

**Estimated Effort**: 3–4 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 1.1 | Initialize Next.js 15 project with TypeScript, App Router | 1h | — |
| 1.2 | Define TypeScript interfaces (ProjectSummary, ProjectDetail, Document, etc.) | 1h | 1.1 |
| 1.3 | Build `ProjectScanner` service — scan `projects/` dir, read `overview.md`, extract metadata | 3h | 1.2 |
| 1.4 | Build `DocumentReader` service — read any `.md` file, parse frontmatter with `gray-matter` | 2h | 1.2 |
| 1.5 | Implement in-memory cache for project data | 2h | 1.3 |
| 1.6 | Set up `chokidar` file watcher — invalidate cache on file changes | 2h | 1.5 |
| 1.7 | Create API routes: `GET /api/projects`, `GET /api/projects/[slug]` | 3h | 1.3, 1.4 |
| 1.8 | Create API route: `GET /api/health` | 0.5h | 1.1 |

### Definition of Done
- [ ] `npm run dev` starts successfully
- [ ] `/api/projects` returns a list of all projects with name, status, summary, lastUpdated
- [ ] `/api/projects/[slug]` returns all documents for a specific project
- [ ] Modifying a Markdown file triggers cache invalidation
- [ ] TypeScript compiles with no errors

---

## Milestone 2: Dashboard UI & Project Cards

**Goal**: Build the main dashboard page with project cards, status indicators, and navigation.

**Estimated Effort**: 4–5 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 2.1 | Design and implement the global layout (header, main content area, footer) | 3h | M1 |
| 2.2 | Set up the light theme design system (CSS variables, typography, spacing) | 3h | 2.1 |
| 2.3 | Build `ProjectCard` component (name, status badge, summary, last updated) | 3h | 2.2 |
| 2.4 | Build the dashboard grid page — fetch from `/api/projects`, render cards | 3h | 2.3 |
| 2.5 | Implement status emoji badges (🟡 Planning, 🟢 Ready, 🔵 In Development) | 1h | 2.3 |
| 2.6 | Add status filter bar — filter cards by planning status | 2h | 2.4 |
| 2.7 | Integrate `fuse.js` — search bar with fuzzy matching across project names/summaries | 3h | 2.4 |
| 2.8 | Add responsive design — works on tablet and desktop | 2h | 2.4 |
| 2.9 | Add loading states and empty states | 1h | 2.4 |

### Definition of Done
- [ ] Dashboard shows all projects as cards in a responsive grid
- [ ] Status filter correctly shows/hides cards
- [ ] Search bar finds projects by name or keyword
- [ ] Cards link to project detail pages
- [ ] UI uses light theme with clean typography

---

## Milestone 3: Project Detail View & Markdown Rendering

**Goal**: Build the drill-down view for reading all project documents with full Markdown and Mermaid support.

**Estimated Effort**: 5–6 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 3.1 | Build project detail page layout — sidebar nav (doc tabs) + content area | 4h | M2 |
| 3.2 | Integrate `react-markdown` + `remark-gfm` for Markdown rendering | 3h | 3.1 |
| 3.3 | Style rendered Markdown (headings, tables, code blocks, lists, blockquotes) | 4h | 3.2 |
| 3.4 | Integrate `mermaid` for diagram rendering — custom component for Mermaid code blocks | 4h | 3.2 |
| 3.5 | Add syntax highlighting for code blocks with `rehype-highlight` | 2h | 3.2 |
| 3.6 | Build document tab navigation — switch between overview, research, tech-stack, etc. | 2h | 3.1 |
| 3.7 | Handle missing documents gracefully (show "not yet created" state) | 1h | 3.6 |
| 3.8 | Add "back to dashboard" navigation | 0.5h | 3.1 |
| 3.9 | Add GitHub-style alert rendering (`[!NOTE]`, `[!WARNING]`, etc.) | 2h | 3.2 |

### Definition of Done
- [ ] Clicking a project card navigates to its detail page
- [ ] All document types render correctly (overview, research, tech-stack, architecture, implementation-plan, handoff)
- [ ] Markdown tables render cleanly
- [ ] Mermaid diagrams render as SVG
- [ ] Code blocks have syntax highlighting
- [ ] GitHub-style alerts render with appropriate styling

---

## Milestone 4: Version History & Diff Comparison

**Goal**: Implement the version history viewer and side-by-side diff comparison.

**Estimated Effort**: 4–5 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 4.1 | Build CHANGELOG parser — extract version entries from `CHANGELOG.md` or version history table in `overview.md` | 3h | M1 |
| 4.2 | Create API route: `GET /api/projects/[slug]/versions` | 2h | 4.1 |
| 4.3 | Create API route: `GET /api/projects/[slug]/diff` | 3h | 4.1 |
| 4.4 | Build version history timeline UI — list of versions with dates and summaries | 3h | 4.2 |
| 4.5 | Build side-by-side diff view component — split pane with highlighted changes | 6h | 4.3 |
| 4.6 | Integrate `diff` library for text comparison | 2h | 4.5 |
| 4.7 | Add version selector dropdowns (from version → to version) | 2h | 4.4, 4.5 |
| 4.8 | Style diff view — green for additions, red for deletions, line numbers | 2h | 4.5 |

### Definition of Done
- [ ] Version history shows all past versions with dates
- [ ] Users can select two versions to compare
- [ ] Side-by-side diff view highlights additions and deletions
- [ ] Diff view shows line numbers
- [ ] Works for all document types (overview, research, etc.)

---

## Milestone 5: Polish, Testing & Team Deployment

**Goal**: Final polish, performance optimization, documentation, and deployment for team use.

**Estimated Effort**: 3–4 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 5.1 | Performance audit — optimize re-renders, cache efficiency, Mermaid init | 3h | M3, M4 |
| 5.2 | Cross-browser testing (Chrome, Firefox, Safari) | 2h | M3, M4 |
| 5.3 | Add keyboard shortcuts (Ctrl+K for search, Esc to close, arrow keys for navigation) | 2h | M2 |
| 5.4 | Write README.md — setup instructions, environment requirements, usage guide | 2h | All |
| 5.5 | Configure `0.0.0.0` binding for LAN access | 0.5h | M1 |
| 5.6 | Add `npm run start` production script | 1h | M1 |
| 5.7 | Create `/api/health` dashboard diagnostics (project count, uptime, last scan time) | 1h | 1.8 |
| 5.8 | Final UI polish — transitions, hover effects, micro-animations | 3h | M3, M4 |
| 5.9 | Team onboarding — share URL, verify access for all teammates | 1h | 5.5 |

### Definition of Done
- [ ] All features work reliably across Chrome, Firefox, Safari
- [ ] README documents setup and usage
- [ ] Teammates can access the dashboard over LAN
- [ ] Search, filtering, and navigation feel responsive (<200ms)
- [ ] No console errors or TypeScript warnings

---

## Milestone 6: Database Layer & Task Management API

**Goal**: Add SQLite database via Prisma ORM, implement task CRUD API, and build task seeding from implementation-plan.md.

**Estimated Effort**: 4–5 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 6.1 | Install Prisma + `@prisma/client`, configure SQLite provider | 1h | M1 |
| 6.2 | Design Prisma schema: Task, Sprint, Milestone, Comment, Review, Issue entities | 3h | 6.1 |
| 6.3 | Run Prisma migrations, generate client | 0.5h | 6.2 |
| 6.4 | Build `TaskService` — CRUD operations for tasks (create, read, update, delete, reorder) | 4h | 6.3 |
| 6.5 | Build `SprintService` — CRUD for sprints, assign/unassign tasks to sprints | 3h | 6.3 |
| 6.6 | Build `MilestoneService` — CRUD for milestones, acceptance criteria tracking | 2h | 6.3 |
| 6.7 | Build `CommentService` — CRUD for comments on tasks and documents | 2h | 6.3 |
| 6.8 | Build `ReviewService` — CRUD for reviews with 3-tier checklists and sign-off | 2h | 6.3 |
| 6.9 | Build `IssueService` — CRUD for structured issue/defect log | 1.5h | 6.3 |
| 6.10 | Build `TaskSeeder` — parse Markdown task tables from implementation-plan.md | 4h | 6.4 |
| 6.11 | Create API routes: `/api/projects/[slug]/tasks` (GET, POST, PATCH, DELETE) | 3h | 6.4 |
| 6.12 | Create API routes: `/api/projects/[slug]/sprints` (GET, POST, PATCH) | 2h | 6.5 |
| 6.13 | Create API routes: `/api/projects/[slug]/milestones` (GET, POST, PATCH) | 1.5h | 6.6 |
| 6.14 | Create API routes: `/api/projects/[slug]/comments` (GET, POST) | 1.5h | 6.7 |
| 6.15 | Create API routes: `/api/projects/[slug]/reviews` (GET, POST, PATCH) | 1.5h | 6.8 |
| 6.16 | Create API routes: `/api/projects/[slug]/issues` (GET, POST, PATCH) | 1h | 6.9 |
| 6.17 | Implement task health scoring algorithm (overdue, blocked, missing owner → 🟢🟡🔴) | 2h | 6.4 |
| 6.18 | Add activity logging — record task status changes, comments, reviews as events | 2h | 6.4, 6.7, 6.8 |

### Definition of Done
- [ ] Prisma schema compiles and migrations run successfully
- [ ] All CRUD API routes return correct data
- [ ] Task seeder parses task tables from at least 3 existing projects
- [ ] Task health scoring correctly calculates 🟢🟡🔴 for test cases
- [ ] Activity log captures task transitions, new comments, and review events
- [ ] SQLite database is created in project root as `data/project-viewer.db`

---

## Milestone 7: Task Board, Sprint Management & Progress Dashboard UI

**Goal**: Build the interactive PM UI — Kanban board with drag-and-drop, sprint management views, and progress dashboard.

**Estimated Effort**: 6–7 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 7.1 | Install and configure `@hello-pangea/dnd` for drag-and-drop | 1h | M6 |
| 7.2 | Build `TaskCard` component (title, owner, priority badge, due date, health indicator) | 3h | 7.1 |
| 7.3 | Build `KanbanBoard` component — 5 columns with drag-and-drop between status lanes | 6h | 7.2 |
| 7.4 | Build `TaskDetailPanel` — slide-out panel for viewing/editing task details | 4h | 7.2 |
| 7.5 | Build `TaskCreateModal` — form for creating new tasks with all fields | 2h | 7.3 |
| 7.6 | Build `SprintList` component — list of sprints with name, dates, status, progress bar | 3h | M6 |
| 7.7 | Build `SprintBoard` component — filtered Kanban showing only active sprint tasks | 2h | 7.3, 7.6 |
| 7.8 | Build `SprintPlanningView` — drag unassigned tasks into sprint backlog | 3h | 7.3, 7.6 |
| 7.9 | Install and configure `recharts` for charts | 0.5h | M6 |
| 7.10 | Build `BurndownChart` component — tasks remaining vs. days left in sprint | 3h | 7.9, 7.6 |
| 7.11 | Build `ProgressDashboard` page — project-level progress, milestone timeline, health overview | 5h | 7.9, M6 |
| 7.12 | Build `MilestoneTimeline` component — horizontal bar with milestone completion status | 3h | 7.11 |
| 7.13 | Build `AcceptanceCriteriaChecklist` — checkable items per milestone from Definition of Done | 2h | 7.12 |
| 7.14 | Build `DependencyDAG` component — Mermaid-based task dependency visualization | 3h | 7.3 |
| 7.15 | Update `ProjectCard` component — add progress bar, task count, sprint name, health badge | 2h | M2, M6 |
| 7.16 | Add task board tab to project detail page navigation | 1h | 7.3 |

### Definition of Done
- [ ] Kanban board renders tasks in correct columns
- [ ] Drag-and-drop changes task status and persists to database
- [ ] Sprint board shows only tasks in the active sprint
- [ ] Burndown chart renders correctly with real sprint data
- [ ] Progress dashboard shows project-level metrics
- [ ] Milestone timeline displays milestones with acceptance criteria
- [ ] Dependency DAG renders task dependency graph using Mermaid
- [ ] Enhanced project cards show progress and task counts

---

## Milestone 8: Comments, Reviews, Issues & Activity Feed

**Goal**: Implement the collaboration features — comments, review workflow, issue tracking, and activity feed.

**Estimated Effort**: 4–5 days

### Tasks

| # | Task | Effort | Dependencies |
|---|---|---|---|
| 8.1 | Build `CommentThread` component — list of comments with author, timestamp, content | 3h | M6 |
| 8.2 | Build `CommentForm` component — text input with submit for adding comments | 1h | 8.1 |
| 8.3 | Integrate comments into `TaskDetailPanel` — comment thread below task details | 1h | 7.4, 8.1 |
| 8.4 | Integrate document-level comments — comment thread per document tab | 2h | M3, 8.1 |
| 8.5 | Build `ReviewPanel` component — 3-tier checklist (critical/important/nice-to-have) with pass/fail | 4h | M6 |
| 8.6 | Build `ReviewSignOff` component — reviewer name, date, approve/request-changes buttons | 2h | 8.5 |
| 8.7 | Add review status badges to document tabs (pending/approved/changes-requested) | 1.5h | 8.5 |
| 8.8 | Build `IssueList` component — structured issue log table with severity, status, root cause | 3h | M6 |
| 8.9 | Build `IssueCreateForm` — form for reporting new issues | 1.5h | 8.8 |
| 8.10 | Build `ActivityFeed` component — chronological feed of task updates, comments, reviews, handoffs | 4h | 6.18 |
| 8.11 | Add activity feed to project detail sidebar | 1h | 8.10 |
| 8.12 | Add handoff alerts — highlight when tasks change owner in activity feed | 1h | 8.10 |
| 8.13 | Add issues tab to project detail page navigation | 0.5h | 8.8 |

### Definition of Done
- [ ] Comments can be added and display correctly on tasks and documents
- [ ] Review workflow allows creating 3-tier checklists and signing off
- [ ] Review status badges display on document tabs
- [ ] Issues can be created with severity, description, root cause
- [ ] Activity feed shows chronological events with correct formatting
- [ ] Handoff alerts appear when task ownership changes

---

## Timeline Summary

```mermaid
gantt
    title Project Viewer — Implementation Timeline (with PM Upgrade)
    dateFormat  YYYY-MM-DD
    axisFormat  %b %d

    section Viewer (Original)
    Scaffolding & Data Layer     :m1, 2026-03-10, 4d
    Dashboard UI & Cards         :m2, after m1, 5d
    Detail View & Markdown       :m3, after m2, 6d
    Version History & Diff       :m4, after m3, 5d
    Polish & Deployment          :m5, after m4, 4d

    section PM Upgrade (New)
    Database Layer & Task API    :m6, after m5, 5d
    Task Board & Sprint UI      :m7, after m6, 7d
    Comments, Reviews & Feed    :m8, after m7, 5d
```

## Risks & Assumptions

| Assumption | Risk if Wrong | Mitigation |
|---|---|---|
| Solo developer builds all milestones | Timeline slips if blocked | Milestones are independent enough to be parallelized if team expands |
| Version history comes from CHANGELOG.md or overview.md table | Not all projects have version tables | Gracefully show "no version history" state |
| ~5 concurrent users | More users could cause performance issues | Phase 2 architecture adds PM2 cluster mode |
| Projects follow the `project-planning` folder convention | Unexpected folder structures cause parsing errors | Defensive parsing with fallbacks for missing files |
| Mermaid diagrams render client-side | Flash of unstyled content on load | Add loading skeleton while Mermaid initializes |
| SQLite is sufficient for ~5 writers | Concurrent write contention | WAL mode enabled; Phase 3 migrates to PostgreSQL |
| implementation-plan.md task tables follow consistent format | Parser fails on non-standard tables | Defensive parsing with fallback to manual task creation |
| `@hello-pangea/dnd` supports React 19 | DnD library breaks with React 19 | Fallback to `dnd-kit` which is also React 19 compatible |
| Prisma SQLite adapter is stable | Migration issues or query bugs | `better-sqlite3` as direct fallback, Prisma is well-tested with SQLite |

