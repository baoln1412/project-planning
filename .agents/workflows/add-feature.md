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

3. **Apply Socratic Refinement** — Do NOT passively accept the feature. Challenge it:
   - **"Why this?"** — "What problem does this solve that the existing plan doesn't?"
   - **"Why now?"** — "Is this truly needed for the next milestone, or is it scope creep?"
   - **"What's the lighter version?"** — "Could we achieve 80% of the value with a simpler approach?"
   - **"What's the cost?"** — "What gets delayed or deprioritized if we add this?"
   - **"What breaks?"** — "Does this introduce new dependencies or architectural changes?"

   Only proceed once the user has justified the feature against these probes.

4. **Research feasibility** (if needed):
   - Use web search to evaluate technical feasibility
   - Check if the current tech stack supports it
   - Identify any new dependencies or infrastructure needed

5. **Create a version snapshot** (same as `/update-project`):
   // turbo
   ```
   # Determine next version number and snapshot
   mkdir -p projects/<project-slug>/v<N>
   cp projects/<project-slug>/*.md projects/<project-slug>/v<N>/
   ```

6. **Write a `CHANGELOG.md` entry** in the version folder noting the feature addition.

7. **Update affected planning documents**:
   - `overview.md` — Add the feature to the features list
   - `architecture.md` — Update diagrams if the feature impacts system design
   - `tech-stack.md` — Add any new technologies required
   - `implementation-plan.md` — Add a new milestone or tasks for the feature
   - `findings.md` — Record the feature decision and rationale

8. **Present the updated plan** to the user:
   - Show what changed in each document
   - Ask if the scope and approach look right
   - Confirm priority/timeline positioning

9. **Suggest next steps**: Continue with more features, or `/export-project` when ready.
