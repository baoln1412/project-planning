# Project Viewer

> **Status**: 🟢 Ready for Development  
> **Created**: 2026-03-08  
> **Last Updated**: 2026-03-08

## Summary

A web application that provides a visual dashboard for browsing and reviewing all projects created in the `project-planning` workspace. It reads project Markdown files, renders them as an interactive UI with cards, drill-down views, version history, and project status tracking.

## Problem Statement

As the number of planned projects grows, navigating raw Markdown files in folders becomes cumbersome. A visual interface makes it easier to:
- Get an at-a-glance overview of all projects and their status
- Drill into any project to read its full plan (research, tech stack, architecture, implementation)
- Compare versions of a project side-by-side
- Track which projects are ready for development vs. still in planning

## Target Users

- The workspace owner (you) — for reviewing and managing project plans
- Collaborators — anyone you share the planning workspace with

## Key Features

- [ ] Dashboard with project cards showing name, status, summary, and last updated date
- [ ] Drill-down view to read all project documents (overview, research, tech stack, architecture, implementation plan, handoff)
- [ ] Version history viewer — see previous versions and what changed (CHANGELOG)
- [ ] Status filtering — filter by planning status (🟡 Planning, 🟢 Ready, 🔵 In Development)
- [ ] Markdown rendering — proper rendering of Markdown including tables and Mermaid diagrams
- [ ] Search — find projects by name or keyword
- [ ] Dark mode — because we're planning at all hours

## Ideas & Notes

- Could be a static site that reads from the `projects/` directory at build time
- Or a lightweight Node.js app that reads files at runtime for live updates
- Mermaid diagram rendering is important since architecture docs use them heavily
- Should feel premium and modern — this is the "control center" for all project planning

## Version History

| Version | Date | Summary |
|---|---|---|
| Initial | 2026-03-08 | Project created |
