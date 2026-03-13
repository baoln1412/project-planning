---
name: project-planning
description: "Dual-track project planning from a solutions architect perspective. Produces (1) AI-implementation-ready technical handoffs and (2) business stakeholder proposals. Uses Sequential Thinking for complex decisions and NotebookLM for domain knowledge."
allowed-tools: Read Write Glob
metadata:
  author: project-planning
  version: "4.0"
---

# Project Planning — Solutions Architect Edition

## When to Use This Skill

Use this skill when you need to:
- Architect a technical solution from requirements to implementation-ready handoff
- Produce a development brief that AI coding agents (Antigravity / Claude Code) can use to start building
- Translate a technical plan into a business proposal for stakeholder sign-off
- Evaluate technology choices with formal Architecture Decision Records

**DO NOT** use this skill for code implementation, bug fixing, or ongoing project management. This produces **planning documentation only** — two output tracks:

| Track | Audience | Output | Format |
|---|---|---|---|
| **Technical** | AI coding agents / dev teams | `handoff.md` + supporting docs | Markdown |
| **Business** | Executives / business stakeholders | `business-proposal.md` | Markdown → `.docx` via `/plan-export` |

---

## Core Principles

1. **GATE RULE** — Never proceed to the next phase without explicit user confirmation.
2. **DATA-FIRST RULE** — Define input/output data contracts and NFRs before designing architecture.
3. **FINDINGS RULE** — Update `findings.md` after every phase. This is the persistent memory.
4. **ADR RULE** — Log every significant technical decision as an Architecture Decision Record in `findings.md`.
5. **TEMPLATE RULE** — Always create files from `projects/_template/`, never from scratch.
6. **SCOPE RULE** — Out-of-scope ideas go to `findings.md → Ideas & Future Scope`, not into active work.
7. **THINK-DEEP RULE** — Use Sequential Thinking MCP for any decision with 3+ options, tradeoffs, or multi-step reasoning.
8. **KNOWLEDGE RULE** — Query NotebookLM before research phases if the user has domain knowledge notebooks.
9. **DUAL-TRACK RULE** — Every phase feeds both the technical handoff AND the business proposal. Note which sections each phase contributes to.
10. **ARCHITECT RULE** — Reason from a solutions architecture perspective: think in systems, components, integrations, failure modes, and operational concerns — not just features.
11. **DEEP-RESEARCH RULE** — At each phase, automatically run the most relevant domain skills from the bundle. Do not just recommend them — read their SKILL.md and execute them. Append their outputs to `findings.md → Skills Used` and feed insights into the current phase's template.

---

## Phase 0: Intake & Sizing

Before anything else, classify the project to determine which phases to include.

### Ask the User

> "Describe your project — what system are you building, what problem does it solve, who uses it, and what does the existing landscape look like?"

### 🧠 Use Sequential Thinking

- Thought 1: Parse the description — what is the core system being built?
- Thought 2: Identify complexity signals — how many services, integrations, user types, data stores, compliance domains?
- Thought 3: Assess existing system landscape — greenfield vs. brownfield, migration complexity
- Thought 4: Match to size category and determine phase sequence

### 📚 Check NotebookLM

- Use `list_notebooks` to see available notebooks
- If a notebook matches the project domain, activate it with `select_notebook`
- Query: "What are the key technical requirements and architectural constraints for {{project domain}}?"

### Classify

| Size | Description | Examples | Phases |
|---|---|---|---|
| 🟢 **Small** | Single-purpose tool, script, utility, or integration | CLI tool, data transformer, API wrapper, browser extension | 0 → 1 → 5 → 6 → 7 → 8 |
| 🟡 **Medium** | Multi-component application with defined scope | Web app, API service, internal tool, dashboard, bot | 0 → 1 → 4 → 5 → 6 → 7 → 8 |
| 🔴 **Large** | Multi-service system, platform, or enterprise solution | SaaS platform, marketplace, data platform, multi-tenant system | All phases 0–8 |

### Create the Project Folder

```
projects/{{project-slug}}/
```

Copy all files from `projects/_template/` into the new folder.

**GATE: Confirm project size and folder creation before proceeding.**

---

## Phase 1: Requirements & Discovery

**Output**: `overview.md`
**Feeds Business Proposal**: §1 Executive Summary, §2 Business Objectives, §3 Scope & Boundaries

### Required Inputs

| Input | What to Ask |
|---|---|
| **What** | "What system are you building? Describe it in one sentence." |
| **Why** | "What problem does it solve? What's the business case?" |
| **Who** | "Who are the users and stakeholders? What are their needs?" |
| **North Star** | "What is the singular success metric?" |
| **Existing Systems** | "What existing systems does this interact with or replace?" |
| **Integrations** | "What external services, APIs, or data sources are needed?" |
| **Data Sovereignty** | "Where must data reside? Any compliance requirements (GDPR, HIPAA, SOC2)?" |
| **NFRs** | "What are the performance, availability, scalability, and security requirements?" |
| **Constraints** | "Any technology constraints, team skill limitations, or timeline pressures?" |
| **Scope Boundaries** | "What is this project explicitly NOT?" |

### Fill `overview.md`

Populate the template including:
- Non-Functional Requirements table (performance, availability, scalability, security targets)
- System Context Diagram (C4 Level 1 — the system in its environment)
- Integration Inventory (external systems, protocols, auth methods, SLAs)
- Stakeholder Map (RACI for technical and business stakeholders)

### 🔍 Run Domain Skills (Auto)

Based on the project domain, automatically run the 2–3 most relevant skills. Read each skill's SKILL.md and execute it. Feed the outputs into `overview.md` and `findings.md`.

| Project Type | Skills to Run |
|---|---|
| **API / Platform** | `api-documentation`, `tech-stack-recommendation`, `tool-stack-audit` |
| **Data system** | `data-collection-plan`, `analytics-setup-guide`, `data-dashboard-design` |
| **Internal tool** | `sop-builder`, `process-automation-audit`, `workflow-mapper` |
| **B2B / SaaS** | `customer-persona`, `customer-segmentation`, `saas-evaluation` |
| **Compliance-heavy** | `compliance-checklist`, `gdpr-compliance-checklist`, `privacy-policy` |

For each skill run: read `.agents/skills/{{category}}/{{skill-name}}/SKILL.md`, execute it, and append key findings to `findings.md → Skills Used`.

### Update `findings.md`

Log discovery answers, constraints, NFRs, skill outputs, and any early ADRs.

**GATE: User confirms `overview.md` before proceeding.**

---

## Phase 2: Feasibility & Research (🔴 Large projects only)

**Output**: `research.md`
**Feeds Business Proposal**: §4 Solution Overview, §8 Dependencies & Assumptions

### 📚 Query NotebookLM First

- "What do we know about the {{project domain}} technical landscape, existing solutions, and common pitfalls?"

### Research Scope

| Section | What to Research |
|---|---|
| **Technical Landscape** | Existing solutions, open-source alternatives, build-vs-buy candidates |
| **API & Integration Feasibility** | Available APIs, rate limits, auth flows, data format compatibility, versioning |
| **Infrastructure Options** | Cloud providers, managed services, serverless vs. containers, edge computing |
| **Compliance & Licensing** | Open-source license compatibility, regulatory requirements, data residency |
| **PoC Recommendations** | What to prototype, expected outcomes, success criteria for spikes |
| **Risk Assessment** | Technical, operational, and integration risks with likelihood and impact |
| **Key Insights** | 3–5 actionable findings that shape the architecture |

### 🧠 Use Sequential Thinking for Risk Assessment

- Thought 1: List all technical risks (integration failures, scalability limits, vendor lock-in)
- Thought 2: List all operational risks (monitoring gaps, deployment complexity, team skill gaps)
- Thought 3: Assess likelihood × impact, prioritize
- Thought 4: Propose mitigations, identify risks that need PoC validation
- Thought 5: Synthesize into architectural constraints

### 🔍 Run Domain Skills (Auto)

Automatically run research-relevant skills:

| Project Type | Skills to Run |
|---|---|
| **Any project** | `competitor-analysis`, `benchmarking-report` |
| **Consumer-facing** | `conversion-funnel-analysis`, `sentiment-analysis` |
| **Marketplace** | `marketplace-metrics`, `marketplace-fee-structure` |
| **Subscription** | `saas-metrics-dashboard`, `churn-analysis`, `customer-lifetime-value` |

### Update `findings.md`

Log research findings, skill outputs, risk items, PoC recommendations, and any new ADRs.

**GATE: User reviews research findings before proceeding.**

---

## Phase 3: Solution Strategy (🔴 Large projects only)

**Output**: Updates to `findings.md` (Key Decisions section)
**Feeds Business Proposal**: §2 Business Objectives, §5 Key Milestones

### Strategy Questions

| Question | Purpose |
|---|---|
| "Build, buy, or extend? For each major component." | Build-vs-buy analysis |
| "What is the phasing strategy? MVP → Production → Scale." | Release strategy |
| "What is the migration path from existing systems?" | Migration planning |
| "How will this make money (or save money)?" | Business model |
| "What are the KPIs and SLAs?" | Success metrics |

### 🧠 Use Sequential Thinking for Strategy

- Thought 1: For each component, evaluate build vs. buy vs. extend with cost/risk/timeline tradeoffs
- Thought 2: Define the phasing strategy — what ships first, what scales later
- Thought 3: Design the migration path — cutover vs. strangler fig vs. parallel run
- Thought 4: Connect technical decisions to business outcomes (cost savings, revenue, time-to-market)
- Thought 5: Define success metrics that satisfy both technical SLAs and business KPIs

### 🔍 Run Domain Skills (Auto)

Automatically run strategy-relevant skills:

| Focus Area | Skills to Run |
|---|---|
| **Revenue** | `pricing-strategy`, `unit-economics`, `revenue-forecast` |
| **Growth** | `social-media-strategy`, `content-strategy`, `seo-strategy` |
| **Operations** | `business-plan`, `annual-planning`, `budget-planner` |
| **Legal** | `terms-of-service`, `privacy-policy`, `contract-writer` |

### Update `findings.md`

Log all strategic decisions as ADRs with rationale. Append skill outputs.

**GATE: User confirms strategy decisions before proceeding.**

---

## Phase 4: Technology Evaluation (🟡 Medium + 🔴 Large)

**Output**: `tech-stack.md`
**Feeds Business Proposal**: §6 Resource Requirements

### Evaluation Process

For each technology layer (frontend, backend, database, messaging, hosting, observability, etc.):

1. Identify 2–3 candidates
2. Score on: Maturity, Community, Scalability, DX, Performance, Cost, Ecosystem Fit, Operational Complexity
3. Assess team capability fit — existing skills vs. learning curve
4. Write an ADR for each decision (Status, Context, Decision, Consequences)
5. Document alternatives considered and why rejected

### Compatibility Matrix

| | {{Tech A}} | {{Tech B}} | {{Tech C}} |
|---|---|---|---|
| {{Tech A}} | — | ✅ Native | ⚠️ Adapter |
| {{Tech B}} | ✅ Native | — | ✅ Native |

### Cost Estimation

| Tier | Monthly Cost | Annual Cost |
|---|---|---|
| Development | $X | $X |
| Production | $X | $X |
| Scale (10x) | $X | $X |

### 🔍 Run Domain Skills (Auto)

Automatically run tech-evaluation-relevant skills:

| Focus Area | Skills to Run |
|---|---|
| **Automation-heavy** | `automation-workflow`, `process-automation-audit` |
| **AI-powered** | `ai-use-case-finder`, `ai-content-policy`, `ai-ethics-policy` |
| **Data-heavy** | `data-collection-plan`, `analytics-setup-guide`, `data-dashboard-design` |

### Update `findings.md`

Log tech decisions as ADRs. Update NFR tracker. Append skill outputs.

**GATE: User approves technology choices before proceeding.**

---

## Phase 5: Solution Architecture (All sizes)

**Output**: `architecture.md`
**Feeds Business Proposal**: §4 Solution Overview

### ⚠️ Data-First Rule

**Before designing architecture, define:**

1. What data goes IN? (Input schemas — TypeScript interfaces or JSON)
2. What data comes OUT? (Output schemas)
3. What data is STORED? (Database entities / ERD)
4. What are the NFR targets? (From `overview.md`)

Present and confirm before proceeding.

### 🧠 Use Sequential Thinking for Architecture Design

- Thought 1: Review all constraints from `findings.md`, tech stack from Phase 4, and NFRs from Phase 1
- Thought 2: Design the C4 System Context (Level 1) — the system in its environment
- Thought 3: Design the Container Diagram (Level 2) — major deployable units
- Thought 4: Design Component Diagrams (Level 3) — internal structure of key containers
- Thought 5: Design Sequence Diagrams — key user flows and system interactions
- Thought 6: Design Security Architecture — AuthN/AuthZ, encryption, API security, threat model
- Thought 7: Design Observability Strategy — logging, metrics, tracing, alerting, SLOs
- Thought 8: Design Deployment Topology — environments, IaC approach, scaling strategy
- Thought 9: Verify architecture satisfies all NFRs and constraints
- Thought 10: Estimate costs for each deployment phase

### Architecture Scope by Size

| Size | Scope |
|---|---|
| 🟢 Small | Container diagram, key data flows, basic deployment |
| 🟡 Medium | MVP + Production architecture, security, observability |
| 🔴 Large | MVP + Production + Scale architecture, full C4, security review, DR plan |

### 🔍 Run Domain Skills (Auto)

Automatically run architecture-relevant skills:

| Focus Area | Skills to Run |
|---|---|
| **Data pipeline** | `data-collection-plan`, `batch-processing-system` |
| **Analytics** | `analytics-setup-guide`, `kpi-dashboard`, `metric-definition-guide` |
| **E-commerce** | `inventory-management`, `fulfillment-sop` |
| **Security** | `compliance-checklist`, `data-processing-agreement` |

### Update `findings.md`

Log architecture decisions as ADRs. Update NFR tracker. Append skill outputs.

**GATE: User approves architecture before proceeding.**

---

## Phase 6: Implementation Planning (All sizes)

**Output**: `implementation-plan.md`
**Feeds Business Proposal**: §5 Key Milestones, §6 Resource Requirements

### Milestone Breakdown

For each milestone:
1. Name, goal, and estimated duration
2. Task table (task, effort, dependencies, acceptance criteria, status)
3. Definition of Done (checklist)

### Dependency Graph

Generate a Mermaid diagram showing milestone and task dependencies.

### Testing Strategy

| Level | Scope | Tools | Automation |
|---|---|---|---|
| Unit | Individual functions/methods | {{tool}} | CI |
| Integration | Service interactions, API contracts | {{tool}} | CI |
| E2E | Critical user flows | {{tool}} | Staging |
| Performance | Load, stress, soak testing | {{tool}} | Pre-release |
| Security | SAST, DAST, dependency scanning | {{tool}} | CI |

### DevOps Pipeline

Design the CI/CD pipeline:
- Build → Test → Scan → Deploy → Smoke Test → Promote
- Environment promotion: dev → staging → production
- Release strategy: blue/green, canary, or rolling

### Technical Debt Register

| Shortcut | Reason | Remediation | Priority |
|---|---|---|---|
| {{shortcut taken in MVP}} | {{why}} | {{how to fix later}} | {{P1/P2/P3}} |

### 🔍 Run Domain Skills (Auto)

Automatically run implementation-planning-relevant skills:

| Focus Area | Skills to Run |
|---|---|
| **Project management** | `scope-of-work`, `risk-assessment`, `project-tracker` |
| **Quality** | `quality-assurance-checklist`, `retrospective` |
| **Team** | `delegation-framework`, `meeting-agenda` |

### Update `findings.md`

Log assumptions, risks, open questions. Append skill outputs.

**GATE: User confirms implementation plan before proceeding.**

---

## Phase 7: Technical Handoff (All sizes)

**Output**: `handoff.md`

This is the **AI Implementation Brief** — the single document an AI coding agent reads to start building.

### Consolidation

Write `handoff.md` as a dense, implementation-ready reference:

1. **Project Context** — 1-paragraph summary (what, why, who)
2. **Repository Scaffold** — Exact folder structure to create
3. **Tech Stack** — What to install, versions, package managers
4. **Data Contracts** — Input/output schemas (TypeScript interfaces or JSON)
5. **Database Schema** — Entity definitions, relationships, migrations
6. **Architecture Reference** — Key patterns, component responsibilities, interaction rules
7. **API Design** — Endpoints, methods, request/response shapes, auth
8. **Implementation Sequence** — Ordered list of what to build first → last, with dependencies
9. **Testing Requirements** — What tests to write, coverage targets, test data strategy
10. **Environment Configuration** — Env vars, external services, secrets management
11. **Coding Conventions** — Naming, file structure, patterns, error handling standards
12. **Out of Scope** — What NOT to build (prevent AI from over-building)
13. **References** — Links to all planning documents

### Final `findings.md` Update

Mark all open questions as resolved or flag remaining blockers.

**GATE: User approves handoff document.**

---

## Phase 8: Business Proposal (All sizes)

**Output**: `business-proposal.md`

Translate all technical work into a business-facing proposal. Use business language — no technical jargon unless explained. This document is for executives and business stakeholders.

### Synthesis Process

Read through all completed planning documents and synthesize into the 11-section structure:

| § | Section | Source |
|---|---|---|
| 1 | Executive Summary | `overview.md` — 1 paragraph of what, why, expected outcome |
| 2 | Business Objectives & Success Criteria | `overview.md` NFRs → business KPIs, `findings.md` strategy decisions |
| 3 | Scope & Boundaries | `overview.md` scope boundaries, features list |
| 4 | Solution Overview | `architecture.md` → conceptual diagram (NOT technical), business process fit |
| 5 | Key Milestones & Timeline | `implementation-plan.md` → high-level roadmap, value delivery points |
| 6 | Resource Requirements | `tech-stack.md` costs, `implementation-plan.md` effort, team/budget/tools |
| 7 | Risks & Mitigations | `findings.md` risks → reframed as business impact (revenue, reputation, compliance) |
| 8 | Dependencies & Assumptions | `findings.md` + `research.md` → external systems, vendor decisions, regulatory |
| 9 | Change Management & Training | Inferred from `overview.md` users + `architecture.md` new capabilities |
| 10 | Governance & Decision Points | Planning gates → business approval points, reporting cadence |
| 11 | Next Steps | Concrete actions: approvals, resources, decisions with deadlines |

### Diagrams

Replace technical diagrams with conceptual business-friendly versions:
- **System Context** → "How the solution fits into our business"
- **Timeline** → "When value is delivered" (Gantt with business milestones only)
- **Cost** → "Investment breakdown" (table, not architecture diagram)

### Export to docx

After completing `business-proposal.md`, suggest running `/plan-export` to generate a `.docx` file for editing and presentation.

**GATE: User gives final approval. Project status changes to 🟢 Ready for Development.**

---

## Skill Discovery Reference

When recommending skills at each phase, match to the project's technical domain:

| Project Type | Primary Categories | Key Skills |
|---|---|---|
| **API / Platform** | AI & Technology, Industry-Specific | `api-documentation`, `tech-stack-recommendation`, `saas-evaluation` |
| **Data Pipeline** | Analytics & Data, Operations | `data-collection-plan`, `batch-processing-system`, `analytics-setup-guide` |
| **Web Application** | AI & Technology, SEO & Search | `site-architecture-plan`, `technical-seo-checklist`, `schema-markup-guide` |
| **SaaS Product** | Analytics & Data, Finance & Pricing | `saas-metrics-dashboard`, `pricing-strategy`, `churn-analysis` |
| **Marketplace** | E-commerce & Products, Legal & Compliance | `marketplace-metrics`, `marketplace-fee-structure`, `terms-of-service` |
| **Internal Tool** | Operations & Systems, AI & Technology | `sop-builder`, `workflow-mapper`, `process-automation-audit` |
| **Event-Driven System** | Operations & Systems, Analytics | `batch-processing-system`, `report-automation`, `quality-assurance-checklist` |
| **Migration / Modernization** | Operations & Systems, AI & Technology | `tool-stack-audit`, `risk-assessment`, `change-management-plan` |

To find specific skills:
```
# List all skills in a category
ls ".agents/skills/{{category-name}}/"

# Read a skill's description
head -10 ".agents/skills/{{category-name}}/{{skill-name}}/SKILL.md"
```

---

## MCP Tools Reference

### Sequential Thinking MCP

**When to use**: Any decision point with 3+ options or multi-step reasoning.

| Phase | Use Case |
|---|---|
| Phase 0 | Analyze complexity signals, existing landscape, sizing |
| Phase 2 | Evaluate risks across technical, operational, integration categories |
| Phase 3 | Build-vs-buy analysis, phasing strategy, migration path |
| Phase 4 | Compare technology candidates with ADR-format evaluation |
| Phase 5 | Design architecture with C4 model, tradeoff analysis, NFR verification |
| Phase 6 | Dependency analysis, effort estimation, DevOps pipeline design |
| Phase 8 | Translate technical decisions into business impact language |

### NotebookLM MCP

**When to use**: Before any research or analysis phase.

| Phase | Example Query |
|---|---|
| Research | "What do we know about {{domain}} technical landscape and common pitfalls?" |
| Strategy | "What build-vs-buy decisions have worked for {{project type}}?" |
| Architecture | "What architecture patterns handle {{NFR requirement}} at scale?" |
| Tech Stack | "What are the operational tradeoffs of {{technology}} vs {{technology}}?" |

---

## Anti-Patterns

- **Feature-first thinking** — Design systems, not feature lists. Features emerge from good architecture.
- **Skipping NFRs** — Non-functional requirements shape architecture more than features do. Define them early.
- **Premature optimization** — Design for Phase 1 (MVP), document scaling path for Phase 2/3, but don't build it yet.
- **Over-planning small projects** — A CLI tool doesn't need a C4 diagram. Use the sizing system.
- **Technical jargon in business proposal** — The business document must be readable by non-technical stakeholders.
- **Ignoring operational concerns** — Deployment, monitoring, and incident response are architecture concerns, not afterthoughts.
- **Analysis paralysis** — Time-box research. A good-enough decision now beats a perfect decision next month.

---

## Recovery

- **Lost context between sessions** → Read `findings.md` — it has all decisions and ADRs.
- **Stuck on a phase** → Skip it, mark as incomplete in `findings.md`, continue. Circle back later.
- **User changed a decision** → Update the ADR in `findings.md` with new status and rationale. Propagate to affected documents.
- **Project scope grew** → Re-run Phase 0. A Small project may now be Medium.
- **Need to regenerate business proposal** → Re-run Phase 8 only. It reads from all other documents.
