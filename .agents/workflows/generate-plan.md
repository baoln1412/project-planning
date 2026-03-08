---
description: Generate a full project plan (tech stack, architecture, implementation)
---

# /generate-plan — Full Planning Pipeline

## Prerequisites
- A project folder must exist under `projects/<project-slug>/` with at least `overview.md`
- Ideally `research.md` already exists (if not, run `/research-project` first or do inline research)

## Steps

1. **Identify the project.** If no project slug is provided, list available projects and ask the user to pick one.

2. **Read existing files** (`overview.md`, `research.md` if available) to build context.

3. **Phase 1: Idea Refinement**
   - Refine raw features into concrete, scoped functionality
   - Ask the user:
     - What's the MVP vs. nice-to-have?
     - Any non-negotiable features?
     - Target launch timeline?
   - Update `overview.md` with refined scope

4. **Phase 2: Tech Stack Selection**
   - Use the `tech-stack-advisor` skill. Read the skill file first:
     ```
     .agents/skills/tech-stack-advisor/SKILL.md
     ```
   - Evaluate options for: language, framework, database, hosting, CI/CD, monitoring
   - Present a comparison table to the user and ask for preferences
   - Write `tech-stack.md`

5. **Phase 3: Architecture Design**
   - Use the `scalable-architecture` skill. Read the skill file first:
     ```
     .agents/skills/scalable-architecture/SKILL.md
     ```
   - Design 3-phase architecture:
     - **Test/MVP**: Minimal infra for validation
     - **Production**: Production-grade with security, backups, monitoring
     - **Scale**: Horizontal scaling, caching, CDN, load balancing
   - Include Mermaid diagrams for each phase
   - Ask the user about scale expectations and budget
   - Write `architecture.md`

6. **Phase 4: Implementation Plan**
   - Break the build into milestones (3–6 milestones typical)
   - Each milestone includes:
     - Goal / deliverables
     - Tasks with estimated effort
     - Dependencies
     - Definition of done
   - Ask the user about team size and timeline
   - Write `implementation-plan.md`

7. **Summary**: Present a quick overview of all generated docs and suggest `/export-project` when ready to start development.
