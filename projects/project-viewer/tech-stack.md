# Tech Stack: Project Viewer

## Summary

The Project Viewer is built as a **Next.js 14+** application using the **App Router** and **Server-Side Rendering (SSR)**. This choice allows the app to read Markdown files directly from the local filesystem at runtime, ensuring the dashboard is always up-to-date without requiring a rebuild.

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | Type safety for project data and versioning logic. |
| Frontend | Next.js (App Router) | 14.x | SSR for live file reading; efficient routing. |
| Backend | Next.js Server Components | — | Direct `fs` access within components. |
| Styling | Vanilla CSS (Modules) | — | Maximum design flexibility for premium aesthetic. |
| Parsing | gray-matter | 4.x | Parsing YAML frontmatter from project files. |
| Markdown | react-markdown | 9.x | Flexible React component for rendering MD. |
| Diagrams | Mermaid.js | 10.x | Support for architecture and flowchart diagrams. |
| Search | Fuse.js | 7.x | Client-side fuzzy search for fast project lookup. |

## Detailed Evaluation

### Framework Selection

| Criterion | Next.js | Astro | Remix |
|---|---|---|---|
| SSR for Live Reading | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Developer Experience | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Total** | **4.7** | **4.2** | **4.1** |

**Recommendation**: **Next.js** — It provides the best balance of SSR performance (crucial for reading local files live) and developer experience for a prototype/demo project.

## Cost Estimate

| Service | Free Tier | Production Est. | Scale Est. |
|---|---|---|---|
| Hosting | Vercel (Hobby) | $0 (Local/Hobby) | $20/mo (Pro) |
| Database | N/A (Filesystem) | $0 | $0 |
| Storage | Local Disk | $0 | $0 |

## Alternatives Considered

- **Astro Starlight**: Great for docs, but less flexible for a custom industrial dashboard UI and requires a build step for file changes to reflect.
- **Vite + React**: Requires a separate backend (Express/FastAPI) to access the filesystem, which adds unnecessary complexity for a demo.
