---
name: game-publishing-data-architect
description: "Designs data architecture, integration pipelines, and metric frameworks for game publishing companies. Covers log requirements, S2S integrations, retention/monetization metrics, and regulatory compliance for the mobile/online gaming industry."
allowed-tools: Read Write Glob
metadata:
  author: vngbaoln2
  version: "1.0"
  industry: game-publishing
---

# Game Publishing Data Architect

## When to Use This Skill

Use this skill when you need to:
- Design or review data architecture for a game publishing company
- Define log requirements for new game integrations (in-game, SDK, marketing, audit)
- Plan data pipelines for game analytics (ingestion → storage → BI)
- Standardize game industry metrics (UA, engagement, monetization, retention)
- Implement S2S integrations with ad platforms (AppsFlyer, Facebook, Firebase, Clevertap)
- Ensure regulatory compliance (financial audit logs, minor playing time restrictions)

**DO NOT** use this skill for game development, game design, or generic data engineering unrelated to game publishing. This is specific to the operational data needs of a game publisher.

---

## Core Principle

GAME DATA MUST FLOW IN REAL-TIME, BE AUDITABLE FOR COMPLIANCE, AND PRODUCE METRICS THAT DIRECTLY DRIVE UA SPEND, MONETIZATION DECISIONS, AND REGULATORY REPORTING. EVERY LOG HAS A PURPOSE — EVERY METRIC HAS A FORMULA.

---

## Domain Knowledge Base

### 1. Standard Data Architecture & Tech Stack

The game publishing industry standardizes data integration using the following layers:

#### Data Processing Flow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────────────┐     ┌─────────────────────┐
│  DATA SOURCES   │     │      ETL        │     │    DATA WAREHOUSE       │     │    BUSINESS         │
│                 │ ──→ │                 │ ──→ │                         │ ──→ │    INTELLIGENCE      │
│  Marketing      │     │  Extract        │     │  Analytics Query        │     │  Dashboards         │
│  Event Tracking │     │  Transform      │     │  & Reporting            │     │  Reports            │
│  In-game Data   │     │  Load           │     │  (PostgreSQL / Iceberg) │     │  (Metabase)         │
│  Platform Data  │     │                 │     │                         │     │                     │
└─────────────────┘     └─────────────────┘     └─────────────────────────┘     └─────────────────────┘
```

#### Required Data Sources for Game Integration

| # | Data Source | Provider / Platform | Data Provided |
|---|-----------|-------------------|---------------|
| 1 | **Marketing Data** | Facebook Ads, Google Ads, TikTok Ads | Cost, Campaign Info, Ads Performance |
| 2 | **Event Tracking** | AppsFlyer (MMP) | Installs, In-app Events, Attribution |
| 3 | **In-game Data** | Game Server (via Kafka / Rsync) | Register, Login, Logout, Recharge, CCU |
| 4 | **Platform Data** | VNG SDK | SDK Login + Billing/Payment |

> **Integration rule**: All 4 data sources must be connected before a game can go live on dashboards. Missing any source creates blind spots in UA, monetization, or compliance reporting.

#### Data Ingestion & Streaming
| Method | Use Case | Latency | Details |
|--------|----------|---------|---------|
| **Kafka** | Real-time in-game logs (Register, Login, Logout, Recharge) | Real-time | Primary ingestion for player event tracking |
| **Rsync** | Alternative near real-time tracking | 5-10 minute intervals | Fallback when Kafka is unavailable |

#### Data Storage
| Layer | Purpose | Format |
|-------|---------|--------|
| **Warm Storage** | Daily synchronized in-game, SDK, finance, and marketing logs | Raw logs |
| **Google Cloud Storage (GCS)** | Data Locker raw reporting data | `Parquet` files with `Snappy` compression |

#### Databases & Query Engines
| Technology | Role | Status |
|-----------|------|--------|
| **PostgreSQL** | Standard relational DB hosting aggregated Game Databases | Current — production |
| **Iceberg Tables + Trino** | Large-scale game data with time-travel and schema evolution | Planned — future replacement for Postgres |
| **ClickHouse** | Concurrent User (CCU) API integrations | Specialized — high-throughput time-series |

#### BI & Pipelines
| Tool | Purpose |
|------|---------|
| **Metabase** | Primary BI tool for game studios (custom reports, SQL, data export) |
| **DAG BI Pipeline** | Build, orchestrate, and deploy game data pipelines (staging + prod) |

#### Orchestration & Monitoring
| Tool | Purpose |
|------|---------|
| **GitLab** | Game layout configuration repositories |
| **Monitoring Hub** | Health tracking for daily and real-time raw log pipelines |

#### S2S Integrations
- Direct server-side integrations send payment data (IAP, Web Purchases) to:
  - **AppsFlyer** (attribution), **Firebase** (analytics), **Facebook/Meta** (CAPI), **Clevertap** (engagement)

---

### 2. Log Requirements

When integrating a new game, ensure the following logs are provided:

#### 2.1 Basic In-Game Logs (MANDATORY)
| Event | Transmission | Priority |
|-------|-------------|----------|
| **Register** | Kafka (real-time) or Rsync (5-10 min) | P0 — Critical |
| **Login** | Kafka (real-time) or Rsync (5-10 min) | P0 — Critical |
| **Logout** | Kafka (real-time) or Rsync (5-10 min) | P0 — Critical |
| **Recharge** | Kafka (real-time) or Rsync (5-10 min) | P0 — Critical |

#### 2.2 Third-Party & Marketing Logs
- **Marketing SDK Logs**: Collected via API push and Data Locker workflows
- **ID Mapping**: Internal in-game data ↔ Marketing data via `DeviceID` or `SDK UserID`
- **Ad Account Setup Rule**: Ad accounts (Facebook, Google, TikTok, ASA) must be provided **≥7 days before campaign launch**
- **Multi-Game Ad Accounts**: Must use strict **campaign naming patterns** to differentiate games sharing the same ad account

#### 2.3 Audit Logs (Financial Compliance)
Required for revenue recognition and standard financial reporting:
- Recharge in-game logs
- SDK Login logs
- Payment Billing logs

#### 2.4 User Online Time Logs (Regulatory Compliance)
For jurisdictions with minor playing time regulations (e.g., max 60 min/game/day, 180 min total):
- **In-app event generated every minute** while user is online
- Data sources: SDK Login Info + Account Login Logs + Marketing SDK API pushes

---

### 3. AppsFlyer Data Schema

#### `in_app_event` — Event Tracking Data

**Attribution & Timing**

| Field | Category | Data Type |
|-------|----------|-----------|
| `attributed_touch_type` | Category | Text |
| `attributed_touch_time` | — | DateTimeWithTZ |
| `install_time` | — | DateTimeWithTZ |
| `event_time` | — | DateTimeWithTZ |

**Event Details**

| Field | Category | Data Type |
|-------|----------|-----------|
| `event_name` | Category | Text |
| `event_value` | JSON | Text |
| `event_revenue` | — | Float |
| `event_revenue_currency` | Category | Text |
| `event_revenue_usd` | — | Float |
| `event_source` | Source | Text |
| `is_receipt_validated` | — | Text |

**Campaign & Ad Hierarchy**

| Field | Category | Data Type | Notes |
|-------|----------|-----------|-------|
| `af_prt` | — | Text | Agency/partner |
| `media_source` | Source | Text | e.g., `Facebook Ads`, `googleadwords_int` |
| `af_channel` | Source | Text | Sub-channel |
| `keywords` | — | Text | Search keywords |
| `campaign` | Category | Text | Campaign name |
| `af_c_id` | Category | Text | Campaign ID |
| `af_adset` | Category | Text | Ad set name |
| `af_adset_id` | Category | Text | Ad set ID |
| `af_ad` | Category | Text | Ad name |
| `af_ad_id` | — | Text | Ad ID |
| `af_ad_type` | Category | Text | Ad format |
| `af_site_id` | — | Text | Publisher site ID |
| `af_sub_site_id` | — | Text | Sub-publisher |

**Cost Data**

| Field | Category | Data Type |
|-------|----------|-----------|
| `af_cost_model` | — | Text |
| `af_cost_value` | Cost | Float |
| `af_cost_currency` | — | Text |

**Multi-Touch Attribution (Contributors 1-3)**

| Field Pattern | Category | Data Type |
|--------------|----------|-----------|
| `contributor{1-3}_af_prt` | — | Text |
| `contributor{1-3}_media_source` | Source | Text |
| `contributor{1-3}_campaign` | — | Text |
| `contributor{1-3}_touch_type` | Category | Text |
| `contributor{1-3}_touch_time` | — | DateTimeWithTZ |

> 3 contributor slots available (contributor1, contributor2, contributor3), each with prt, media_source, campaign, touch_type, and touch_time.

**Geo & Network**

| Field | Category | Data Type |
|-------|----------|-----------|
| `region` | Category | Text |
| `country_code` | Country | Text |
| `state` | — | Text |
| `city` | City | Text |
| `postal_code` | Zip Code | Text |
| `dma` | — | Text |
| `ip` | — | Text |
| `wifi` | Category | Text |
| `operator` | — | Text |
| `carrier` | — | Text |
| `language` | Category | Text |

**Device Identifiers**

| Field | Category | Data Type | Notes |
|-------|----------|-----------|-------|
| `appsflyer_id` | — | Text | AppsFlyer's unique device ID |
| `advertising_id` | — | Text | Google Ads ID (Android) |
| `idfa` | — | Text | iOS Identifier for Advertisers |
| `android_id` | — | Text | Android device ID |
| `customer_user_id` | — | Text | Publisher's user ID (maps to in-game `user_id`) |
| `imei` | — | Text | Device IMEI |
| `idfv` | — | Text | iOS Identifier for Vendors |

> **Key mapping**: `customer_user_id` → in-game `user_id` for joining AppsFlyer data with in-game logs.

**Device & App Info**

| Field | Category | Data Type |
|-------|----------|-----------|
| `platform` | Category | Text |
| `device_type` | Category | Text |
| `device_model` | — | Text |
| `os_version` | Category | Text |
| `app_version` | Category | Text |
| `sdk_version` | Category | Text |
| `app_id` | Category | Text |
| `app_name` | Category | Text |
| `bundle_id` | Category | Text |

**Retargeting**

| Field | Category | Data Type |
|-------|----------|-----------|
| `is_retargeting` | Category | Text |
| `retargeting_conversion_type` | Category | Text |
| `attribution_lookback` | Category | Text |
| `af_reengagement_window` | — | Text |
| `is_primary_attribution` | Category | Text |

**Metadata & Misc**

| Field | Category | Data Type |
|-------|----------|-----------|
| `user_agent` | — | Text |
| `original_url` | URL | Text |
| `raw_date` | Category | Text |
| `ds` | — | Date |

#### `install` — Install Attribution Data

The install schema shares most fields with `in_app_event` above. Below documents the **differences only**.

**Fields NOT present in `install`** (present only in `in_app_event`):

| Removed Field | Reason |
|--------------|--------|
| `event_value` | No event payload for installs |
| `event_revenue` | No revenue at install time |
| `event_revenue_currency` | No revenue at install time |
| `event_revenue_usd` | No revenue at install time |
| `event_source` | Not applicable |
| `is_receipt_validated` | No purchase to validate |
| `af_sub_site_id` | Not tracked at install level |
| `http_referrer` | Not tracked at install level |
| `dma` | Not included |

**Fields UNIQUE to `install`** (not in `in_app_event`):

| Field | Category | Data Type | Notes |
|-------|----------|-----------|-------|
| `install_app_store` | — | Text | App store where install occurred (e.g., `Google Play`, `App Store`) |

**Other minor differences**:
- `contributor1_campaign`: Category type in install (vs no field type in in_app_event)
- `is_primary_attribution`: No field type in install (vs Category in in_app_event)

> **Usage**: The `install` table is the source of truth for **Install**, **CPI**, **NRU/Install Rate**, and **attribution** metrics. Join with `in_app_event` on `appsflyer_id` or `customer_user_id` for post-install event analysis.

---

### 4. In-Game Log Schemas

#### 4.0 Basic Fields (Common Across All Logs)

| Field | Description | Data Type | Requirement |
|-------|------------|-----------|-------------|
| `user_id` | Unique user account ID from VNG's SDK | string | Must match SDK logs |
| `role_id` | Unique in-game character ID | string | |
| `role_name` | Character name | string | |
| `role_class` | Character type/class (e.g., Warrior, Mage) | string | |
| `role_level` | Character level/rank | string/int | |
| `client_ip` | User's device IP address | string | |
| `country_code` | Country from user's IP; use ISO 3166-1 alpha-2 | string | `VN`, `TH`, `ID` |
| `server_id` | Unique game server ID | string | |
| `os_platform` | Device OS (Android, iOS) | string | |
| `os_version` | OS version number | string | |
| `device_id` | iOS: IDFA (fallback IDFV). Android: Google Ads ID | string | |
| `device_name` | Device model (e.g., "iPhone 13", "SM-G7810") | string | |
| `login_channel` | Login method from VNG's SDK | string | |

#### 4.1 New Register Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3358892800593829888` |
| `role_id` | Character ID | string | `169` |
| `role_name` | Character name | string | `測測appsf` |
| `role_class` | Character class | string | `SWORD` |
| `role_level` | Character level | string | `1` |
| `register_time` | Role creation timestamp | timestamp | `2025-11-24 15:11:58` |
| `client_ip` | User IP at creation | string | `211.200.24.175` |
| `country_code` | Country at registration | string | `VN` |
| `server_id` | Server ID | string | `1011` |
| `os_platform` | OS platform | string | `iOS` |
| `os_version` | OS version | string | `Android OS 12 / API-32` |
| `device_id` | Device ID | string | `2131a6772e5667857703a73885b6b6cd` |
| `device_name` | Device model | string | `SM-G7810` |

#### 4.2 Login Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3358892800593829888` |
| `role_id` | Character ID | string | `167` |
| `role_name` | Character name | string | `needle` |
| `role_level` | Character level | string | `17` |
| `vip_level` | VIP level | int | `1` |
| `fighting_power` | Character fighting power | int | `1000` |
| `country_code` | Country at login | string | `VN` |
| `login_time` | Login timestamp | timestamp | `2025-12-02 10:22:38` |
| `login_channel` | Login method | string | `Google` |
| `server_id` | Server ID | string | `1011` |
| `client_ip` | User IP at login | string | `211.200.24.175` |
| `os_platform` | OS platform | string | `iOS` |
| `os_version` | OS version | string | `Android OS 12 / API-32` |
| `device_id` | Device ID | string | `2131a6772e5667857703a73885b6b6cd` |
| `device_name` | Device model | string | `SM-G7810` |

#### 4.3 Logout Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3358892800593829888` |
| `role_id` | Character ID | string | `167` |
| `role_name` | Character name | string | `needle` |
| `role_level` | Character level | string | `17` |
| `vip_level` | VIP level | int | `1` |
| `fighting_power` | Character fighting power | int | `1000` |
| `country_code` | Country at logout | string | `VN` |
| `logout_time` | Logout timestamp | timestamp | `2025-12-02 12:22:38` |
| `login_channel` | Login method of the session | string | `Google` |
| `server_id` | Server ID | string | `1011` |
| `client_ip` | User IP at logout | string | `211.200.24.175` |
| `online_time` | Session duration in seconds | number | `30110` |
| `os_platform` | OS platform | string | `iOS` |
| `os_version` | OS version | string | `Android OS 12 / API-32` |
| `device_id` | Device ID | string | `2131a6772e5667857703a73885b6b6cd` |
| `device_name` | Device model | string | `SM-G7810` |

> **Key field**: `online_time` (seconds) — critical for DAU/session calculations and regulatory online time compliance.

#### 4.4 Recharge Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3358892800593829888` |
| `role_id` | Character ID | string | `167` |
| `role_name` | Character name | string | `needle` |
| `role_level` | Character level | int | `17` |
| `vip_level` | VIP level | int | `1` |
| `fighting_power` | Character fighting power | int | `100` |
| `country_code` | Country at recharge | string | `VN` |
| `currency_code` | Transaction currency | string | `VND`, `USD` |
| `transaction_id` | Unique transaction ID from SDK (critical for audits) | string | `1761537858895` |
| `recharged_value` | Actual amount paid by user | number | `0.99` |
| `product_id` | ID of purchased product | string | `com.vng.x.web.crystal.150` |
| `ingame_value` | In-game currency received (optional if same as recharged_value) | number | `2920` |
| `transaction_time` | Transaction completion time | timestamp | `2025-12-03 00:55:22` |
| `server_id` | Server ID | string | `1011` |
| `payment_channel` | Payment platform | string | `IAP`, `Webpay`, `3rd Webpay` |
| `os_platform` | OS platform | string | `android` |
| `device_id` | Device ID | string | `2131a6772e5667857703a73885b6b6cd` |

**Recharge JSON example** (raw payload from game server):
```json
{
  "gameID": "a",
  "country": "VN",
  "amount": "0.99",
  "orderNumber": "1761537858895",
  "appTransID": "f81e24152b620735590f3af66eb27cf2APPTRANS",
  "productID": "com.vng.x.web.crystal.150",
  "roleID": "CBNBW6525",
  "channel": "1",
  "serverID": "1",
  "userID": "3331545738719600640"
}
```

**Field mapping**: `orderNumber` → `transaction_id` | `amount` → `recharged_value` | `productID` → `product_id`

#### 4.5 CCU Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `event_time` | CCU snapshot timestamp (GMT+7), consistent across servers | timestamp | `2025-12-02 01:00:00` |
| `server_id` | Server ID | string | `1011` |
| `online_users` | Number of online users on server | number | `3` |

> Used by ClickHouse for high-throughput time-series queries (ACU, PCU, LCU calculations).

#### 4.6 Money Flow Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3322770528219815936` |
| `role_id` | Character ID | string | `26` |
| `role_name` | Character name | string | `OAO` |
| `server_id` | Server ID | string | `1011` |
| `country_code` | Country code | string | `VN` |
| `transaction_id` | Unique ID for this money action | string | `Receiving money or using money` |
| `money_type` | Type of currency | string | `0` = Gold, `1` = Gold Key |
| `flow_type` | Type of transaction | string | `0` = Receive, `1` = Spend |
| `action_id` | ID describing source/sink (quest, purchase, etc.) | string | `16100033-3502-5610-0-1-1764337684` |
| `money_value` | Amount received or spent | number | `1435` |
| `before_value` | Balance before action | number | `55060` |
| `after_value` | Balance after action | number | `53625` |
| `transaction_time` | Action timestamp | timestamp | `2025-12-03 07:27:11` |

> **Audit rule**: `after_value` must equal `before_value ± money_value` depending on `flow_type`.

#### 4.7 Item Flow Log

| Column | Description | Type | Example |
|--------|------------|------|---------|
| `user_id` | User account ID | string | `3322770528219815936` |
| `role_id` | Character ID | string | `26` |
| `role_name` | Character name | string | `OAO` |
| `server_id` | Server ID | string | `1011` |
| `country_code` | Country code | string | `VN` |
| `transaction_id` | Unique ID for this item action | string | `Receiving an item or using an item` |
| `item_id` | ID of item | string | `502511185532` |
| `item_quantity` | Quantity of item | number | `10` |
| `item_price` | Unit price of item | number | `18.99` |
| `money_value` | In-game currency spent (0 if none) | number | `1435` |
| `money_type` | In-game currency type | string | `66` |
| `transaction_time` | Action timestamp | timestamp | `2025-12-03 07:27:11` |

---

### 4. Metrics Definitions & Calculations

#### General Rules
- **Timezone**: All metrics use **UTC+7** (00:00 → 23:59)
- **Retention method**: Classic **exact-day retention** (Day n = report_date + n), NOT rolling retention
- **Exclusions**: Test server users are excluded from all performance metrics

#### 3.1 User Acquisition Metrics

| Metric | Definition | Formula |
|--------|-----------|---------|
| **Installs** | Total installations (paid + organic), deduplicated by tracker ID. Excludes block installs and reinstalls | Count of unique tracking IDs |
| **Paid / Organic Install** | Categorized by first media source (organic vs non-organic) | Count of respective installs |
| **CTI** | Click-to-Install Rate | `(Installs / Clicks) × 100%` |
| **CPI** | Cost Per Install | `Cost / Installs` |
| **CPN** | Cost Per New Registered User | `Cost / NRU` |
| **NRU** | New Register User — first-time active user, deduplicated by user_id (combines register, login, logout, paying) | Count of unique `user_id` |
| **NRU/Install Rate** | Ratio of new registrations to installs | `(NRU / Installs) × 100%` |

#### 3.2 Engagement Metrics

| Metric | Definition | Formula |
|--------|-----------|---------|
| **DAU / WAU / MAU** | Daily, Weekly, Monthly Active Users — unique users active in period | Unique active users |
| **CCU** | Concurrent Users — unique users online at the exact same moment | Measured at specific timestamps |
| **ACU / PCU / LCU** | Average, Peak, Lowest Concurrent Users per hour | Avg, Max, Min of sampled CCU |
| **New Role** | Unique game characters active for the first time | Counted once at creation time |
| **Active Role** | Unique game characters active in reporting period | Counted once per period |

#### 3.3 Monetization Metrics

| Metric | Definition | Formula |
|--------|-----------|---------|
| **PU / WPU / MPU** | Daily, Weekly, Monthly Paying Users | Unique paying users in period |
| **Paying Rate** | Ratio of paying to active users | `(PU / DAU) × 100%` |
| **Gross Bookings** | Total charged before deductions/refunds (billing system) | Total charged via orders |
| **Revenue** | Value of items successfully delivered (recognized on delivery date) | May differ from Gross Bookings |
| **ARPU / ARPPU** | Avg Revenue Per User / Per Paying User | `Revenue / DAU` and `Revenue / PU` |
| **NPU** | New Paying Users (first-time payment) | Count of unique users |
| **RevNPU / ARPNPU** | Revenue from NPU / Avg Revenue from NPU | `Sum(Rev from NPU)` and `RevNPU / NPU` |
| **NNPU** | New Register & New Paying User — registered AND paid on same day | Count matching both criteria |
| **ROAS(n)** | Return on Ad Spend for n days | `(RevRPI(n) / Marketing Cost) × 100%` |
| **LTV(n)** | Lifetime Value — cumulative revenue of NRU | `RevNRU(n) / NRU` |
| **MKT/Rev** | Marketing Cost to Revenue Ratio | `(Marketing Cost / Revenue) × 100%` |

#### 3.4 Retention & Funnel Metrics

| Metric | Definition | Formula |
|--------|-----------|---------|
| **RR(n)** | New User Retention Rate on exact Day n | `(Ruser(n) / NRU) × 100%` |
| **RP(n)** | Paying Retention Rate of NPU on exact Day n | `(Rpuser(n) / NPU) × 100%` |
| **APR(n)** | Active Retention Rate of NPU | `(Retained Active NPU / Day 0 NPU) × 100%` |
| **CVR {Funnel}** | Conversion Rate for SDK funnel steps (Download CDN, Show Login, Registration) | `(Completed IDs / Total IDs) × 100%` |

---

## Phase 1: Brief

### Required Inputs

| Input | What to Ask | Default |
|-------|------------|---------|
| **Game title & genre** | "What game is being integrated?" | Must be provided |
| **Platform** | "Mobile (iOS/Android), PC, or cross-platform?" | Mobile |
| **Current data state** | "What logs/pipelines already exist?" | Starting fresh |
| **Monetization model** | "IAP, ads, subscription, hybrid?" | IAP + ads |
| **Ad platforms** | "Which platforms for UA? (Facebook, Google, TikTok, ASA)" | Facebook + Google |
| **MMP** | "Which Mobile Measurement Partner? (AppsFlyer, Adjust)" | AppsFlyer |
| **Regulatory region** | "Which market? (affects minor time restrictions)" | SEA |
| **BI requirements** | "Who needs dashboards? What decisions do they make?" | UA + Product teams |

**GATE: Confirm brief and game scope before proceeding.**

---

## Phase 2: Design

### Data Architecture Checklist

For each new game integration, design:

1. **Ingestion Layer** — Kafka topics or Rsync schedule for all 4 base events
2. **ID Mapping Strategy** — How DeviceID / SDK UserID links in-game to marketing data
3. **Storage Layout** — Warm storage partitioning, GCS Data Locker config
4. **Database Schema** — PostgreSQL tables or Iceberg schema for aggregated data
5. **Pipeline Orchestration** — DAG definitions for staging → production
6. **BI Connectivity** — Metabase data source configuration and access grants
7. **S2S Integration** — Payment event forwarding to ad platforms
8. **Compliance Logs** — Audit log schema + online time tracking event

### Data Flow Diagram Template

```
Game Client → Kafka/Rsync → Warm Storage → PostgreSQL/Iceberg
                                ↓
Marketing SDK → API Push/Data Locker → GCS (Parquet/Snappy)
                                ↓
                    DAG Pipeline → Metabase (BI)
                                ↓
                    S2S → AppsFlyer/Firebase/Meta/Clevertap
```
