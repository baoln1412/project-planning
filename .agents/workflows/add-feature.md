---
description: Add a new feature to an existing project
---

# /add-feature — Add a Feature

## Prerequisites
- A project folder must exist under `projects/<project-slug>/` with planning files
- Ideally the project has gone through `/generate-plan` already

## Steps

1. **Identify the project.** If no project slug is provided, list available projects and ask the user to pick one.

2. **Ask the user about the new feature**:
   - What does the feature do?
   - Why is it needed? (user request, market demand, technical necessity)
   - Is it MVP or post-launch?
   - Any constraints or preferences?

3. **Research feasibility** (if needed):
   - Use web search to evaluate technical feasibility
   - Check if the current tech stack supports it
   - Identify any new dependencies or infrastructure needed

4. **Create a version snapshot** (same as `/update-project`):
   // turbo
   ```
   # Determine next version number and snapshot
   mkdir -p projects/<project-slug>/v<N>
   cp projects/<project-slug>/*.md projects/<project-slug>/v<N>/
   ```

5. **Write a `CHANGELOG.md` entry** in the version folder noting the feature addition.

6. **Update affected planning documents**:
   - `overview.md` — Add the feature to the features list
   - `architecture.md` — Update diagrams if the feature impacts system design
   - `tech-stack.md` — Add any new technologies required
   - `implementation-plan.md` — Add a new milestone or tasks for the feature

7. **Present the updated plan** to the user:
   - Show what changed in each document
   - Ask if the scope and approach look right
   - Confirm priority/timeline positioning

8. **Suggest next steps**: Continue with more features, or `/export-project` when ready.
