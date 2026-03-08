# Handoff: Project Viewer

> This document contains everything needed to start development in a separate workspace.

## Project Overview

A Next.js 14 application that serves as a visual control center for all project planning documents. It allows for quick review, search, and version-aware browsing of planning artifacts created in the `project-planning` workspace.

**Planning Documents**: See the full project folder at `projects/project-viewer/`

---

## Repository Structure

```
project-viewer/
├── src/
│   ├── app/                # Next.js App Router (Dashboard, Details)
│   ├── components/         # Card, Renderer, VersionSwitcher
│   ├── services/           # File system discovery & parsing
│   ├── styles/             # CSS Modules
│   └── utils/              # Fuse.js setup
├── public/
├── .env.example
├── .gitignore
├── README.md
└── package.json
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 14.x |
| Language | TypeScript | 5.x |
| Rendering | react-markdown | 9.x |
| Diagrams | Mermaid.js | 10.x |
| Search | Fuse.js | 7.x |

See [tech-stack.md](./tech-stack.md) for full evaluation and rationale.

---

## Environment Setup

### Prerequisites

-   Node.js 18+
-   npm or yarn

### Setup Commands

```bash
# Initialize project
npx create-next-app@latest project-viewer --typescript --eslint --tailwind --app --src-dir ./src

# Install dependencies
cd project-viewer
npm install react-markdown mermaid gray-matter fuse.js remark-gfm date-fns

# Run development server
npm run dev
```

### Environment Variables

| Variable | Description | Required |
|---|---|---|
| `PROJECTS_DIR` | Absolute path to the /projects folder | Yes |

---

## Architecture Reference

See [architecture.md](./architecture.md) for full diagrams.

**Key Decisions**:
-   **SSR over SSG**: Using server-side rendering allows the app to dynamically discover and read new project folders as soon as they are created.

---

## First Sprint / MVP Scope

### Goals

Build a functional dashboard that reads two existing project folders (`_template` and `project-viewer`) and displays them as cards.

### Features to Build

- [ ] Next.js project setup.
- [ ] Folder-scanning service for `/projects`.
- [ ] Component for rendering Markdown and Mermaid diagrams.

### Acceptance Criteria

- [ ] Dashboard is accessible at `localhost:3000`.
- [ ] Project cards show correct metadata.
- [ ] Clicking a project displays its documentation.
