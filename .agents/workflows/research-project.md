---
description: Deep-research an existing project idea
---

# /research-project — Research a Project

## Prerequisites
- A project folder must already exist under `projects/<project-slug>/` with at least `overview.md`

## Steps

1. **Identify the project.** If no project slug is provided, list available projects under `projects/` and ask the user to pick one.

2. **Read `overview.md`** to understand the project scope.

3. **Conduct structured research** using the `project-research` skill. Read the skill file first:
   ```
   .agents/skills/project-research/SKILL.md
   ```

4. **Research the following areas** (use web search):
   - **Market Analysis**: Market size, growth trends, demand signals
   - **Competitor Analysis**: Existing solutions, their strengths/weaknesses, pricing
   - **Technical Feasibility**: Available APIs, libraries, infrastructure options
   - **Open Source Alternatives**: Existing open-source projects that solve similar problems
   - **Risk Assessment**: Technical risks, business risks, regulatory concerns

5. **Present findings to the user** and ask:
   - Does this change your vision for the project?
   - Any competitors you want to differentiate from specifically?
   - Any risks that concern you?

6. **Write/update `research.md`** in the project folder with:
   - Executive summary
   - Market analysis with data
   - Competitor comparison table
   - Technical feasibility assessment
   - Risk matrix (likelihood × impact)
   - Sources / references

7. **Suggest next steps**: Run `/generate-plan` to continue with the full planning pipeline.
