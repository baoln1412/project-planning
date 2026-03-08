# Architecture: {{PROJECT_NAME}}

## Overview

{{1-paragraph architecture philosophy and approach}}

## Phase 1: Test / MVP

### Design Goals

- Fastest path to a working prototype
- Validate core assumptions
- Minimize infrastructure cost and complexity

### Architecture Diagram

```mermaid
graph LR
    User[👤 User] --> Frontend[Frontend]
    Frontend --> API[API Server]
    API --> DB[(Database)]
```

### Components

| Component | Technology | Purpose |
|---|---|---|
| {{component}} | {{tech}} | {{purpose}} |

### Estimated Cost: ~${{X}}/mo

---

## Phase 2: Production

### Trigger to Transition

{{When to move from Phase 1 → Phase 2, e.g. "100+ daily active users"}}

### Architecture Diagram

```mermaid
graph TB
    subgraph "Client"
        Web[🌐 Web]
    end
    subgraph "Edge"
        CDN[CDN]
        LB[Load Balancer]
    end
    subgraph "App"
        API[API Server]
        Worker[Worker]
    end
    subgraph "Data"
        DB[(Primary DB)]
        Cache[(Cache)]
    end
    Web --> CDN --> LB --> API
    API --> DB & Cache & Worker
```

### New Components

{{What's added over Phase 1}}

### Security Measures

- {{measure 1}}
- {{measure 2}}

### Estimated Cost: ~${{X}}/mo

---

## Phase 3: Scale

### Trigger to Transition

{{When to move from Phase 2 → Phase 3, e.g. "10K+ DAU or response time > 500ms"}}

### Architecture Diagram

```mermaid
graph TB
    subgraph "Edge"
        DNS[DNS] --> CDN[CDN] --> WAF[WAF]
    end
    subgraph "Compute"
        LB[Load Balancer]
        API1[API 1]
        API2[API N]
    end
    subgraph "Data"
        Primary[(Write DB)]
        Replica[(Read Replicas)]
        Cache[(Cache Cluster)]
        Queue[Message Queue]
    end
    WAF --> LB --> API1 & API2
    API1 & API2 --> Primary & Replica & Cache & Queue
```

### Scaling Strategy

{{How each component scales}}

### Performance Optimizations

- {{optimization 1}}
- {{optimization 2}}

### Estimated Cost: ~${{X}}/mo

---

## Data Architecture

### ERD

```mermaid
erDiagram
    USER ||--o{ ORDER : places
    ORDER ||--|{ LINE_ITEM : contains
```

### Key Data Flows

{{Description of how data moves through the system}}
