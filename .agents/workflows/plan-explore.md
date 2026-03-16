---
description: Deep-dive into a specific business domain using bundle skills for any project
---

# /plan-explore — Explore a Domain with Skills & Knowledge

## Steps

### 1. Identify the target project
// turbo
List all projects in `projects/` (excluding `_template`).
Ask which project to explore for, or allow "general exploration" without a project.

### 2. Ask the exploration domain
Present the available domains:

| # | Domain | What It Covers | Example Skills |
|---|--------|---------------|----------------|
| 1 | **Customers** | Personas, journeys, segmentation | `customer-persona`, `customer-journey-map` |
| 2 | **Competitors** | Market analysis, benchmarking | `competitor-analysis`, `benchmarking-report` |
| 3 | **Pricing & Revenue** | Pricing models, unit economics, forecasts | `pricing-strategy`, `unit-economics`, `revenue-forecast` |
| 4 | **Go-to-Market** | Launch, content, social, email, SEO | `social-media-strategy`, `email-sequence`, `seo-strategy` |
| 5 | **Analytics & KPIs** | Metrics, dashboards, A/B testing | `kpi-dashboard`, `ab-test-plan`, `cohort-analysis` |
| 6 | **Operations** | SOPs, workflows, risk, vendors | `sop-builder`, `risk-assessment`, `vendor-evaluation` |
| 7 | **Legal & Compliance** | Contracts, policies, terms | `contract-writer`, `terms-of-service`, `privacy-policy` |
| 8 | **Finance** | Budgets, cash flow, fundraising | `budget-planner`, `cash-flow-forecast`, `financial-model` |
| 9 | **AI & Automation** | AI use cases, automation workflows | `ai-use-case-finder`, `automation-workflow` |
| 10 | **Branding** | Brand architecture, voice, identity | `brand-architecture`, `brand-audit`, `brand-voice` |

Ask: "Which domain do you want to explore? (pick a number or describe it)"

Also check if the user has a **NotebookLM notebook** relevant to this domain:
- Run `list_notebooks` and present any matching notebooks
- Ask: "I also found these notebooks that might be relevant: [list]. Want me to query them too?"

### 3. Map domain to skills and knowledge sources
// turbo
Based on the selected domain, scan the corresponding skill category folders and list all available skills with their descriptions.

Present the skills as a numbered list:
```
Available skills for [Domain]:
1. skill-name — description
2. skill-name — description
...
```

Ask: "Which skills do you want to run? (pick numbers, or 'all')"

### 4. Query NotebookLM (if applicable)
If the user has a relevant notebook:
1. Activate it with `select_notebook`
2. Ask domain-specific questions:
   - "What are the key insights about {{domain}} for our project?"
   - "What best practices or patterns should we follow?"
   - "What pitfalls or risks should we be aware of?"
3. Use Sequential Thinking MCP to synthesize notebook answers with skill outputs

### 5. Run selected skills
For each selected skill:
// turbo
1. Read the SKILL.md file in the skill folder
2. Follow the skill's own phase structure (Brief → Map → Build → Polish)
3. Collect the output

### 6. Append findings to the project
If a project was selected in Step 1:
- Append key findings to `findings.md → Session Notes`
- Add any new constraints to `findings.md → Discovered Constraints`
- Add competitor insights to `findings.md → Competitor Intelligence`
- Log the skill run in `findings.md → Skills Used`

Format:
```markdown
### {{Date}} — Domain Exploration: {{Domain}}
**Skills Used**: {{skill-1}}, {{skill-2}}
- {{key finding 1}}
- {{key finding 2}}
- {{key decision or insight}}
```

### 7. Suggest next steps
Based on the exploration results, suggest:
- Related skills the user might want to run next
- Which planning phase these findings are most relevant to
- Whether any existing template files should be updated with new insights

## Notes

- Skills can be run at any time, not just during the formal planning phases.
- Multiple domains can be explored in sequence within a single session.
- If exploration reveals the project size should change, suggest re-running Phase 0 via `/plan-new`.
