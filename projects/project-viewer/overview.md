# Project Viewer

> **Status**: 🟡 Planning  
> **Created**: 2026-03-08  
> **Last Updated**: 2026-03-08

## Summary

A live web dashboard for browsing and reviewing all projects in the `project-planning` workspace. A lightweight Node.js server reads project Markdown files at runtime and renders them as an interactive, team-friendly UI with project cards, drill-down document views, version comparison, and status tracking. Designed for a small team (~5 people) with a clean, light-themed aesthetic optimized for document readability.

## Problem Statement

As the number of planned projects grows, navigating raw Markdown files in folders becomes cumbersome. A visual interface makes it easier to:
- Get an at-a-glance overview of all projects and their status
- Drill into any project to read its full plan (research, tech stack, architecture, implementation)
- Compare versions of a project side-by-side
- Track which projects are ready for development vs. still in planning

## Target Users

- **Primary**: The workspace owner — reviewing and managing project plans
- **Secondary**: ~4 teammates — collaborators who need read access to project plans

## Key Features (v1 — MVP)

- [ ] Dashboard with project cards showing name, status, summary, and last updated date
- [ ] Drill-down view to read all project documents (overview, research, tech stack, architecture, implementation plan, handoff)
- [ ] Version history viewer — see previous versions and what changed (CHANGELOG)
- [ ] Side-by-side version diff comparison
- [ ] Status filtering — filter by planning status (🟡 Planning, 🟢 Ready, 🔵 In Development)
- [ ] Markdown rendering — proper rendering of Markdown including tables and Mermaid diagrams
- [ ] Search — find projects by name or keyword
- [ ] Light theme — clean white mode for best document readability

## Post-v1 Ideas

- Dark mode toggle
- Export to PDF
- Project comparison (side-by-side projects, not just versions)
- Commenting / annotations
- Live collaboration features

## Architecture Decisions

- **Live Node.js server** (not static site) — reads `projects/` directory at runtime for real-time updates
- **File watcher** — auto-detects changes when Markdown files are added/modified
- **Read-only** — no editing from the dashboard; source of truth stays in the Markdown files
- **Team-shared** — accessible over the local network for ~5 teammates

## Version History

| Version | Date | Summary |
|---|---|---|
| Initial | 2026-03-08 | Project created |
| Refined | 2026-03-08 | Scope refined after research: live server, light theme, team sharing, version comparison in v1 |
