# Project Viewer

> **Status**: 🟡 Re-Planning (PM Upgrade)  
> **Size**: 🟡 Medium  
> **Created**: 2026-03-08  
> **Last Updated**: 2026-03-16

## Summary

A web application that provides both a **visual dashboard for browsing project plans** and a **lightweight project management hub** for tracking implementation progress. Built with Next.js 15, it reads project Markdown files from the `projects/` directory and renders them as an interactive UI, while also providing task boards, sprint management, progress tracking, comments, reviews, and issue tracking through a local SQLite database.

## Problem Statement

As the number of planned projects grows, navigating raw Markdown files in folders becomes cumbersome. Additionally, there's no way to track the **execution** of those plans — tasks, sprints, progress, reviews, and issues live outside the planning workspace. A unified interface makes it easier to:
- Get an at-a-glance overview of all projects and their status
- Drill into any project to read its full plan (research, tech stack, architecture, implementation)
- **Track implementation progress** with task boards and sprint management
- **Collaborate** with comments, reviews, and issue tracking
- Compare versions of a project side-by-side
- **Monitor project health** with automated bottleneck detection

## Target Users

- The workspace owner (you) — for reviewing and managing project plans and tracking execution
- Collaborators (~4 teammates) — anyone you share the planning workspace with

## Key Features

### Existing (Viewer)
- [x] Dashboard with project cards showing name, status, summary, and last updated date
- [x] Drill-down view to read all project documents
- [x] Version history viewer — see previous versions and what changed
- [x] Status filtering — filter by planning status (🟡 Planning, 🟢 Ready, 🔵 In Development)
- [x] Markdown rendering — proper rendering including tables and Mermaid diagrams
- [x] Search — find projects by name or keyword
- [x] Dark mode

### New (Project Management)
- [ ] **Task Board** — Kanban board with drag-and-drop (Not Started → In Progress → In Review → Complete → Blocked)
- [ ] **Task Seeding** — Auto-parse implementation-plan.md task tables to bootstrap tasks
- [ ] **Task Health Scoring** — Auto-detect 🟢🟡🔴 based on overdue, blocked, missing owner
- [ ] **Sprint Management** — Create sprints, assign tasks, track sprint progress with burndown chart
- [ ] **Milestone Tracking** — Track milestones with acceptance criteria checklists from Definition of Done
- [ ] **Progress Dashboard** — Project-level progress bars, milestone timeline, traffic-light status
- [ ] **Activity Feed** — Recent task updates, comments, reviews, handoff alerts
- [ ] **Comments** — Thread-based comments on tasks and project documents
- [ ] **Review Workflow** — 3-tier review checklists (critical/important/nice-to-have), formal sign-off
- [ ] **Issue Tracker** — Structured defect/issue log per project
- [ ] **Dependency Visualization** — DAG view of task dependencies using Mermaid
- [ ] **Enhanced Project Cards** — Progress bars, task count (done/total), sprint name, review badges

## Ideas & Notes

- SQLite via Prisma ORM for PM data — zero external dependencies, single file DB
- Markdown files remain read-only source of truth for planning docs
- Task seeding bridges existing Markdown plans to the PM system automatically
- Phase 3 (Scale) would migrate SQLite → PostgreSQL
- 3-tier review system inspired by quality-assurance-checklist skill

## Version History

| Version | Date | Summary |
|---|---|---|
| Initial | 2026-03-08 | Project created as read-only viewer |
| v2 | 2026-03-16 | PM upgrade — added task board, sprints, comments, reviews, issue tracking |
