---
description: Resume planning on a partially completed project
---

# /plan-resume — Resume Project Planning

## Steps

### 1. Scan for existing projects
// turbo
List all folders in `projects/` (excluding `_template`).
For each project, read its `overview.md` to extract the name and status.

### 2. Present the project list
Show a table:

| # | Project | Status | Last Updated |
|---|---------|--------|-------------|

If there's only one project, auto-select it. Otherwise ask the user which project to resume.

### 3. Restore context from findings.md
// turbo
Read the selected project's `findings.md` to restore:
- All previous decisions and ADRs
- Discovered constraints and NFR status
- Open questions
- Session notes (which phases were completed)

### 4. Detect the current phase
Check which template files are populated vs. still containing `{{placeholders}}`:

| File | Status |
|------|--------|
| `overview.md` | ✅ Complete / ⬜ Template |
| `research.md` | ✅ Complete / ⬜ Template / ⏭️ Skipped |
| `tech-stack.md` | ✅ Complete / ⬜ Template / ⏭️ Skipped |
| `architecture.md` | ✅ Complete / ⬜ Template |
| `implementation-plan.md` | ✅ Complete / ⬜ Template |
| `handoff.md` | ✅ Complete / ⬜ Template |
| `business-proposal.md` | ✅ Complete / ⬜ Template |

### 5. Resume from the next incomplete phase
// turbo
Read `.agents/skills/project-planning/SKILL.md` for the phase instructions.

Present to the user:
- "You left off at Phase X: {{phase name}}"
- Summary of what's been completed so far
- "Ready to continue?"

### 6. Continue the planning flow
Follow the SKILL.md phase sequence from the identified phase.
Apply all gates, ADR logging, and skill recommendations as normal.

If all technical phases (1–7) are complete but business-proposal.md is still a template, proceed directly to Phase 8 (Business Proposal).

## Notes

- If `findings.md` has no session notes, ask the user what they remember about where they left off.
- If the user wants to revise a completed phase, allow it — update the template file and log the revision in `findings.md → Key Decisions`.
- If only the business proposal needs regeneration, skip directly to Phase 8.
