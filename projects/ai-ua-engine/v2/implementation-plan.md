# Implementation Plan: AI Agentic UA Engine

> **Created**: 2026-03-13
> **Timeline**: 8 weeks (4 sprints × 2 weeks)
> **Team**: 1 engineer + 1 UA manager (part-time reviewer)

## Milestones

| Milestone | Sprint | Week | Success Criteria | Gate |
|---|---|---|---|---|
| **M1: Integrated Foundation** | 1 | 2 | Internal n8n connected to Enterprise PG & S3; Google Sheets synced | Demo: sheet row creates PG record |
| **M2: Data-Driven Planning** | 2 | 4 | Spark/Iceberg LTV views created; Claude logic generates campaign plans | Demo: high-LTV segment targeting |
| **M3: Creative & Deploy** | 3 | 6 | S3-linked creative pipeline; TikTok API live; HITL Slack flow | Demo: S3 asset → TikTok live |
| **M4: Monitoring & Scale** | 4 | 8 | Automated monitoring active; full loop finished; operational runbook | Hand-off to UA team |

---

## Sprint Breakdown

### Sprint 1: Enterprise Integration (Weeks 1-2)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 1.1 | Setup internal n8n instance + authentication (SSO) | Eng | 1d | None | Working n8n with SSO |
| 1.2 | Create `ua_engine` schema/tables in Enterprise PostgreSQL | Eng | 1d | 1.1 | State DB ready |
| 1.3 | Configure S3/MinIO bucket & media serving endpoints | Eng | 1d | 1.1 | Creative vault ready |
| 1.4 | Sync Google Sheets console to internal n8n/PG | Eng | 1d | 1.2 | Bidirectional sheet sync |
| 1.5 | Connect AppsFlyer real-time feed to internal n8n | Eng | 1d | 1.1 | Events flowing to PG |
| 1.6 | Setup TikTok Ads API credentials in vault | Eng | 0.5d | 1.1 | Verified API access |
| 1.7 | Integration testing: sheet-to-backend connectivity | Eng | 1d | 1.4 | CRUD test passed |

**Sprint 1 Total**: ~6.5 person-days

---

### Sprint 2: AI Logic + Analytics Layer (Weeks 3-4)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 2.1 | Define Spark SQL views (Iceberg) for LTV/Spend signals | DataEng | 2d | 1.5 | Analytics views ready |
| 2.2 | Build Claude 3.5 Sonnet "Strategist" workflow in n8n | Eng | 1.5d | 2.1 | Plan logic: Analytics → AI → Sheet |
| 2.3 | Implement Budget Guardrails node in n8n (Deterministic) | Eng | 1d | 1.2 | Safety rules script |
| 2.4 | Configure creative prompt templates for GenAI tools | UA | 1d | None | Prompt library ready |
| 2.5 | Build campaign planning flow: Sheet trigger → AI recommend | Eng | 1.5d | 2.2 | Full planning loop |
| 2.6 | Validation: Campaign plans align with LTV targets | UA | 1d | 2.5 | Strategy sign-off |

**Sprint 2 Total**: ~8 person-days

---

### Sprint 2: Creative Pipeline + TikTok Deploy (Weeks 3-4)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 2.1 | Integrate Creatify API in n8n (video generation) | Eng | 1.5d | 1.7 | n8n node: brief → video asset |
| 2.2 | Integrate OpenArt.ai API in n8n (image generation) | Eng | 1d | 1.7 | n8n node: brief → image asset |
| 2.3 | Build Creative Production workflow (parallel GenAI calls) | Eng | 1.5d | 2.1, 2.2 | Workflow: brief → 10 variants stored |
| 2.4 | Build TikTok Campaign Deployment workflow | Eng | 2d | 1.2 | Workflow: assets + config → live TikTok campaign |
| 2.5 | Build creative quality gate (basic scoring before deploy) | Eng | 1d | 2.3 | Filter: reject bad outputs |
| 2.6 | Build HITL notification workflow (Slack) | Eng | 1d | None | Slack: plan review → approve/reject buttons |
| 2.7 | Set up file storage for creative assets | Eng | 0.5d | 1.1 | Docker volume mounted |
| 2.8 | End-to-end test: brief → creatives → TikTok live | Eng + UA | 1.5d | 2.4, 2.6 | First AI-generated campaign live |

**Sprint 2 Total**: ~10 person-days
**Sprint 2 Cost**: ~$500 TikTok ad spend for pilot

---

### Sprint 3: Monitoring + HITL (Weeks 5-6)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 3.1 | Build Dify "Campaign Monitor" agent (ReAct) | Eng | 2d | 1.3 | Agent analyzes metrics, outputs decisions |
| 3.2 | Build n8n Monitor Trigger workflow (CRON hourly/4hr) | Eng | 1d | 3.1, 1.3 | Scheduled: AppsFlyer → Dify → decision |
| 3.3 | Implement Full Auto tier actions (pause, minor scale) | Eng | 1.5d | 3.2 | Auto-pause on CTR < threshold |
| 3.4 | Implement Semi-Auto tier (Slack approval flow) | Eng | 1.5d | 2.6 | Budget +>20% → Slack → approve → execute |
| 3.5 | Implement anti-hallucination guardrails | Eng | 1d | 3.3 | Daily cap, % limit, schema validation |
| 3.6 | Build audit trail logging (all decisions → PostgreSQL) | Eng | 1d | 1.4 | Append-only agent_decisions table |
| 3.7 | Build Daily Summary workflow (Slack digest) | Eng | 0.5d | 3.6 | Daily 9am: actions taken, spend, performance |
| 3.8 | **📌 Week 6 Review Gate**: compare AI vs. manual performance | UA | 1.5d | 3.2 | Report: AI lift measurement |

**Sprint 3 Total**: ~10 person-days

> **REVIEW GATE** at Week 6:
> - Compare AI-managed campaigns vs. manual baseline
> - Metrics: CPI, ROAS, launch time, creative volume
> - **Go/No-Go decision** for multi-platform expansion

---

### Sprint 4: Multi-Platform Expansion (Weeks 7-8)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 4.1 | Configure Facebook Ads API credentials in n8n | Eng | 0.5d | M3 approved | Verified FB API connection |
| 4.2 | Configure Google Ads API credentials in n8n | Eng | 0.5d | M3 approved | Verified Google API connection |
| 4.3 | Build Facebook Campaign Deployment workflow | Eng | 2d | 4.1 | Workflow: assets → FB campaign live |
| 4.4 | Build Google Campaign Deployment workflow | Eng | 2d | 4.2 | Workflow: assets → Google campaign live |
| 4.5 | Build Spend Data Pull workflow (all 3 platforms, 2-4x/day) | Eng | 1.5d | 4.1, 4.2 | CRON: pull spend → PostgreSQL |
| 4.6 | Extend Monitor agent to cover all 3 platforms | Eng | 1d | 4.5 | Multi-platform monitoring |
| 4.7 | Build Token Refresh workflows (FB long-lived, Google OAuth) | Eng | 1d | 4.1, 4.2 | Auto-refresh for all platforms |
| 4.8 | Integration testing: 3-platform campaign launch | Eng + UA | 1.5d | 4.3, 4.4 | Demo: simultaneous launch |

**Sprint 4 Total**: ~10 person-days

---

### Sprint 5: Analytics + Optimization (Weeks 9-10)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 5.1 | Build Dify "Creative Analyst" agent | Eng | 2d | 3.6 | Agent: tags creatives, finds winning patterns |
| 5.2 | Build creative tagging workflow (platform metrics + LLM) | Eng | 1.5d | 5.1 | Automated: ad metrics → LLM → tags stored |
| 5.3 | Integrate AppLovin Axon 2.0 API | Eng | 1d | 4.5 | LTV-based bid data flowing |
| 5.4 | Integrate Appodeal AutoBid API | Eng | 1d | 4.5 | Automated bid optimization |
| 5.5 | Build closed-loop feedback (analyst → strategist) | Eng | 1d | 5.1, 1.6 | Winning patterns feed next campaign plan |
| 5.6 | Stress testing (simulate 100 campaigns) | Eng | 1d | All | Load test report |
| 5.7 | Write operational runbook | Eng | 1d | All | Troubleshooting guide |
| 5.8 | Handoff to UA team | Eng + UA | 1.5d | 5.7 | Training session + documentation |

**Sprint 5 Total**: ~10 person-days

---

## Gantt Chart

```mermaid
gantt
    title AI Agentic UA Engine — 10-Week Implementation
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Sprint 1: Foundation
    Docker Compose setup           :s1_1, 2026-03-17, 2d
    TikTok API config              :s1_2, after s1_1, 0.5d
    AppsFlyer pipeline connect     :s1_3, after s1_1, 1d
    PostgreSQL schema              :s1_4, after s1_1, 1d
    Load KB data                   :s1_5, after s1_1, 1d
    Campaign Strategist agent      :s1_6, after s1_5, 2d
    Planning workflow              :s1_7, after s1_6, 1d
    Token refresh                  :s1_8, after s1_2, 0.5d
    Integration testing            :s1_9, after s1_7, 1d
    M1: Foundation Live            :milestone, m1, after s1_9, 0d

    section Sprint 2: Creative + Deploy
    Creatify integration           :s2_1, after m1, 1.5d
    OpenArt.ai integration         :s2_2, after m1, 1d
    Creative Production workflow   :s2_3, after s2_1, 1.5d
    TikTok Deploy workflow         :s2_4, after m1, 2d
    Quality gate                   :s2_5, after s2_3, 1d
    HITL Slack workflow            :s2_6, after m1, 1d
    File storage setup             :s2_7, after m1, 0.5d
    E2E test                       :s2_8, after s2_4, 1.5d
    M2: First AI Campaign          :milestone, m2, after s2_8, 0d

    section Sprint 3: Monitor + HITL
    Campaign Monitor agent         :s3_1, after m2, 2d
    Monitor trigger workflow       :s3_2, after s3_1, 1d
    Full Auto actions              :s3_3, after s3_2, 1.5d
    Semi-Auto Slack flow           :s3_4, after s3_3, 1.5d
    Anti-hallucination guards      :s3_5, after s3_3, 1d
    Audit trail logging            :s3_6, after s3_3, 1d
    Daily summary                  :s3_7, after s3_6, 0.5d
    REVIEW GATE                    :crit, milestone, m3, after s3_7, 0d

    section Sprint 4: Multi-Platform
    FB API config                  :s4_1, after m3, 0.5d
    Google API config              :s4_2, after m3, 0.5d
    FB Deploy workflow             :s4_3, after s4_1, 2d
    Google Deploy workflow         :s4_4, after s4_2, 2d
    Spend Data Pull workflow       :s4_5, after s4_1, 1.5d
    Extend Monitor                 :s4_6, after s4_5, 1d
    Token refresh (FB+Google)      :s4_7, after s4_1, 1d
    3-platform test                :s4_8, after s4_3, 1.5d
    M4: Multi-Platform             :milestone, m4, after s4_8, 0d

    section Sprint 5: Analytics + Handoff
    Creative Analyst agent         :s5_1, after m4, 2d
    Creative tagging workflow      :s5_2, after s5_1, 1.5d
    AppLovin Axon 2.0              :s5_3, after m4, 1d
    Appodeal AutoBid               :s5_4, after m4, 1d
    Closed-loop feedback           :s5_5, after s5_1, 1d
    Stress testing                 :s5_6, after s5_5, 1d
    Operational runbook            :s5_7, after s5_6, 1d
    Handoff                        :s5_8, after s5_7, 1.5d
    M5: Full Engine                :milestone, m5, after s5_8, 0d
```

---

## Assumptions

1. Team has existing n8n and Dify experience (no ramp-up time)
2. AppsFlyer data pipeline is stable and delivering real-time events
3. TikTok Ads API sandbox or test account available for Sprint 1
4. GenAI tool API keys obtainable within Sprint 1 timeframe
5. UA manager available ~4 hrs/week for reviews and HITL testing
6. Company server available for production deployment by Sprint 3
7. $500/week ad spend budget approved for pilot (Sprints 2-5)

## Constraints

1. **Budget**: $500/wk ad spend during pilot → limits testing scale
2. **Team size**: 1-2 engineers → tasks must be sequential, not parallel
3. **API access**: Google Ads Standard tier approval may take time
4. **OpenClaw maturity**: If MCP protocol has issues, fallback to n8n-native API nodes
5. **Review Gate**: Sprint 4+ depends on positive results at Week 6 gate

## Risks & Mitigations

| Risk | Sprint | Impact | Mitigation |
|---|---|---|---|
| Docker environment issues | 1 | 1-2 day delay | Pre-test Docker setup on clean machine |
| GenAI API key delays | 2 | Blocks creative pipeline | Apply for keys in Sprint 1, use free tiers |
| TikTok API sandbox limitations | 2 | Can't test deployment | Use production API with minimal spend ($5/day) |
| Monitor agent hallucination | 3 | Wrong budget decisions | Anti-hallucination guardrails built in Sprint 3 |
| Review Gate fails | 3 | Project pivot or cancellation | Early signal monitoring from Sprint 2 data |
| Google API access denied | 4 | Can't expand to Google | Apply early (Sprint 1), fallback to Basic tier with batching |
| Stress test reveals bottlenecks | 5 | Need architecture changes | n8n queue mode, Redis scaling |

## Definition of Done

A task is "done" when:
- [ ] Code committed and working in Docker environment
- [ ] Integration test passing (end-to-end workflow)
- [ ] Audit logging verified (decision trail in PostgreSQL)
- [ ] Error handling tested (API failure → graceful recovery)
- [ ] Documentation updated (workflow description in runbook)
