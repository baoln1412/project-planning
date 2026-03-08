# Architecture: Project Viewer

## Overview

The Project Viewer is a server-side rendered (SSR) application designed to read and visualize Markdown project plans from the local filesystem. It uses a **Next.js 14+** architecture to achieve high performance and live data updates.

## Phase 1: Test / MVP (Current Goal)

### Design Goals

-   **Fast Development**: Minimal setup, reading directly from the `projects/` directory.
-   **Live Updates**: Every page load re-reads the filesystem.
-   **No Database**: All project data is stored in existing Markdown files.

### Architecture Diagram

```mermaid
graph TD
    User[👤 User] --> Web[🌐 Next.js App]
    subgraph "Server (Next.js SSR)"
        Page[Next.js Page] --> FS[fs.readdir / fs.readFile]
        FS --> Matter[gray-matter Parser]
        Matter --> Render[react-markdown Renderer]
    end
    Render --> UI[✨ Visual Dashboard UI]
    UI --> User
```

### Components

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Project Discoverer** | Next.js Server Side | Scans `projects/` for `overview.md` files to build the dashboard. |
| **Doc Renderer** | `react-markdown` | Renders project planning documents into readable HTML. |
| **Diagram Engine** | `mermaid.js` | Client-side rendering of architecture and Gantt charts. |

---

## Phase 2: Production (Transition Trigger: 10+ Projects)

### Trigger to Transition

Transition to Phase 2 when the project list grows enough to require better performance (e.g., more than 10-20 projects, making live re-reads slower).

### Architecture Diagram

```mermaid
graph TB
    subgraph "Client"
        Web[🌐 Dashboard App]
    end
    subgraph "Server"
        API[API Routes]
        Cache[(File Cache / KV Store)]
    end
    subgraph "Data"
        Projects[(Local Markdown Files)]
    end
    Web --> API
    API --> Cache
    Cache -- Check Cache --> API
    API -- Read on Cache Miss --> Projects
```

### New Components

-   **Caching Layer**: Using a simple in-memory cache or KV store to avoid redundant file reads.
-   **Advanced Search**: Indexing project content for full-text fuzzy search (via Fuse.js on the client).

---

## Phase 3: Scale (Transition Trigger: Multi-User Access)

### Trigger to Transition

Transition to Phase 3 if the tool needs to be shared with multiple collaborators or accessed remotely via a hosted server.

### Architecture Diagram

```mermaid
graph TB
    subgraph "Edge"
        LB[Load Balancer]
        CDN[CDN]
    end
    subgraph "Compute"
        Next[Next.js Server Cluster]
    end
    subgraph "Storage"
        DB[(PostgreSQL / Supabase)]
        S3[(S3 Object Storage - for project files)]
    end
    LB --> Next
    Next --> DB
    Next --> S3
    CDN --> Next
```

### Scaling Strategy

-   **S3 for Files**: Moving project files to object storage if they become too large for local disk.
-   **Database**: Introducing a database for project metadata and user-specific viewing preferences.

## Data Architecture

### ERD

```mermaid
erDiagram
    PROJECT ||--o{ VERSION : has
    PROJECT ||--|| OVERVIEW : contains
    PROJECT ||--|| TECH_STACK : contains
    PROJECT ||--|| ARCHITECTURE : contains
```

### Key Data Flows

1.  User visits the dashboard.
2.  Next.js Server reads all subdirectories in `projects/`.
3.  Each `overview.md` is parsed for its title, status, and summary.
4.  Data is passed as props to the React frontend and rendered as project cards.
