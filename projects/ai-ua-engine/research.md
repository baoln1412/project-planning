# Research: AI Agentic UA Engine

> **Researched**: 2026-03-13
> **Status**: Complete

## Executive Summary

The AI Agentic UA Engine is technically feasible with the proposed n8n + Dify + OpenClaw stack. A **major simplification** emerged: the company already has a real-time AppsFlyer data pipeline (attribution, user activities, transactions), which serves as the primary analytics data source. This reduces ad platform API calls by 75-85% — APIs are now needed mainly for campaign management (writes) and periodic spend pulls (2-4x/day). The creative tagging pipeline uses platform-level metrics + LLM analysis instead of GPU video processing. A $500/week TikTok-only pilot is feasible to validate the core loop.

## Technical Landscape

### Existing Solutions

| Solution | Type | Strengths | Weaknesses | Fit for Our Use Case |
|---|---|---|---|---|
| Smartly.io | Buy (SaaS) | Campaign automation, creative optimization | Expensive, less AI-agentic | ⚠️ Partial — good for automation but not custom AI reasoning |
| Revealbot | Buy (SaaS) | Rules-based ad optimization | No creative production, limited AI | ⚠️ Partial — covers Phase 3 monitoring only |
| Segwise AI | Buy (SaaS) | Creative tagging and anomaly detection | Specialized — doesn't cover full loop | ⚠️ Partial — good complement, not alternative |
| Custom n8n + Dify | Build | Full control, AI-agentic, self-hosted | Higher dev effort, more ops complexity | ✅ Good — best fit for full-loop automation |

### Build vs. Buy Analysis

| Component | Build | Buy / SaaS | Open Source | Recommendation |
|---|---|---|---|---|
| Orchestrator | n8n (self-hosted) | Make ($99-299/mo) | n8n Community | **Build (n8n)** — team experience, cost control |
| AI Brain | Dify (self-hosted) | — | Dify Community | **Build (Dify)** — RAG + ReAct built-in |
| MCP Gateway | OpenClaw | — | OpenClaw | **Build** — with n8n fallback |
| Creative Production | — | Apptovid, Creatify, etc. | — | **Buy** — SaaS APIs, don't reinvent |
| Bidding Optimization | — | Axon 2.0, AutoBid | — | **Buy** — integrate, don't build |
| Creative Analytics | Hybrid | Segwise ($TBD) | — | **Hybrid** — platform metrics + LLM first, add Segwise if budget allows |
| Attribution | — | AppsFlyer (existing) | — | **Buy** — already in use |
| Notifications | Slack/Telegram webhook | — | — | **Build** — simple webhook integration |

## API & Integration Feasibility

| API / Service | Version | Auth | Rate Limits | Data Format | SLA | Notes |
|---|---|---|---|---|---|---|
| Facebook Ads | Graph API v21.0 | Long-lived Token + App | 9,000 pts/hr (1hr window) | JSON | HTTP 429 on overage | Writes + spend pulls only. 2-4x/day |
| Google Ads | GAQL | OAuth2 + Dev Token + MCC | 15K ops/day (Basic), Unlimited (Standard) | JSON | RESOURCE_TEMPORARILY_EXHAUSTED | Writes + spend pulls. Basic tier now sufficient |
| TikTok Ads | Marketing API v1.3 | Access Token + App | 600 req/min (1-min sliding window) | JSON | HTTP 429 on overage | Writes + spend pulls only. 2-4x/day |
| AppsFlyer | Pull API + Real-time Pipeline | API Token | Real-time stream | JSON/CSV | — | **Primary data source** — attribution, events, transactions. Already built |
| Apify | Web Scraping | API Key | Per-plan limits | JSON | — | For competitor intelligence (Facebook Ad Library) |
| LLM (Claude/GPT) | Latest | API Key | Per-plan tokens | JSON | — | ~$0.01-0.05 per agent call |
| GenAI tools (6) | Varies | API Key | Per-plan limits | JSON + Media | — | Each has different output latency |

### Integration Risk Assessment

| Integration | Complexity | Risk | Mitigation |
|---|---|---|---|
| Facebook API token refresh | Medium | Token expiry causes workflow halt | Auto-refresh cron job, expiration alerts |
| Google Standard API access | Medium | May not get approval immediately | Apply early, fallback to Basic with batching |
| TikTok batch query design | Low | Need efficient query patterns | Batch ad IDs per request to stay under 600/min |
| GenAI tool availability | Medium | API downtime or slow response | Queue with timeout, fallback to alternative tool |
| OpenClaw maturity | High | MCP protocol still evolving | Fallback: direct n8n API nodes |

## Infrastructure Options

| Option | Provider | Approach | Monthly Cost | Scalability | Complexity |
|---|---|---|---|---|---|
| **Pilot (Dev)** | Local machine | Docker Compose (n8n + Dify + Redis) | $0 (local) | Single game | Low |
| **Production v1** | Company server | Docker Compose on-prem | $50-100/mo (server cost share) | 3-5 games | Medium |
| **Production v2** | Cloud (AWS/GCP) | Docker on VM / Kubernetes | $200-500/mo | 10+ games | High |

## Compliance & Licensing

### Open Source License Compatibility

| Library | License | Compatible? | Notes |
|---|---|---|---|
| n8n | Fair-code (Sustainable Use License) | ⚠️ Conditional | Free for internal use. Cannot resell as service |
| Dify | Apache 2.0 | ✅ | Fully open source |
| OpenClaw | Apache 2.0 | ✅ | Fully open source |
| Redis | BSD 3-Clause | ✅ | Standard open source |

### Regulatory Requirements

| Regulation | Applies? | Impact on Architecture | Status |
|---|---|---|---|
| iOS ATT/IDFA | Yes | Limited user-level targeting data. Must rely on contextual + cohort signals | ⬜ Design around it |
| SKAdNetwork | Yes | Attribution delayed and aggregated. LTV models need adaptation | ⬜ Design around it |
| Ad Platform ToS | Yes | Automation must comply with each platform's API usage policies | ⬜ Review per platform |
| PDPA (Thailand) | TBD | If processing Thai user data, privacy rules apply | ⬜ Not assessed |

## PoC Recommendations

| PoC | Purpose | Success Criteria | Estimated Effort |
|---|---|---|---|
| **Core Loop PoC** | Validate: Dify agent → creative brief JSON → n8n → TikTok deployment | Creative goes live within 15 minutes | 2 weeks |
| **Rate Limit PoC** | Test hourly metric pulls for 100+ ads on TikTok without hitting 429 | 24hrs continuous pulling without errors | 3 days |
| **HITL PoC** | Test Slack approval flow: AI recommends budget increase → Slack alert → human approves → n8n executes | End-to-end approval in <30 mins | 3 days |
| **Creative Quality PoC** | Deploy 10 AI-generated creatives vs. 10 human-made, compare CPI/ROAS | AI creatives within 80% of human performance | 2 weeks ($500 ad spend) |

## Risk Assessment

| Risk | Category | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| API rate limits block pipeline | Tech | Medium | High | Queue + wait + retry + backoff. Confirmed limits are manageable with batching |
| LLM hallucination budget disaster | Tech | Low | Critical | Deterministic guardrails (hard caps), HITL for >20% changes, daily budget ceiling |
| GenAI creative quality too low | Tech | Medium | Medium | Quality scoring gate, A/B test before scaling, human review for key creatives |
| OpenClaw not production-ready | Tech | Medium | Medium | Fallback to direct n8n API nodes — no architectural dependency |
| Google denies Standard API access | Integration | Low | High | Apply early, use batch optimization on Basic tier |
| $500/wk pilot budget too tight | Business | Medium | Low | Focus TikTok-only, use free tiers of GenAI tools, minimal ad variants |
| Team overloaded with new stack | Ops | Low | Medium | Team knows n8n + Dify. OpenClaw is the only new component |
| Ad platform policy blocks automation | Compliance | Low | High | Review ToS pre-launch. Use official APIs only (no scraping for management) |

## Key Insights

1. **Existing AppsFlyer pipeline is a game-changer** — Real-time attribution + user data eliminates the need for constant ad platform polling. API calls drop 75-85%.
2. **API rate limits are now a non-issue** — With only 2-4 spend pulls/day + on-demand writes, we’re far below all platform limits. Google Basic tier is now sufficient.
3. **Creative tagging without GPU is viable** — Platform-level metrics (CTR, CPI, ROAS per creative) + LLM analysis of creative briefs replaces expensive video processing.
4. **Pilot budget is tight but feasible** — $500/wk on TikTok validates the core loop. GenAI tool free tiers cover initial creative production.
5. **n8n + Dify is the right stack** — Team familiarity eliminates ramp-up time. Self-hosted eliminates SaaS costs.

## Sources

1. NotebookLM: AI Agentic UA Engine notebook (3 queries)
2. `data-collection-plan` skill — data inventory framework
3. `customer-lifetime-value` skill — LTV modeling and CAC:LTV ratios
4. `automation-workflow` skill — n8n workflow patterns and error handling
5. `benchmarking-report` skill — UA benchmarking methodology
