# Business Proposal: {{PROJECT_NAME}}

> **Document Purpose**: Business stakeholder proposal for executive review and approval.
> **Prepared by**: {{author}}
> **Date**: {{DATE}}
> **Version**: {{version}}

---

## 1. Executive Summary

{{2–3 paragraph overview: what is being built, why it matters to the business, the expected outcome, and high-level timeline. Write in business language — no technical jargon.}}

**Key Highlights**:
- **Problem**: {{1 sentence — the business problem being solved}}
- **Solution**: {{1 sentence — what we're building}}
- **Timeline**: {{expected delivery timeframe}}
- **Investment**: {{total estimated cost range}}
- **Expected Outcome**: {{measurable business impact}}

---

## 2. Business Objectives & Success Criteria

### Objectives

| # | Objective | Alignment |
|---|---|---|
| 1 | {{business objective}} | {{which company goal/OKR this supports}} |
| 2 | {{business objective}} | {{alignment}} |
| 3 | {{business objective}} | {{alignment}} |

### Success Criteria

| Metric | Current State | Target | Measurement Method | Timeline |
|---|---|---|---|---|
| {{KPI / SLA}} | {{baseline}} | {{target}} | {{how measured}} | {{when}} |
| {{ROI target}} | N/A | {{target}} | {{calculation method}} | {{when}} |

### Definition of Done

> The project is considered successful when:
- [ ] {{business outcome 1}}
- [ ] {{business outcome 2}}
- [ ] {{business outcome 3}}

---

## 3. Scope & Boundaries

### In Scope

| Feature / Capability | Description | Priority |
|---|---|---|
| {{capability}} | {{what it does in business terms}} | Must-Have / Should-Have / Nice-to-Have |

### Out of Scope

> The following items are **explicitly excluded** from this project. They may be considered in future phases.

| Item | Reason for Exclusion |
|---|---|
| {{excluded item}} | {{why — too complex, not needed for MVP, planned for Phase 2}} |

### Phasing Strategy

| Phase | Scope | Target Date | Business Value Delivered |
|---|---|---|---|
| Phase 1 (MVP) | {{scope}} | {{date}} | {{what value is delivered}} |
| Phase 2 | {{scope}} | {{date}} | {{what value is delivered}} |
| Phase 3 | {{scope}} | {{date}} | {{what value is delivered}} |

---

## 4. Solution Overview

> This section describes the solution in business terms. For technical details, refer to the Architecture document.

### How It Works

{{2–3 paragraphs explaining the solution from a user's perspective: what they interact with, how data flows through the process, what changes for them. Use business language.}}

### Conceptual Diagram

> How the solution fits into existing business processes:

```mermaid
graph LR
    A[📋 {{Business Process Start}}] --> B[⚙️ {{New System}}]
    B --> C[📊 {{Business Outcome}}]
    D[👥 {{Users}}] --> B
    E[🏢 {{Existing System}}] <--> B
```

### Integration with Existing Systems

| Existing System | How It Connects | Impact on Current Workflow |
|---|---|---|
| {{system}} | {{integration description in business terms}} | {{what changes for users}} |

---

## 5. Key Milestones & Timeline

### Roadmap

```mermaid
gantt
    title Project Roadmap
    dateFormat YYYY-MM-DD

    section Phase 1 — MVP
        {{Milestone 1}} :m1, {{start}}, {{duration}}
        {{Milestone 2}} :m2, after m1, {{duration}}

    section Phase 2 — Production
        {{Milestone 3}} :m3, after m2, {{duration}}

    section Phase 3 — Scale
        {{Milestone 4}} :m4, after m3, {{duration}}
```

### Milestone Details

| # | Milestone | Target Date | Deliverable | Business Value |
|---|---|---|---|---|
| 1 | {{name}} | {{date}} | {{what's delivered}} | {{why it matters}} |
| 2 | {{name}} | {{date}} | {{what's delivered}} | {{why it matters}} |
| 3 | {{name}} | {{date}} | {{what's delivered}} | {{why it matters}} |

---

## 6. Resource Requirements

### People

| Role | Headcount | Duration | Source |
|---|---|---|---|
| {{role}} | {{count}} | {{duration}} | Internal / External / Contractor |

### Budget

| Category | Estimated Cost | Frequency | Notes |
|---|---|---|---|
| Development | ${{amount}} | One-time | {{notes}} |
| Infrastructure | ${{amount}} | Monthly | {{notes}} |
| Licenses & Tools | ${{amount}} | Annual | {{notes}} |
| Training | ${{amount}} | One-time | {{notes}} |
| Contingency (15%) | ${{amount}} | — | Buffer for unknowns |
| **Total** | **${{total}}** | — | — |

### Business Stakeholder Commitments

> Resources needed from the business side:

| Need | From Whom | When | Duration |
|---|---|---|---|
| Subject Matter Expert access | {{department}} | {{phase}} | {{hours/week}} |
| Data access / permissions | {{data owner}} | {{phase}} | One-time setup |
| Approval gates | {{approver}} | {{milestones}} | Per milestone |

---

## 7. Risks & Mitigations

> Risks are framed in business impact terms.

| # | Risk | Likelihood | Business Impact | Mitigation Plan |
|---|---|---|---|---|
| R1 | {{risk description}} | Low / Medium / High | {{impact on revenue, reputation, compliance, or operations}} | {{mitigation strategy}} |
| R2 | {{risk}} | {{likelihood}} | {{impact}} | {{mitigation}} |
| R3 | {{risk}} | {{likelihood}} | {{impact}} | {{mitigation}} |

---

## 8. Dependencies & Assumptions

### Dependencies

| Dependency | Owner | Required By | Status | Impact if Delayed |
|---|---|---|---|---|
| {{external system, vendor, or decision}} | {{who owns it}} | {{date}} | ✅ Confirmed / ⚠️ Pending / ❌ Blocked | {{what happens if this is late}} |

### Assumptions

| Assumption | Impact if Wrong |
|---|---|
| {{assumption}} | {{what changes in the plan}} |

---

## 9. Change Management & Training

### Communication Plan

| Audience | Message | Channel | Timing |
|---|---|---|---|
| {{stakeholders}} | {{key message}} | {{email / meeting / town hall}} | {{when}} |
| {{end users}} | {{key message}} | {{channel}} | {{when}} |

### Training Plan

| Audience | Training Type | Duration | Delivered By | Timing |
|---|---|---|---|---|
| {{user group}} | {{workshop / documentation / video}} | {{hours}} | {{trainer}} | {{when}} |

### Post-Launch Support

| Support Level | Duration | Coverage | Escalation Path |
|---|---|---|---|
| Hypercare | {{X}} weeks post-launch | {{hours}} | {{path}} |
| Steady State | Ongoing | {{hours}} | {{path}} |

---

## 10. Governance & Decision Points

### Approval Authority

| Decision Type | Approver | Timeline |
|---|---|---|
| Project kickoff | {{name/role}} | Before development starts |
| Scope changes | {{name/role}} | As needed |
| Go-live | {{name/role}} | Before production release |
| Budget overrun (>10%) | {{name/role}} | As needed |

### Progress Reporting

| Report | Frequency | Audience | Format |
|---|---|---|---|
| Status update | {{weekly/biweekly}} | {{audience}} | Email / Dashboard |
| Milestone review | Per milestone | {{audience}} | Meeting + document |
| Executive summary | {{monthly}} | {{audience}} | 1-page brief |

### Business Sign-off Points

| Gate | What's Reviewed | Approver | Criteria for Approval |
|---|---|---|---|
| G1: Requirements sign-off | Scope & objectives | {{approver}} | {{criteria}} |
| G2: Solution design | Architecture & approach | {{approver}} | {{criteria}} |
| G3: Pre-launch | Testing results, readiness | {{approver}} | {{criteria}} |
| G4: Go-live | Production deployment | {{approver}} | {{criteria}} |

---

## 11. Next Steps

> Concrete actions needed from business stakeholders to move forward.

| # | Action | Owner | Deadline |
|---|---|---|---|
| 1 | {{action — e.g., Approve this proposal}} | {{name/role}} | {{date}} |
| 2 | {{action — e.g., Assign SME for requirements validation}} | {{name/role}} | {{date}} |
| 3 | {{action — e.g., Confirm budget allocation}} | {{name/role}} | {{date}} |
| 4 | {{action — e.g., Grant data access}} | {{name/role}} | {{date}} |

---

## Appendix

### Glossary

| Term | Definition |
|---|---|
| {{term}} | {{definition in plain language}} |

### References

- [Project Overview](./overview.md) — Full requirements
- [Architecture](./architecture.md) — Technical design details
- [Implementation Plan](./implementation-plan.md) — Detailed task breakdown
- [Tech Stack](./tech-stack.md) — Technology decisions and rationale
