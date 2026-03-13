---
description: Start planning a new project from idea to development-ready handoff
---

# /plan-new — Start a New Project

## Prerequisites

- The `.agents/skills/project-planning/` folder must exist in the workspace
- The `projects/_template/` folder must contain all template files

## Steps

### 1. Read the SKILL.md orchestrator
// turbo
Read `.agents/skills/project-planning/SKILL.md` to load the full planning flow.

### 2. Check NotebookLM for domain knowledge
Run `list_notebooks` to see if the user has any relevant NotebookLM notebooks.
If a notebook matches the project domain, activate it with `select_notebook`.
This notebook will be queried throughout the planning phases for domain-specific knowledge.

### 3. Ask for the project idea
Ask the user: "Describe your project — what system are you building, what problem does it solve, who uses it, and what does the existing landscape look like?"

### 4. Run Phase 0: Intake & Sizing
Use the Sequential Thinking MCP to analyze the project idea:
- Parse the core system being built
- Identify complexity signals (services, integrations, user types, data stores, compliance)
- Assess existing landscape (greenfield vs. brownfield)
- Match to size category (🟢 Small / 🟡 Medium / 🔴 Large)
Present the classification and ask for confirmation.

### 5. Create the project folder
// turbo
Copy `projects/_template/` to `projects/{{project-slug}}/`
Use a URL-safe slug derived from the project name (lowercase, hyphens, no spaces).

### 6. Walk through applicable phases (Technical Track)
Follow the SKILL.md phase sequence based on the project size:
- 🟢 Small: Phases 1 → 5 → 6 → 7
- 🟡 Medium: Phases 1 → 4 → 5 → 6 → 7
- 🔴 Large: Phases 1 → 2 → 3 → 4 → 5 → 6 → 7

At each phase:
1. Ask the required discovery questions from SKILL.md
2. Fill in the corresponding template file
3. **Use Sequential Thinking MCP** for complex decisions (architecture tradeoffs, tech evaluations, risk assessments)
4. **Query NotebookLM** if a notebook is active — ask domain-specific questions
5. **Auto-run domain skills** — Read and execute the 2–3 most relevant skills for the phase (see SKILL.md skill tables). Append outputs to `findings.md → Skills Used`
6. Log decisions as ADRs in `findings.md`
7. Update `findings.md` with decisions, skill outputs, and insights
8. **GATE**: Present the output and wait for user confirmation before proceeding

### 7. Generate Business Proposal (Phase 8)
After the technical handoff is complete:
1. Read through all completed planning documents
2. Translate technical decisions into business language
3. Fill in `business-proposal.md` with the 11-section structure
4. Present a summary and ask for user confirmation

### 8. Final review
After completing all phases:
1. Set the project status to 🟢 Ready for Development in `overview.md`
2. Present a summary of all completed documents:
   - **Technical Track**: overview, research, tech-stack, architecture, implementation-plan, handoff
   - **Business Track**: business-proposal
3. Suggest running `/plan-export` to convert the business proposal to `.docx`
4. Ask if the user wants to run any additional domain skills for deeper exploration

## Notes

- If the user wants to pause mid-flow, note the current phase in `findings.md` so `/plan-resume` can pick up where they left off.
- If the user's answers suggest a different project size than initially classified, ask if they want to re-classify.
- The Business Proposal (Phase 8) can be re-run independently by re-reading all planning docs and regenerating.
