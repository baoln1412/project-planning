---
description: Update or change an existing project with version snapshots
---

# /update-project — Update a Project

## Prerequisites
- A project folder must exist under `projects/<project-slug>/` with planning files

## Steps

1. **Identify the project.** If no project slug is provided, list available projects and ask the user to pick one.

2. **Ask the user what is changing**:
   - Scope change? (features added/removed)
   - Pivot? (different target audience or problem)
   - Technical change? (different tech stack or architecture)
   - Timeline change?

3. **Determine the next version number**:
   - Check for existing version folders (`v1/`, `v2/`, …)
   - The next version = max existing version + 1, or `v1` if none exist

4. **Snapshot current state**:
   // turbo
   ```
   mkdir -p projects/<project-slug>/v<N>
   cp projects/<project-slug>/*.md projects/<project-slug>/v<N>/
   ```

5. **Create `CHANGELOG.md` in the version folder**:
   - Version number
   - Date
   - Summary of what was in this version (before the update)
   - Reason for update

6. **Apply the updates** to the root-level files:
   - Update affected files (`overview.md`, `architecture.md`, `tech-stack.md`, `implementation-plan.md`, etc.)
   - Ask clarifying questions about the changes before modifying each file
   - Keep changes focused — don't rewrite files that aren't affected

7. **Update `overview.md` status and version history section**:
   - Add a "Version History" section if not present
   - Record the update with date and summary

8. **Suggest next steps**: Run `/export-project` if the project is ready for development, or continue refining.
