# Business Proposal: AI Agentic UA Engine

> **Prepared by**: Solutions Architect (AI-assisted)
> **Date**: 2026-03-13
> **Audience**: Leadership / Stakeholders
> **Export**: Run `/plan-export` to generate .docx version

---

We propose building an **AI-powered User Acquisition Engine** that leverages our **Enterprise Tech Stack** (PostgreSQL, S3, Spark, Iceberg) to automate the full advertising cycle.

The system uses **Google Sheets as the 'Business Console'** for the UA Team, backed by internal AI workflows that make data-driven decisions using our deep data lake.

**Bottom line**: This engine can **reduce creative costs by 50-70%**, **improve ROAS by 15-25%**, and requires **zero new software infrastructure**. We can launch a pilot in just **8 weeks**.

---

## The Problem

| Challenge | Current State | Business Impact |
|---|---|---|
| **Slow campaign launches** | 2-4 weeks to go from idea to live ad | Missed market windows, slow reaction to trends |
| **Expensive creative production** | Manual design for each variant | High cost, low testing volume (5-10/week) |
| **Reactive optimization** | Managers check dashboards daily | 4-24 hour reaction to anomalies, wasted spend |
| **Rising UA costs** | CPI increasing 20-30% YoY | Margins shrinking without ROAS improvement |
| **Talent bottleneck** | Need 3-5 UA people per game | Hard to scale to more titles |

---

## The Solution

```
┌──────────────────────────────────────────────────────────┐
│             Enterprise-Integrated UA Engine              │
│                                                          │
│  ┌──────────────────┐      ┌──────────────────────────┐  │
│  │  Business Console│      │    Enterprise Backend    │  │
│  │ (Google Sheets)  │ ↔  │ (Internal n8n + PG + S3) │  │
│  └──────────────────┘      └────────────┬─────────────┘  │
│                                           │              │
│  ┌────────────────────────────────────────┴───────────┐  │
│  │          Spark + Iceberg Analytics Layer           │  │
│  │      (Predictive LTV, Attribution, Insights)       │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### What the AI Does

| Pillar | Before (Manual) | After (AI Engine) |
|---|---|---|
| **Planning** | UA manager analyzes spreadsheets, sets budgets | AI agent analyzes AppsFlyer data, generates optimized campaign plans |
| **Creative** | Designers create 5-10 variants manually | AI generates 50-100 variants using GenAI tools (Creatify, OpenArt.ai) |
| **Deployment** | Manual upload to each ad platform | Automated deployment to TikTok, Facebook, Google in minutes |
| **Monitoring** | Check dashboards 1-2x daily | AI monitors 24/7, auto-pauses bad ads, alerts on anomalies |
| **Optimization** | Weekly manual adjustments | Continuous AI-driven optimization with human approval for big changes |

### Safety: Human-in-the-Loop

The AI doesn't operate unchecked. Three tiers of control:

| Tier | AI Authority | Example |
|---|---|---|
| 🟢 **Full Auto** | AI acts immediately | Pause underperforming ads, scale budget ≤20% |
| 🟡 **Semi-Auto** | AI recommends, human approves via Slack | New campaigns, budget changes >20% |
| 🔴 **Human Only** | Human decides entirely | Total budget allocation, brand direction |

---

## Expected Impact

| Metric | Current | With AI Engine | Improvement |
|---|---|---|---|
| Campaign launch time | 2-4 weeks | < 2 days (15 min automated) | **95%+ faster** |
| Creative variants per week | 5-10 | 50-100 | **10x more testing** |
| Production costs | Baseline | -50% to -70% | **Significant savings** |
| ROAS | Baseline | +15% to +25% | **Higher returns** |
| Anomaly reaction time | 4-24 hours | < 1 hour | **20x faster** |
| UA headcount per game | 3-5 people | 1-2 people | **60% reduction** |

---

## Investment & Timeline

### 8-Week Pilot Plan

| Phase | Duration | Deliverable |
|---|---|---|
| **P1: Enterprise Setup** | 2 Weeks | n8n connected to Internal PG, S3, & Sheets |
| **P2: AI & Data Logic** | 2 Weeks | Spark views for LTV-based AI campaign plans |
| **P3: Creative & Deploy**| 2 Weeks | S3 creative pipeline + TikTok live deployment |
| **P4: Monitoring & Scale**| 2 Weeks | Automated monitoring loop + multi-platform |

### Cost Breakdown

| Category | Pilot (8 weeks) | Monthly (Production) |
|---|---|---|
| **Software Infra** | **$0** (Using S3, PG, Spark) | **$0** |
| **n8n Hosting** | Internal Host | Internal |
| **Ad Spend** | $4,000 ($500/wk) | $10K+ /mo |
| **AI API Costs** | ~$40 (Claude) | ~$100-200 /mo |
| **GenAI Tools** | Free/Pilot tiers | ~$50 /mo |
| **Total (excl. ad spend)**| **~$40** | **~$150-250 /mo** |

> **Key insight**: The engine costs ~$4,300/year to operate (excluding ad spend). Even a 1% ROAS improvement on $100K annual ad spend would save $1,000+ — the system pays for itself with minimal improvement.

### ROI Projection

| Scenario | Annual Ad Spend | ROAS Improvement | Additional Revenue | ROI |
|---|---|---|---|---|
| Conservative | $100K | +10% | +$10K | 2.3x |
| Target | $300K | +20% | +$60K | 14x |
| Optimistic | $600K | +25% | +$150K | 35x |

*Plus: headcount savings of 2-3 FTE per game × salary*

---

## Risk Management

| Risk | Likelihood | Mitigation |
|---|---|---|
| AI makes bad budget decisions | Low | 6 anti-hallucination guardrails enforce hard limits |
| AI-generated creatives underperform | Medium | Quality gate + A/B test before scaling |
| Ad platform blocks automation | Low | Official APIs only, comply with ToS |
| System downtime disrupts campaigns | Low | Manual fallback always available |
| Pilot doesn't show lift | Medium | Review Gate at Week 6 — pivot before further investment |

---

## Why Now?

1. **AI-generated creatives are mainstream** — 30-40% of top publishers already use GenAI
2. **UA costs rising 20-30% YoY** — automation is the only way to maintain margins
3. **We already have the data infrastructure** — AppsFlyer pipeline gives us real-time signals
4. **Team has the skills** — n8n and Dify experience means fast time-to-value
5. **Low-risk pilot** — $5K investment validates the approach before committing

---

## Decision Requested

**Approve the 8-week pilot** to validate the AI UA Engine on 1 game (Total Football) with TikTok Ads.

| What We Need | From Whom |
|---|---|
| Engineering allocation (1 FTE, 8 weeks) | Engineering Lead |
| TikTok Ads pilot budget ($500/week × 8 weeks) | Marketing / Finance |
| API keys for TikTok, GenAI tools | UA Team |
| Weekly 1-hour review sessions | UA Manager |
| Production server access (Week 5+) | DevOps / IT |

**Success criteria at Week 6 Review Gate**:
- AI-managed campaigns achieve comparable or better CPI vs. manual
- Creative production is ≥5x faster
- System operates reliably with <1hr anomaly response

If the gate passes → expand to Facebook and Google (Sprints 4-5).
If it doesn't → lessons learned documented, minimal investment lost.

---

## Appendix: Key Documents

| Document | Contents |
|---|---|
| [Overview](./overview.md) | Requirements, NFRs, integrations, HITL framework |
| [Research](./research.md) | Build-vs-buy analysis, API feasibility, PoCs |
| [Tech Stack](./tech-stack.md) | Technology evaluations with ADR-format decisions |
| [Architecture](./architecture.md) | C4 diagrams, data contracts, security, deployment |
| [Implementation Plan](./implementation-plan.md) | Sprint tasks, Gantt chart, milestones |
| [Technical Handoff](./handoff.md) | Developer-ready setup guide |
| [Findings](./findings.md) | All 15 ADRs, risk log, session notes |

> 💡 **Tip**: Run `/plan-export` to export this proposal as a formatted .docx for presentation.
