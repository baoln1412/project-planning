# 🧠 Project Planning Workspace

> **An AI-powered workspace for planning any project idea — from rough concept to production-ready implementation plan.**

This workspace helps you think through new project ideas logically and produce structured planning documents that can be handed off to a separate development workspace (e.g., another Antigravity workspace) for actual coding.

---

## System Prompt / How This Workspace Works

This workspace is designed to be used with an AI coding assistant (Antigravity). When you start a new conversation in this workspace, the AI acts as a **senior technical project planner and solutions architect**. It will:

1. **Listen to your raw idea** — you describe the project summary and your initial thoughts
2. **Ask clarifying questions** — the AI engages you in a structured dialogue at every planning phase. You are the decision-maker; the AI is your thinking partner
3. **Research the landscape** — market analysis, competitor review, technical feasibility, risk assessment
4. **Refine your idea** — polish raw concepts into concrete features with clear scope (MVP vs. nice-to-have)
5. **Design scalable architecture** — three-phase system design: Test/MVP → Production → Scale, with diagrams
6. **Recommend a tech stack** — scored comparison tables with rationale for every technology choice
7. **Create an implementation plan** — milestones, tasks, dependencies, timeline, and Gantt charts
8. **Export a handoff document** — everything a development workspace needs to start building

### Key Principles

- **You are always involved.** The AI asks questions at every phase before proceeding.
- **Each project is versioned.** When you update or change a project, the previous state is preserved in versioned folders (`v1/`, `v2/`, …).
- **Output is Markdown only.** No code files — just structured documentation, tables, and Mermaid diagrams.
- **Handoff-ready.** Every project produces a `handoff.md` that another Antigravity workspace can consume to start development.

---

## Folder Structure

```
project-planning/
├── .gemini/
│   └── rules.md                    # Workspace behavior rules
├── .agents/
│   ├── workflows/                  # Slash-command workflows
│   │   ├── new-project.md          # /new-project
│   │   ├── research-project.md     # /research-project
│   │   ├── generate-plan.md        # /generate-plan
│   │   ├── update-project.md       # /update-project
│   │   ├── add-feature.md          # /add-feature
│   │   └── export-project.md       # /export-project
│   └── skills/                     # Reusable AI skills
│       ├── project-research/       # Market & feasibility research
│       ├── tech-stack-advisor/     # Technology evaluation
│       └── scalable-architecture/  # 3-phase architecture design
├── projects/
│   ├── _template/                  # Template files for new projects
│   └── <project-slug>/            # One folder per project
│       ├── overview.md
│       ├── research.md
│       ├── tech-stack.md
│       ├── architecture.md
│       ├── implementation-plan.md
│       ├── handoff.md
│       ├── v1/                    # Version snapshots
│       └── v2/
└── README.md                      # This file
```

---

## Available Workflows

Use these slash commands to drive the planning process:

| Command | Description |
|---|---|
| `/new-project` | Create a new project from a rough idea |
| `/research-project` | Deep-research a project (market, competitors, feasibility) |
| `/generate-plan` | Run the full planning pipeline (tech stack → architecture → implementation) |
| `/update-project` | Update/change a project with automatic version snapshot |
| `/add-feature` | Add a new feature to an existing project |
| `/export-project` | Generate the handoff document for a dev workspace |

### Typical Flow

```
/new-project → /research-project → /generate-plan → /export-project
```

For updates after initial planning:
```
/update-project  or  /add-feature  →  /export-project
```

---

## Available Skills

Skills are specialized capabilities the AI can use during workflows:

| Skill | Purpose |
|---|---|
| **Project Research** | Structured web research with market analysis, competitor tables, risk matrix |
| **Tech Stack Advisor** | Technology evaluation with scoring matrices and cost estimates |
| **Scalable Architecture** | 3-phase architecture design (MVP → Prod → Scale) with Mermaid diagrams |

---

## Getting Started

1. **Start a conversation** in this workspace
2. **Describe your idea**: _"I want to build a [thing] that [does what] for [whom]"_
3. **Run `/new-project`** — the AI will ask you questions and create the project folder
4. **Run `/generate-plan`** — walk through the full planning pipeline interactively
5. **Run `/export-project`** — get a handoff document ready for development

---

## Connecting to a Development Workspace

Once you've run `/export-project`, the generated `handoff.md` contains everything needed to start coding:

1. Open your **development** Antigravity workspace
2. Reference the handoff file: `projects/<project-slug>/handoff.md`
3. The dev workspace can read the project plan and begin implementation

---

## Projects

| Project | Status | Description |
|---|---|---|
| `_template` | — | Template files for new projects |
| *(your projects will appear here)* | | |