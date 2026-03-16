# Architecture: {{PROJECT_NAME}}

## Overview

{{1-paragraph architecture philosophy: monolith vs. services, sync vs. async, cloud-native vs. hybrid, key design principles}}

---

## C4 Level 1: System Context

> How the system fits into the broader environment.

```mermaid
C4Context
    title System Context — {{PROJECT_NAME}}

    Person(user, "{{User}}", "{{description}}")
    System(system, "{{PROJECT_NAME}}", "{{core purpose}}")
    System_Ext(ext1, "{{External System}}", "{{purpose}}")

    Rel(user, system, "{{uses}}")
    Rel(system, ext1, "{{integrates}}")
```

---

## C4 Level 2: Container Diagram

> Major deployable units and how they communicate.

### Phase 1: MVP

**Design Goals**: Fastest path to a working system. Validate core assumptions. Minimize cost.

```mermaid
C4Container
    title Container Diagram — MVP

    Person(user, "{{User}}")

    Container_Boundary(system, "{{PROJECT_NAME}}") {
        Container(web, "{{Web App}}", "{{tech}}", "{{purpose}}")
        Container(api, "{{API Server}}", "{{tech}}", "{{purpose}}")
        ContainerDb(db, "{{Database}}", "{{tech}}", "{{purpose}}")
    }

    System_Ext(ext, "{{External}}", "{{purpose}}")

    Rel(user, web, "Uses", "HTTPS")
    Rel(web, api, "Calls", "REST/JSON")
    Rel(api, db, "Reads/Writes", "SQL")
    Rel(api, ext, "Integrates", "{{protocol}}")
```

| Container | Technology | Purpose | Scaling |
|---|---|---|---|
| {{container}} | {{tech}} | {{purpose}} | {{strategy}} |

**Estimated Cost**: ~${{X}}/mo

---

### Phase 2: Production

**Trigger**: {{When to transition, e.g., "100+ DAU or first paying customer"}}

```mermaid
C4Container
    title Container Diagram — Production

    Person(user, "{{User}}")

    Container_Boundary(edge, "Edge") {
        Container(cdn, "CDN", "{{provider}}", "Static assets, caching")
        Container(lb, "Load Balancer", "{{provider}}", "Traffic distribution")
    }

    Container_Boundary(app, "Application") {
        Container(api, "API Server", "{{tech}}", "Business logic")
        Container(worker, "Worker", "{{tech}}", "Background jobs")
    }

    Container_Boundary(data, "Data") {
        ContainerDb(db, "Primary DB", "{{tech}}", "Persistent storage")
        ContainerDb(cache, "Cache", "{{tech}}", "Session & query cache")
        ContainerQueue(queue, "Queue", "{{tech}}", "Async messaging")
    }

    Rel(user, cdn, "HTTPS")
    Rel(cdn, lb, "Forward")
    Rel(lb, api, "Route")
    Rel(api, db, "Read/Write")
    Rel(api, cache, "Cache")
    Rel(api, queue, "Publish")
    Rel(worker, queue, "Consume")
    Rel(worker, db, "Write")
```

**New over Phase 1**:
- {{addition 1}}
- {{addition 2}}

**Estimated Cost**: ~${{X}}/mo

---

### Phase 3: Scale (🔴 Large only)

**Trigger**: {{When to transition, e.g., "10K+ DAU or p95 > 500ms"}}

```mermaid
C4Container
    title Container Diagram — Scale

    Person(user, "Users")

    Container_Boundary(edge, "Edge") {
        Container(dns, "DNS", "{{provider}}")
        Container(cdn, "CDN", "{{provider}}")
        Container(waf, "WAF", "{{provider}}")
    }

    Container_Boundary(compute, "Compute - Auto-Scaled") {
        Container(api1, "API Pod 1..N", "{{tech}}")
        Container(worker1, "Worker Pod 1..N", "{{tech}}")
    }

    Container_Boundary(data, "Data - Replicated") {
        ContainerDb(primary, "Write DB", "{{tech}}")
        ContainerDb(replica, "Read Replicas", "{{tech}}")
        ContainerDb(cache, "Cache Cluster", "{{tech}}")
        ContainerQueue(queue, "Message Queue", "{{tech}}")
    }

    Rel(user, dns, "Resolve")
    Rel(dns, cdn, "Route")
    Rel(cdn, waf, "Filter")
    Rel(waf, api1, "Forward")
    Rel(api1, primary, "Write")
    Rel(api1, replica, "Read")
    Rel(api1, cache, "Cache")
    Rel(api1, queue, "Publish")
    Rel(worker1, queue, "Consume")
```

**Scaling Strategy**:
- {{how each component scales — horizontal/vertical, auto-scaling triggers}}

**Estimated Cost**: ~${{X}}/mo

---

## C4 Level 3: Component Diagram (Key Containers)

> Internal structure of the most critical container(s).

```mermaid
C4Component
    title Component Diagram — {{Key Container}}

    Container_Boundary(api, "{{Container Name}}") {
        Component(auth, "Auth Module", "{{tech}}", "Authentication & authorization")
        Component(core, "Core Logic", "{{tech}}", "Business rules")
        Component(repo, "Data Access", "{{tech}}", "Repository pattern")
        Component(events, "Event Publisher", "{{tech}}", "Domain events")
    }

    ContainerDb(db, "Database", "{{tech}}")
    ContainerQueue(queue, "Queue", "{{tech}}")

    Rel(auth, core, "Validates")
    Rel(core, repo, "Queries")
    Rel(core, events, "Publishes")
    Rel(repo, db, "SQL")
    Rel(events, queue, "Publish")
```

---

## Sequence Diagrams

### {{Key Flow 1, e.g., "User Authentication"}}

```mermaid
sequenceDiagram
    actor User
    participant Web as Web App
    participant API as API Server
    participant Auth as Auth Provider
    participant DB as Database

    User->>Web: {{action}}
    Web->>API: {{request}}
    API->>Auth: {{verify}}
    Auth-->>API: {{response}}
    API->>DB: {{query}}
    DB-->>API: {{result}}
    API-->>Web: {{response}}
    Web-->>User: {{result}}
```

### {{Key Flow 2, e.g., "Data Processing Pipeline"}}

```mermaid
sequenceDiagram
    {{add second key flow}}
```

---

## Security Architecture

### Authentication & Authorization

| Concern | Approach |
|---|---|
| **AuthN** | {{e.g., OAuth 2.0 + PKCE via Auth0}} |
| **AuthZ** | {{e.g., RBAC with role hierarchy}} |
| **API Security** | {{e.g., JWT bearer tokens, rate limiting, CORS}} |
| **Session Management** | {{e.g., httpOnly secure cookies, 24h TTL}} |

### Data Protection

| Layer | Encryption | Standard |
|---|---|---|
| In Transit | TLS 1.3 | {{compliance}} |
| At Rest | AES-256 | {{compliance}} |
| Secrets | {{vault/KMS}} | {{compliance}} |

### Threat Model Summary

| Threat | Impact | Mitigation |
|---|---|---|
| {{threat}} | {{impact}} | {{control}} |

---

## Observability Strategy

| Pillar | Tool | What to Capture |
|---|---|---|
| **Logging** | {{tool}} | Structured logs: request ID, user ID, error traces |
| **Metrics** | {{tool}} | Latency, throughput, error rate, saturation (RED/USE) |
| **Tracing** | {{tool}} | Distributed traces across service boundaries |
| **Alerting** | {{tool}} | SLO breaches, error spikes, resource exhaustion |

### SLO Targets

| Service | SLI | SLO Target |
|---|---|---|
| {{service}} | {{e.g., latency p99}} | {{e.g., < 500ms}} |
| {{service}} | {{e.g., availability}} | {{e.g., 99.9%}} |

---

## Deployment Topology

```mermaid
graph LR
    subgraph "Dev"
        Dev_App[App] --> Dev_DB[(DB)]
    end
    subgraph "Staging"
        Stg_App[App] --> Stg_DB[(DB)]
    end
    subgraph "Production"
        Prod_App[App x N] --> Prod_DB[(DB Primary)]
        Prod_DB --> Prod_Replica[(Read Replica)]
    end
    Dev -->|PR merge| Staging
    Staging -->|Release tag| Production
```

| Environment | Purpose | Infra | Data |
|---|---|---|---|
| **Development** | Feature development | {{minimal}} | Seed data |
| **Staging** | Integration testing, UAT | {{mirrors prod}} | Anonymized prod data |
| **Production** | Live traffic | {{full scale}} | Real data |

### IaC Approach

{{e.g., Terraform for infrastructure, Docker Compose for local dev, Kubernetes manifests for production}}

---

## Data Architecture

### Entity-Relationship Diagram

```mermaid
erDiagram
    {{ENTITY_A}} ||--o{ {{ENTITY_B}} : "{{relationship}}"
    {{ENTITY_B}} ||--|{ {{ENTITY_C}} : "{{relationship}}"
```

### Key Data Flows

{{Description of how data moves through the system — ingestion, transformation, storage, retrieval, archival}}

---

## API Design

### Key Endpoints

| Method | Path | Description | Auth |
|---|---|---|---|
| {{GET/POST/PUT/DELETE}} | {{/api/v1/resource}} | {{purpose}} | {{required/public}} |

### Response Shapes

```typescript
interface {{EntityName}} {
  {{field}}: {{type}};
}
```

---

## Architecture Decision Records

> Key ADRs are logged in `findings.md`. Below are the most critical ones summarized.

| ADR# | Decision | Status | Rationale |
|---|---|---|---|
| {{ADR-001}} | {{decision}} | ✅ Accepted | {{why}} |
