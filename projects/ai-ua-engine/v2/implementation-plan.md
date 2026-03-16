# Implementation Plan: AI Agentic UA Engine

> **Created**: 2026-03-13 | **Updated**: 2026-03-16
> **Timeline**: 16 weeks (8 sprints × 2 weeks) — Core (Sprints 1-5) + pLTV Extension (Sprints 6-8)
> **Team**: 1 engineer + 1 UA manager (Sprints 1-5) | + 1 data analyst + 1 data engineer (Sprints 6-8)

## Milestones

| Milestone | Sprint | Week | Success Criteria | Gate |
|---|---|---|---|---|
| **M1: Integrated Foundation** | 1 | 2 | Internal n8n connected to Enterprise PG & S3; Web App Console scaffold | Demo: Web App creates PG record via API |
| **M2: Data-Driven Planning** | 2 | 4 | Spark/Iceberg LTV views created; Claude logic generates campaign plans | Demo: high-LTV segment targeting |
| **M3: Creative & Deploy** | 3 | 6 | S3-linked creative pipeline; Facebook API live; HITL Slack flow | Demo: S3 asset → Facebook live |
| **M4: Monitoring & Scale** | 4 | 8 | Automated monitoring active; full loop finished; operational runbook | Hand-off to UA team |
| **M5: Full Engine** | 5 | 10 | Analytics + optimization + closed-loop feedback | Hand-off complete |
| **M6: pLTV Foundation** | 6 | 12 | Branch A: D7-D60 correlation validated, feature pipeline live. Branch B: SDKs integrated | Both branches have data flowing |
| **M7: pLTV Models Live** | 7 | 14 | Branch A: BG-NBD + XGBoost scoring daily. Branch B: Axon ROAS campaigns running | pLTV scores available |
| **M8: pLTV Activated** | 8 | 16 | Branch A: Synthetic Events to Meta/Google/TikTok. Branch B: Validated vs actual D30 | pLTV feeding bidding |

---

## Sprint Breakdown

### Sprint 1: Enterprise Integration (Weeks 1-2)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 1.1 | Setup internal n8n instance + authentication (SSO) | Eng | 1d | None | Working n8n with SSO |
| 1.2 | Create `ua_engine` schema/tables in Enterprise PostgreSQL | Eng | 1d | 1.1 | State DB ready |
| 1.3 | Configure S3/MinIO bucket & media serving endpoints | Eng | 1d | 1.1 | Creative vault ready |
| 1.4 | Scaffold Web App Console with REST API to n8n/PG | Eng | 1d | 1.2 | Web App Console connected to backend |
| 1.5 | Connect AppsFlyer real-time feed to internal n8n | Eng | 1d | 1.1 | Events flowing to PG |
| 1.6 | Setup Facebook Ads API credentials in vault | Eng | 0.5d | 1.1 | Verified API access |
| 1.7 | Integration testing: Web App → backend connectivity | Eng | 1d | 1.4 | CRUD test passed |
| 1.8 | Validate AppsFlyer data schemas (`install` + `in_app_event` tables) per [`game-publishing-data-architect`](../../.agents/skills/Analytics%20%26%20Data/game-publishing-data-architect/SKILL.md) skill | DE | 0.5d | 1.5 | Schema validated |
| 1.9 | Validate 4 in-game log schemas (Register/Login/Logout/Recharge) per skill | DE | 0.5d | 1.5 | Logs flowing correctly |

**Sprint 1 Total**: ~7.5 person-days

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

### Sprint 2: Creative Pipeline + Facebook Deploy (Weeks 3-4)

| # | Task | Owner | Est. | Dependencies | Deliverable |
|---|---|---|---|---|---|
| 2.1 | Integrate Creatify API in n8n (video generation) | Eng | 1.5d | 1.7 | n8n node: brief → video asset |
| 2.2 | Integrate OpenArt.ai API in n8n (image generation) | Eng | 1d | 1.7 | n8n node: brief → image asset |
| 2.3 | Build Creative Production workflow (parallel GenAI calls) | Eng | 1.5d | 2.1, 2.2 | Workflow: brief → 10 variants stored |
| 2.4 | Build Facebook Campaign Deployment workflow | Eng | 2d | 1.2 | Workflow: assets + config → live Facebook campaign |
| 2.5 | Build creative quality gate (basic scoring before deploy) | Eng | 1d | 2.3 | Filter: reject bad outputs |
| 2.6 | Build HITL notification workflow (Slack) | Eng | 1d | None | Slack: plan review → approve/reject buttons |
| 2.7 | Set up file storage for creative assets | Eng | 0.5d | 1.1 | Docker volume mounted |
| 2.8 | End-to-end test: brief → creatives → Facebook live | Eng + UA | 1.5d | 2.4, 2.6 | First AI-generated campaign live |

**Sprint 2 Total**: ~10 person-days
**Sprint 2 Cost**: ~$500 Facebook ad spend for pilot

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
| 4.1 | Configure TikTok Ads API credentials in n8n | Eng | 0.5d | M3 approved | Verified TikTok API connection |
| 4.2 | Configure Google Ads API credentials in n8n | Eng | 0.5d | M3 approved | Verified Google API connection |
| 4.3 | Build TikTok Campaign Deployment workflow | Eng | 2d | 4.1 | Workflow: assets → TikTok campaign live |
| 4.4 | Build Google Campaign Deployment workflow | Eng | 2d | 4.2 | Workflow: assets → Google campaign live |
| 4.5 | Build Spend Data Pull workflow (all 3 platforms, 2-4x/day) | Eng | 1.5d | 4.1, 4.2 | CRON: pull spend → PostgreSQL |
| 4.6 | Extend Monitor agent to cover all 3 platforms | Eng | 1d | 4.5 | Multi-platform monitoring |
| 4.7 | Build Token Refresh workflows (TikTok, Google OAuth) | Eng | 1d | 4.1, 4.2 | Auto-refresh for all platforms |
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

### Sprint 6: pLTV Data Foundation — Layers 1-2 (Weeks 11-12)

> **Complexity**: ⭐⭐⭐ Medium-High | **Both branches in parallel** | **Critical path: Feature Store**

#### Branch A: Internal pLTV (Layers 1-2: Ingestion + Feature Store)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 6A.1 | Feasibility: D7 vs D60 revenue correlation analysis | L2 | DA | 2d | ⭐ | Spark data | Correlation report (r > 0.7 = proceed) |
| 6A.2 | Create Iceberg schema: `raw.appsflyer_events`, `raw.game_telemetry` (per skill Section 3-4 schemas) | L1 | DE | 1d | ⭐ | Spark cluster | Landing zone tables live |
| 6A.3 | Build AppsFlyer → Iceberg ingestion pipeline (hourly) — validate against skill `install` + `in_app_event` schemas | L1 | DE | 2d | ⭐⭐ | 6A.2 | Events flowing to Lakehouse |
| 6A.4 | Build D0-D3 feature extraction PySpark job | L2 | DE | 3d | ⭐⭐⭐ | 6A.3 | `features.d0_d3` table with 11 features |
| 6A.5 | Build behavioral features table (RFM for lifetimes) | L2 | DE | 1d | ⭐⭐ | 6A.3 | `features.behavior` table live |
| 6A.6 | Validate feature quality: null rates, distribution checks | L2 | DA | 1d | ⭐ | 6A.4, 6A.5 | Feature quality report |

#### Branch B: External Vendors (SDK Integration)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 6B.1 | Integrate AppLovin SDK into game build (staging) | — | DE | 2d | ⭐⭐ | M5 | SDK events visible in Axon dashboard |
| 6B.2 | Integrate Appodeal SDK into game build | — | DE | 1d | ⭐ | M5 | AutoBid receiving events |
| 6B.3 | Configure MMP event forwarding (4 required events) | — | DE | 1d | ⭐⭐ | 6B.1, 6B.2 | AppsFlyer → both vendors verified |
| 6B.4 | Verify event receipt + data quality in vendor dashboards | — | DA | 1d | ⭐ | 6B.3 | Screenshot proof + event count validation |

**Sprint 6 Total**: ~15 person-days | **Risk**: 6A.1 feasibility check may fail → pivot to heuristic multipliers

---

### Sprint 7: pLTV Models + Vendor Campaigns — Layers 3-4 (Weeks 13-14)

> **Complexity**: ⭐⭐⭐⭐ High | **Most technically challenging sprint** | **Critical path: Model Training + Validation**

#### Branch A: Internal pLTV (Layers 3-4: Training + Registry)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 7A.1 | Train BG-NBD + Gamma-Gamma model (Jupyter → validated) | L3 | DA | 2d | ⭐⭐ | 6A.5 | Trained models with MAE/RMSE metrics |
| 7A.2 | Train XGBoost payer classifier (D0-D3 → payer Y/N) | L3 | DA | 2d | ⭐⭐⭐ | 6A.4 | Classifier: AUC > 0.75 target |
| 7A.3 | Build Prophet revenue forecasting (cohort curves D7→D90) | L3 | DA | 1.5d | ⭐⭐ | 6A.4 | Revenue curves per acquisition channel |
| 7A.4 | Set up model registry (S3 artifacts + PG metadata) | L4 | DE | 1.5d | ⭐⭐ | — | `model_registry` table + S3 bucket |
| 7A.5 | Build combined pLTV scoring function (3 models → 1 score) | L3-4 | DE + DA | 2d | ⭐⭐⭐ | 7A.1-7A.4 | `compute_pltv()` validated against holdout |
| 7A.6 | Validate pLTV scores against actual D30 revenue | L4 | DA | 1.5d | ⭐⭐ | 7A.5 | Accuracy report: predicted vs actual |

#### Branch B: External Vendors (Campaign Configuration)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 7B.1 | Create Axon 2.0 ROAS campaign (creatives + targeting) | — | DE + UA | 2d | ⭐⭐ | 6B.4 | Live Axon campaign with ROAS target |
| 7B.2 | Configure Appodeal AutoBid rules (budget, pacing) | — | DE + UA | 1d | ⭐ | 6B.4 | AutoBid actively managing bids |
| 7B.3 | Build vendor monitoring workflow in n8n | — | DE | 1.5d | ⭐⭐ | 7B.1 | n8n pulls vendor metrics → PG hourly |

**Sprint 7 Total**: ~15 person-days | **Risk**: XGBoost AUC may be low → try LightGBM or add more features

---

### Sprint 8: Production Activation — Layers 5-7 (Weeks 15-16)

> **Complexity**: ⭐⭐⭐⭐ High | **Activation + Validation** | **Critical path: S2S integration + A/B test**

#### Branch A: Internal pLTV (Layers 5-7: Scoring → Activation → Retraining)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 8A.1 | Productionize scoring as daily PySpark batch job | L5 | DE | 2d | ⭐⭐⭐ | 7A.5 | `scores.pltv_daily` updated daily at 03:00 UTC |
| 8A.2 | Build drift detection monitor (KL divergence + alerts) | L5 | DE | 1d | ⭐⭐ | 8A.1 | Slack alert on score distribution drift |
| 8A.3 | Build S2S sender: Meta CAPI (Synthetic Purchase events) | L6 | DE | 2d | ⭐⭐⭐ | 8A.1 | pLTV → Meta as `valueToSum` Purchase events |
| 8A.4 | Build S2S sender: Google Offline Conversion Import | L6 | DE | 1.5d | ⭐⭐ | 8A.1 | pLTV → Google Ads conversion values |
| 8A.5 | Build S2S sender: TikTok Events API v2 | L6 | DE | 1d | ⭐⭐ | 8A.1 | pLTV → TikTok CompletePayment events |
| 8A.6 | Run A/B test: pLTV-signal campaigns vs baseline | L6 | DA + UA | 2d | ⭐⭐⭐ | 8A.3 | ROAS lift measurement report |
| 8A.7 | Build retraining pipeline (weekly trigger via n8n) | L7 | DE | 1.5d | ⭐⭐⭐ | 8A.1 | Auto-retrain with champion/challenger swap |
| 8A.8 | Build Kaplan-Meier retention curves per channel | L3 | DA | 1d | ⭐⭐ | 6A.5 | Retention analysis by source/geo |

#### Branch B: External Vendors (Validation + Benchmarking)

| # | Task | Layer | Owner | Est. | Complexity | Dependencies | Deliverable |
|---|---|---|---|---|---|---|---|
| 8B.1 | Compare vendor LTV predictions vs actual D30/D60 | — | DA | 2d | ⭐⭐ | 7B.1 (2+ wks data) | Vendor accuracy report |
| 8B.2 | Benchmark: Branch A pLTV vs Branch B vendor accuracy | — | DA | 1.5d | ⭐⭐⭐ | 8A.6, 8B.1 | Head-to-head comparison report |
| 8B.3 | Write pLTV operational runbook (both branches) | — | DE + DA | 1d | ⭐ | All | Production runbook |

**Sprint 8 Total**: ~16.5 person-days | **Risk**: S2S may break BAU reporting → use separate ad account

---

### Phase Complexity Summary

| Sprint | Phase | Layers | Complexity | Person-Days | Critical Risk |
|---|---|---|---|---|---|
| **1-5** | Core UA Engine | — | ⭐⭐ Medium | ~45d | Review Gate at Week 6 |
| **6** | Data Foundation | L1-L2 | ⭐⭐⭐ Medium-High | ~15d | D7-D60 correlation may be weak |
| **7** | Model Training | L3-L4 | ⭐⭐⭐⭐ High | ~15d | XGBoost AUC may underperform |
| **8** | Production Activation | L5-L7 | ⭐⭐⭐⭐ High | ~16.5d | S2S breaks BAU reporting |
| **Total** | | **7 Layers** | | **~91.5d** | |

---

## Gantt Chart

```mermaid
gantt
    title AI Agentic UA Engine — 16-Week Implementation
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Sprint 1: Foundation
    Docker Compose setup           :s1_1, 2026-03-17, 2d
    Facebook API config             :s1_2, after s1_1, 0.5d
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
    Facebook Deploy workflow        :s2_4, after m1, 2d
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
    Closed-loop feedback           :s5_5, after s5_1, 1d
    Stress testing                 :s5_6, after s5_5, 1d
    Operational runbook            :s5_7, after s5_6, 1d
    Handoff                        :s5_8, after s5_7, 1.5d
    M5: Full Engine                :milestone, m5, after s5_8, 0d

    section Sprint 6: pLTV Feasibility (Branch A + B)
    D7-D60 correlation analysis    :s6a1, after m5, 2d
    D0-D3 feature pipeline         :s6a2, after s6a1, 3d
    Iceberg transaction table      :s6a3, after s6a2, 1d
    BG-NBD + Gamma-Gamma model     :s6a4, after s6a3, 2d
    AppLovin SDK integration       :s6b1, after m5, 2d
    Appodeal SDK integration       :s6b2, after s6b1, 1d
    MMP event forwarding           :s6b3, after s6b2, 1d
    Verify vendor dashboards       :s6b4, after s6b3, 1d
    M6: pLTV Foundation            :milestone, m6, after s6a4, 0d

    section Sprint 7: Model Build + Campaign Config
    XGBoost payer classifier       :s7a1, after m6, 2d
    Prophet revenue forecast       :s7a2, after m6, 1.5d
    pLTV scoring pipeline          :s7a3, after s7a1, 2d
    Validate pLTV vs actual        :s7a4, after s7a3, 1.5d
    Axon ROAS campaign             :s7b1, after m6, 2d
    AutoBid budget rules           :s7b2, after s7b1, 1d
    Vendor monitoring in n8n       :s7b3, after s7b1, 1.5d
    M7: pLTV Models Live           :milestone, m7, after s7a4, 0d

    section Sprint 8: Activation + Validation
    Meta CAPI S2S sender           :s8a1, after m7, 2d
    Google S2S sender              :s8a2, after s8a1, 1.5d
    TikTok S2S sender              :s8a3, after s8a2, 1d
    A/B test pLTV vs baseline      :s8a4, after s8a1, 2d
    KM retention curves            :s8a5, after m7, 1.5d
    Vendor LTV vs actual D30       :s8b1, after m7, 2d
    Branch A vs B benchmark        :s8b2, after s8a4, 1.5d
    pLTV operational runbook       :s8b3, after s8b2, 1d
    M8: pLTV Activated             :crit, milestone, m8, after s8b3, 0d
```

---

## Assumptions

1. Team has existing n8n and Dify experience (no ramp-up time)
2. AppsFlyer data pipeline is stable and delivering real-time events
3. Facebook Ads API credentials available for Sprint 1
4. GenAI tool API keys obtainable within Sprint 1 timeframe
5. UA manager available ~4 hrs/week for reviews and HITL testing
6. Company server available for production deployment by Sprint 3
7. $500/week ad spend budget approved for pilot (Sprints 2-5)
8. **[pLTV]** Data analyst + data engineer available starting Sprint 6
9. **[pLTV]** 2-3 years of historical player transaction data accessible in Spark/Iceberg
10. **[pLTV]** Game client team can integrate AppLovin + Appodeal SDKs (Branch B)

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
| Facebook API rate limit issues | 2 | Can’t test deployment | Use production API with minimal spend ($5/day) |
| Monitor agent hallucination | 3 | Wrong budget decisions | Anti-hallucination guardrails built in Sprint 3 |
| Review Gate fails | 3 | Project pivot or cancellation | Early signal monitoring from Sprint 2 data |
| Google API access denied | 4 | Can't expand to Google | Apply early (Sprint 1), fallback to Basic tier with batching |
| Stress test reveals bottlenecks | 5 | Need architecture changes | n8n queue mode, Redis scaling |
| **[pLTV]** D7-D60 correlation weak | 6 | Branch A model unreliable | Extend observation window to D14, try alternative features |
| **[pLTV]** SDK integration conflicts | 6 | Branch B blocked | Isolate SDK testing to staging build |
| **[pLTV]** Synthetic Events break BAU reporting | 8 | Meta reporting inaccurate | Run in separate ad account first, get business sign-off |
| **[pLTV]** Audience size < 4M for Meta VO | 8 | Can't run VO campaigns | Start with AEO (>2M threshold), scale to VO later |

## Definition of Done

A task is "done" when:
- [ ] Code committed and working in Docker environment
- [ ] Integration test passing (end-to-end workflow)
- [ ] Audit logging verified (decision trail in PostgreSQL)
- [ ] Error handling tested (API failure → graceful recovery)
- [ ] Documentation updated (workflow description in runbook)
