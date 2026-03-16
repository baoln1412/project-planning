# Claude Skills Ultimate Bundle

A project planning toolkit that turns your ideas into two ready-to-use outputs:

- **A technical handoff** that AI coding tools (Antigravity, Claude Code) can read and start building from
- **A business proposal** (`.docx`) that you can present to stakeholders for approval

Built from the perspective of a **senior solutions architect** — it thinks in systems, data contracts, and architecture decisions, not just feature lists.

---

## What's Inside

- **475 domain skills** across 20 categories (AI, Analytics, Finance, Legal, Marketing, Operations, etc.)
- **8 project templates** that get filled in as you plan (requirements, architecture, tech stack, etc.)
- **4 slash commands** that orchestrate the entire flow

---

## How to Start

### 1. Plan a new project

Type `/plan-new` and describe your project idea. The system will:

1. Classify your project size (Small / Medium / Large)
2. Walk you through the relevant planning phases — asking questions, evaluating options, making architecture decisions
3. Fill in all templates automatically based on your answers
4. Produce a **technical handoff** (for AI-assisted development)
5. Generate a **business proposal** (for stakeholder presentation)

### 2. Export for stakeholders

Type `/plan-export` to convert the business proposal to a `.docx` file you can edit in Word and present to executives.

### 3. Resume later

Type `/plan-resume` to pick up where you left off. All decisions are saved in `findings.md` so nothing is lost between sessions.

### 4. Go deeper on a topic

Type `/plan-explore` to run any of the 475 domain skills for deeper research — competitor analysis, pricing strategy, compliance checklists, and more.

---

## Requirements

- **Antigravity** or **Claude Code** (any AI agent that reads `.agents/skills/`)
- **Pandoc** (for `.docx` export) — install with `winget install JohnMacFarlane.Pandoc`
- **Optional**: NotebookLM MCP for domain knowledge, Sequential Thinking MCP for complex decisions