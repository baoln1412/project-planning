# AI Implementation Brief: {{PROJECT_NAME}}

> **This is the handoff document for AI coding agents (Antigravity / Claude Code).** It contains everything needed to start building. Read this document completely before writing any code.

---

## 1. Project Context

{{1-paragraph summary: what system is being built, for whom, why it matters, and the key constraint that shapes the architecture}}

**Planning Documents**: Full project folder at `projects/{{project-slug}}/`

---

## 2. Repository Scaffold

Create this exact folder structure:

```
{{project-slug}}/
├── src/
│   ├── {{component-1}}/
│   │   ├── {{subcomponent}}.{{ext}}
│   │   └── ...
│   ├── {{component-2}}/
│   └── {{shared/common}}/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
├── scripts/
├── .env.example
├── .gitignore
├── README.md
└── {{package-manifest}}
```

---

## 3. Tech Stack

| Layer | Technology | Version | Package |
|---|---|---|---|
| {{layer}} | {{tech}} | {{version}} | {{npm/pip/go package}} |

### Setup Commands

```bash
# Initialize project
{{init command}}

# Install dependencies
{{install command}}

# Run development server
{{dev command}}

# Run tests
{{test command}}
```

---

## 4. Data Contracts

> ⛔ These schemas were confirmed during planning. Do not deviate without explicit approval.

### Input Schemas

```typescript
// {{Flow/Endpoint Name}} — Request
interface {{RequestName}} {
  {{field}}: {{type}};
}
```

### Output Schemas

```typescript
// {{Flow/Endpoint Name}} — Response
interface {{ResponseName}} {
  {{field}}: {{type}};
}
```

---

## 5. Database Schema

```typescript
// {{Entity Name}}
interface {{EntityName}} {
  id: string;
  {{field}}: {{type}};
  createdAt: Date;
  updatedAt: Date;
}
```

### Relationships

```mermaid
erDiagram
    {{ENTITY_A}} ||--o{ {{ENTITY_B}} : "{{relationship}}"
```

### Migrations

{{Describe migration approach: ORM auto-migration, manual SQL scripts, etc.}}

---

## 6. Architecture Reference

### Key Patterns

- **{{Pattern 1, e.g., Repository Pattern}}** — {{where and why}}
- **{{Pattern 2, e.g., Event-Driven}}** — {{where and why}}
- **{{Pattern 3, e.g., Middleware Pipeline}}** — {{where and why}}

### Component Responsibilities

| Component | Responsibility | Depends On |
|---|---|---|
| {{component}} | {{what it does}} | {{dependencies}} |

### Interaction Rules

- {{rule 1, e.g., "All database access goes through repository layer, never direct queries"}}
- {{rule 2, e.g., "External API calls must go through a service client with retry/circuit-breaker"}}
- {{rule 3, e.g., "Business logic lives in service layer, controllers are thin"}}

---

## 7. API Design

| Method | Path | Description | Auth | Request | Response |
|---|---|---|---|---|---|
| {{METHOD}} | {{/api/v1/path}} | {{purpose}} | {{required/public}} | `{{RequestType}}` | `{{ResponseType}}` |

### Error Response Shape

```typescript
interface ErrorResponse {
  error: {
    code: string;       // Machine-readable error code
    message: string;    // Human-readable message
    details?: unknown;  // Additional context
  };
}
```

---

## 8. Implementation Sequence

Build in this order. Each step should result in a working (if incomplete) system.

| Order | What to Build | Why This Order | Dependencies |
|---|---|---|---|
| 1 | {{component/feature}} | {{rationale}} | — |
| 2 | {{component/feature}} | {{rationale}} | Step 1 |
| 3 | {{component/feature}} | {{rationale}} | Step 1, 2 |
| 4 | {{component/feature}} | {{rationale}} | Step 2 |

---

## 9. Testing Requirements

| What to Test | Type | Priority | Notes |
|---|---|---|---|
| {{component/flow}} | Unit / Integration / E2E | P1/P2/P3 | {{approach}} |

### Test Data Strategy

{{How to generate test data: fixtures, factories, seed scripts}}

---

## 10. Environment Configuration

### Environment Variables

| Variable | Description | Required | Default | Example |
|---|---|---|---|---|
| `{{VAR_NAME}}` | {{purpose}} | Yes/No | {{default}} | {{example}} |

### External Services

| Service | Purpose | Setup | Docs |
|---|---|---|---|
| {{service}} | {{why needed}} | {{how to get credentials}} | {{link}} |

### Secrets Management

{{How secrets are stored and accessed: env vars, vault, cloud secrets manager}}

---

## 11. Coding Conventions

### Naming

| Element | Convention | Example |
|---|---|---|
| Files | {{convention}} | {{example}} |
| Functions | {{convention}} | {{example}} |
| Classes | {{convention}} | {{example}} |
| Database tables | {{convention}} | {{example}} |
| API endpoints | {{convention}} | {{example}} |

### Error Handling

{{Standard error handling pattern: try/catch, Result type, error codes, logging}}

### Logging

{{Structured logging format, log levels, what to log, what NOT to log (PII, secrets)}}

---

## 12. Out of Scope

> Do NOT build these. They are explicitly excluded from this implementation.

- ❌ {{excluded feature 1}}
- ❌ {{excluded feature 2}}
- ❌ {{excluded capability}}

---

## 13. References

- [Project Overview](./overview.md)
- [Research](./research.md)
- [Findings & ADRs](./findings.md)
- [Tech Stack](./tech-stack.md)
- [Architecture](./architecture.md)
- [Implementation Plan](./implementation-plan.md)
- [Business Proposal](./business-proposal.md)
