# Business Proposal: Market Insights Platform

## Executive Summary

Our market analysis team spends **days manually assembling reports** by cross-referencing Sensor Tower data, monitoring online communities, and checking Vietnamese gaming regulations. This proposal outlines a purpose-built internal web application that **automates 80% of this workflow using AI**, reducing report generation time from days to under 30 minutes while improving data coverage from 1-2 sources per report to 4-5.

The MVP can be built in **6 weeks** at a running cost of **~$50/month**, with the full production version operational within 12 weeks at ~$200/month.

---

## The Problem

| Current Pain Point | Business Impact |
|---|---|
| Reports take **2-5 days** to compile manually | Delayed market decisions; missed windows of opportunity |
| Each report typically cites only **1-2 data sources** | Incomplete picture; blind spots in analysis |
| Community sentiment is tracked **ad hoc** (if at all) | Missing player feedback signals that predict market shifts |
| Vietnamese regulation changes (G1-G4 licenses) are discovered **reactively** | Compliance surprises; missed new market entrants |
| No standardized report format | Inconsistent quality; difficult to compare across periods |
| Analysis is trapped in individual analysts' heads | Knowledge silos; no institutional memory |

---

## The Solution

An internal web application where the team can:

1. **See everything in one place** — Dashboard combining app rankings, revenue trends, community sentiment scores, and regulatory updates
2. **Generate reports with AI** — Select a template, define parameters (market, time period, focus apps), and let AI produce a structured draft in minutes
3. **Edit and refine collaboratively** — Rich text editor where analysts review, edit, and enhance AI-generated content
4. **Publish and share** — Standardized, exportable reports with full data traceability

### How AI is Used (Responsibly)

| AI Task | What the AI Does | What the Human Does |
|---|---|---|
| **Report drafting** | Synthesizes data from 4-5 sources into a structured draft | Reviews, edits, adds judgment, and publishes |
| **Sentiment analysis** | Classifies community posts as positive/neutral/negative | Validates sentiment labels, identifies actionable themes |
| **Data summarization** | Condenses thousands of data points into key takeaways | Interprets business implications, makes recommendations |

> **All AI outputs are editable drafts** — no report is published without human review. AI assists with speed and coverage; humans provide judgment and accountability.

---

## Value Proposition

### Quantified Benefits

| Metric | Current State | With Platform | Improvement |
|---|---|---|---|
| Time to produce 1 report | 2-5 days | <30 minutes | **~90% reduction** |
| Data sources per report | 1-2 | 4-5 (market + community + regulation) | **2-3x coverage** |
| Reports produced per week | 1-2 | 5-10 | **5x throughput** |
| Regulation tracking | Reactive (weeks late) | Proactive (monthly automated) | **Eliminates surprises** |
| Community sentiment visibility | Ad hoc / none | Systematic with scoring | **New capability** |

### Strategic Value

- **Faster decisions**: Market insights available in hours instead of days
- **Better decisions**: More data sources = fewer blind spots
- **Regulatory readiness**: Automated tracking of G1-G4 license changes under the new Decree 147/2024
- **Institutional knowledge**: Reports stored centrally with data snapshots — any analyst can reference past analyses
- **Scalable**: Same tool works for 5 or 20 analysts without proportional effort increase

---

## Scope & Timeline

### Phase 1: MVP (Weeks 1-6)

| What's Included | What's Not Included |
|---|---|
| ✅ Dashboard with Sensor Tower KPIs and trends | ❌ Multi-model AI (single provider for now) |
| ✅ App Store review scraping + sentiment analysis | ❌ Reddit, Discord, Facebook scraping |
| ✅ Vietnamese regulation tracker (abei.gov.vn) | ❌ Real-time collaborative editing |
| ✅ AI report generation (2 templates) | ❌ Automated report scheduling |
| ✅ Rich text editor for review and editing | ❌ Custom template builder |
| ✅ Report export (Markdown, HTML) | ❌ API for external integrations |

### Phase 2: Production (Weeks 7-12)

Adds: All community data sources (Reddit, Discord, Facebook), multi-AI routing for cost optimization, real-time collaborative editing, 4+ report templates, automated scheduling.

### Phase 3: Scale (Months 4-6+)

Adds: Historical context AI (learns from past reports), custom template builder, auto-distribution (email/Slack), support for 20+ users.

---

## Investment & Running Costs

### Development Cost

| Item | Estimate |
|---|---|
| MVP development (6 weeks, AI-assisted) | Internal team time |
| Production enhancements (6 weeks) | Internal team time |
| **No external development cost** | Built internally with AI assistance |

### Monthly Operating Costs

| Phase | Monthly Cost | What It Covers |
|---|---|---|
| **MVP** | ~$50/mo | AI API usage ($30-50) + hosting (free tier) |
| **Production** | ~$200/mo | AI APIs ($60-130) + scraping ($49) + hosting ($20) |
| **Scale** | ~$650/mo | AI APIs ($230-580) + scraping ($149) + hosting ($45) + jobs ($50) |

### ROI Estimate

| Factor | Calculation |
|---|---|
| Analyst time saved | 5 analysts × 4 hrs/week saved × 48 weeks = **960 hours/year** |
| At avg analyst cost of $25/hr | 960 × $25 = **$24,000/year in labor savings** |
| Annual platform cost (Production) | 12 × $200 = **$2,400/year** |
| **Net annual value** | **~$21,600/year (10x ROI)** |

---

## Risk Management

| Risk | Likelihood | Impact | Our Mitigation |
|---|---|---|---|
| AI produces inaccurate analysis | Medium | High | All AI outputs are drafts — human review required before publishing |
| Community scraping blocked by platforms | Medium | Medium | Using managed scraping service with automatic maintenance; manual import fallback |
| Regulation website changes structure | Low | Medium | Alert monitoring on scraper failures; manual data entry as backup |
| AI API costs exceed budget | Medium | Medium | Usage monitoring + cost alerts; AI routing to cheaper models for simple tasks |
| Team doesn't adopt the tool | Low | Medium | Involve analysts in template design; start with supervised mode |

---

## Success Metrics

We propose tracking these KPIs to measure the platform's effectiveness:

| KPI | Target (3 months) | Target (6 months) |
|---|---|---|
| Report generation time | < 30 minutes | < 15 minutes |
| Reports produced per week | 3-5 | 5-10 |
| Team adoption rate | 3/5 analysts using weekly | 5/5 analysts using weekly |
| AI cost per report | < $5 | < $2 |
| Analyst satisfaction (survey) | 7/10 | 8/10 |
| Data source coverage per report | 2 sources | 4-5 sources |

---

## Recommendation

We recommend **proceeding with the MVP build (6 weeks)** because:

1. **Low risk**: Internal tool for 5 users — no external customer exposure
2. **Low cost**: ~$50/month running cost, no external development spend
3. **High leverage**: 10x ROI from analyst time savings alone
4. **Proven technology**: All AI APIs, scraping tools, and frameworks are production-tested
5. **Quick validation**: Working prototype in 6 weeks provides concrete data on value

### Requested Approval

- [ ] Approve MVP development timeline (6 weeks)
- [ ] Confirm Anthropic (Claude AI) API budget (~$50/month)
- [ ] Confirm Apify scraping service budget ($49/month at Production phase)
- [ ] Confirm Vercel hosting ($0 MVP, $20/month at Production phase)

---

## Appendix: Planning Documents

For technical details, see the full planning documentation:

| Document | Contents |
|---|---|
| [overview.md](./overview.md) | Problem, users, requirements, scope |
| [research.md](./research.md) | Market analysis, competitor intelligence, API feasibility |
| [findings.md](./findings.md) | All decisions, risk log, session notes |
| [tech-stack.md](./tech-stack.md) | Technology evaluation with scored comparisons |
| [architecture.md](./architecture.md) | System diagrams, database schema, API contracts |
| [implementation-plan.md](./implementation-plan.md) | 6 milestones, 56 tasks, Gantt timeline |
| [handoff.md](./handoff.md) | Complete development brief for coding agent |
