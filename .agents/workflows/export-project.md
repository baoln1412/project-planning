---
description: Export a project for handoff to a development workspace
---

# /export-project — Export for Development

## Prerequisites
- A project folder must exist under `projects/<project-slug>/` with all planning files completed:
  - `overview.md`, `research.md`, `findings.md`, `tech-stack.md`, `architecture.md`, `implementation-plan.md`

## Steps

1. **Identify the project.** If no project slug is provided, list available projects and ask the user to pick one.

2. **Read all planning documents** (including `findings.md`) to build comprehensive context.

3. **Verify completeness**. Check that all 6 planning documents exist and have substantive content. If any are missing or thin, warn the user and suggest running the relevant workflow first.

4. **⛔ DATA-FIRST GATE (MANDATORY)** — Before generating `handoff.md`, enforce the Data-First Rule:

   - **Demand JSON schemas** — Ask the user to define the exact input and output data shapes for all core data flows. For example:
     ```
     "What does the main API request payload look like?"
     "What does the response payload look like?"
     "What does a [core entity] object look like in the database?"
     ```
   - **Present schemas for confirmation** — Format them as JSON and ask the user to approve or modify.
   - **🚫 BLOCK HANDOFF** — Do NOT proceed to generate `handoff.md` until the user has confirmed the data schemas.

5. **Generate `handoff.md`** with the following sections:

   ### Project Overview
   - 1-paragraph summary of what's being built
   - Link back to `overview.md`

   ### Repository Structure
   - Recommended directory tree for the development repo
   - Explain the purpose of each top-level directory

   ### Tech Stack
   - Exact technologies with **specific versions**
   - Link back to `tech-stack.md`

   ### Data Contracts ← NEW (Data-First Rule)
   - JSON input schemas for all core endpoints/flows
   - JSON output schemas for all core endpoints/flows
   - Database entity schemas
   - Any data transformation rules

   ### Environment Setup
   - Step-by-step setup commands (clone, install, configure)
   - Required environment variables (with descriptions, not values)
   - Required external services and how to set them up

   ### Architecture Reference
   - Key architectural decisions and constraints
   - Link back to `architecture.md` for full diagrams

   ### First Sprint / MVP Scope
   - Exactly what to build first
   - Acceptance criteria
   - Link back to `implementation-plan.md`

   ### CI/CD Recommendations
   - Suggested pipeline (build → test → deploy)
   - Recommended tools

   ### References
   - Links to all planning documents in this project folder

6. **Update `overview.md` status** to `🟢 Ready for Development`.

7. **Present the handoff document** to the user for final review.

8. **Instruct the user**: This `handoff.md` can now be referenced from another Antigravity workspace to begin development. Point the dev workspace to this file path.
