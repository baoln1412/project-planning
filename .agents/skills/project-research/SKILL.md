---
name: Project Research
description: Conduct structured web research for a project including market analysis, competitor analysis, technical feasibility, and risk assessment
---

# Project Research Skill

## Purpose

Perform comprehensive, structured research on a project idea to produce a well-sourced `research.md` document.

## When to Use

- During the `/research-project` workflow
- During the research phase of `/generate-plan`
- Anytime the user wants to validate a project idea

## Research Framework

### 1. Market Analysis

Use web search to gather:
- **Market size**: TAM, SAM, SOM if available
- **Growth trends**: Is the market growing? YoY growth?
- **Demand signals**: Are people searching for this? Forum discussions? Social media mentions?
- **Target demographics**: Who would use this and why?

Search queries to try:
- `"<problem domain>" market size <current year>`
- `"<problem domain>" industry trends`
- `"<product type>" user demographics`

### 2. Competitor Analysis

Find and evaluate existing solutions:
- **Direct competitors**: Products solving the same problem
- **Indirect competitors**: Alternative approaches to the same problem
- **Open-source alternatives**: Free tools or libraries that exist

For each competitor, document:
| Field | Details |
|---|---|
| Name | Product name |
| URL | Website |
| Pricing | Free/paid/freemium |
| Strengths | What they do well |
| Weaknesses | Gaps or complaints |
| Tech Stack | (if discoverable) |
| Users | Size of user base (if known) |

Search queries to try:
- `"<product type>" alternatives`
- `"<product type>" vs`
- `"<product type>" comparison <current year>`
- `"<product type>" open source`

### 3. Technical Feasibility

Assess whether the project can be built:
- **APIs available**: Are there APIs for required data/services?
- **Libraries & frameworks**: Existing tools that accelerate development
- **Infrastructure needs**: What hosting/services are required?
- **Data requirements**: What data is needed and how to get it?
- **Integration complexity**: How hard are third-party integrations?

Search queries to try:
- `"<feature>" API`
- `"<tech need>" library <language>`
- `"<service type>" pricing comparison`

### 4. Risk Assessment

Create a risk matrix:

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| [risk description] | Technical / Business / Legal | Low/Med/High | Low/Med/High | [strategy] |

Categories to evaluate:
- **Technical risks**: Scalability, complexity, team expertise gaps
- **Business risks**: Market timing, competition, monetization uncertainty
- **Legal/Regulatory**: Data privacy (GDPR, CCPA), compliance, licensing
- **Operational**: Vendor lock-in, single points of failure

## Output Format

Write all findings to `projects/<project-slug>/research.md` with this structure:

```markdown
# Research: <Project Name>

## Executive Summary
<!-- 2-3 paragraph overview of findings -->

## Market Analysis
### Market Size & Trends
### Target Demographics
### Demand Signals

## Competitor Analysis
### Direct Competitors
<!-- comparison table -->
### Indirect Competitors
### Open Source Alternatives

## Technical Feasibility
### Available APIs & Services
### Recommended Libraries
### Infrastructure Requirements

## Risk Assessment
<!-- risk matrix table -->

## Key Insights
<!-- Bullet points of the most important takeaways -->

## Sources
<!-- Numbered list of all sources with URLs -->
```

## Important Notes

- Always cite sources with URLs
- Present findings to the user before finalizing — ask if anything changes their vision
- Flag any deal-breakers prominently with `[!CAUTION]` alerts
- If research reveals the idea is already well-served by existing products, be honest about it and suggest differentiation strategies
