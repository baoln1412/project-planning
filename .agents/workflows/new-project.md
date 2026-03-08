---
description: Create a new project from a rough idea
---

# /new-project — Create a New Project

## Prerequisites
- The user has a project idea (even a rough one)

## Steps

1. **Ask the 5 Discovery Questions.** These are **mandatory** before proceeding. Do not skip any:

   | # | Question | Purpose |
   |---|---|---|
   | 🎯 | **North Star**: What is the singular desired outcome? | Defines the project's success metric |
   | 🔌 | **Integrations**: Which external services or APIs do we need? | Maps the dependency landscape |
   | 💾 | **Source of Truth**: Where does the primary data live? | Establishes the data architecture foundation |
   | 📦 | **Delivery Payload**: How and where should the final result be delivered? | Defines the output channel (web, API, mobile, etc.) |
   | 🚫 | **Behavioral Rules**: Are there specific constraints or "Do Not" rules? | Sets hard boundaries for the system |

   Wait for the user to answer **all 5** before proceeding.

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
   - **Discovery Answers** (record all 5 answers here)
   - **Key Ideas / Features** (refined from user input)
   - **Status**: `🟡 Planning`
   - **Created**: current date

5. **Apply Socratic Refinement** — Before finalizing `overview.md`, challenge the user's assumptions:
   - "What evidence do you have that users need this?"
   - "If you could only ship ONE feature, which would it be?"
   - "Have you considered [lighter alternative]?"

6. **Ask follow-up questions** to clarify scope and direction before suggesting next steps.

7. **Suggest next steps**: Run `/research-project` to research the idea, or `/generate-plan` to run the full pipeline.
