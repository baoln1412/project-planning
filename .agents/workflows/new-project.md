---
description: Create a new project from a rough idea
---

# /new-project — Create a New Project

## Prerequisites
- The user has a project idea (even a rough one)

## Steps

1. **Ask the user** to describe their project idea. Prompt them with:
   - What problem does this solve?
   - Who is the target audience?
   - Any initial feature ideas or technical preferences?

2. **Generate a project slug** from the project name (lowercase, hyphens, no spaces). Confirm with the user.

3. **Create the project folder and `overview.md`**:
   // turbo
   ```
   mkdir -p projects/<project-slug>
   ```

4. **Write `overview.md`** with the following sections:
   - **Project Name**
   - **Summary** (polished version of user's description)
   - **Problem Statement**
   - **Target Users**
   - **Key Ideas / Features** (refined from user input)
   - **Status**: `🟡 Planning`
   - **Created**: current date

5. **Ask follow-up questions** to clarify scope and direction before suggesting next steps.

6. **Suggest next steps**: Run `/research-project` to research the idea, or `/generate-plan` to run the full pipeline.
