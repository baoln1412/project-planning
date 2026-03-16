# Tech Stack: {{PROJECT_NAME}}

## Summary

{{1-paragraph summary: technology philosophy, key choices, and how they fit together}}

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| {{layer}} | {{tech}} | {{version}} | {{why}} |

## Compatibility Matrix

> How chosen technologies interact with each other.

| | {{Tech A}} | {{Tech B}} | {{Tech C}} | {{Tech D}} |
|---|---|---|---|---|
| **{{Tech A}}** | — | ✅ Native | ✅ SDK | ⚠️ Adapter |
| **{{Tech B}}** | ✅ Native | — | ✅ Native | ✅ Native |
| **{{Tech C}}** | ✅ SDK | ✅ Native | — | ❌ Manual |
| **{{Tech D}}** | ⚠️ Adapter | ✅ Native | ❌ Manual | — |

Legend: ✅ Native integration | ⚠️ Requires adapter/wrapper | ❌ Manual integration needed

---

## Detailed Evaluations

### {{Category 1, e.g., Backend Framework}}

#### ADR-{{NNN}}: {{Decision Title}}

- **Status**: ✅ Accepted
- **Context**: {{What is the technical context? What problem are we solving?}}
- **Decision**: {{What did we decide?}}
- **Consequences**:
  - ✅ {{positive consequence}}
  - ⚠️ {{tradeoff or risk}}

**Evaluation Matrix**:

| Criterion | {{Option A}} | {{Option B}} | {{Option C}} |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Community & Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Developer Experience | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Operational Complexity | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Team Capability Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Weighted Total** | **X.XX** | **X.XX** | **X.XX** |

**Recommendation**: **{{Winner}}** — {{rationale}}

*Why not {{Runner-up}}?* {{reason}}

---

### {{Category 2, e.g., Database}}

#### ADR-{{NNN}}: {{Decision Title}}

{{Repeat the ADR + evaluation matrix format for each technology decision}}

---

## Operational Complexity Assessment

| Technology | Deploy Difficulty | Monitoring | Scaling | Backup/Recovery | Overall |
|---|---|---|---|---|---|
| {{tech}} | 🟢 Easy | 🟢 Built-in | 🟡 Manual | 🟢 Automated | 🟢 Low |
| {{tech}} | 🟡 Moderate | 🟡 Config needed | 🟢 Auto | 🟡 Semi-auto | 🟡 Medium |
| {{tech}} | 🔴 Complex | 🔴 Custom setup | 🔴 Custom | 🔴 Manual | 🔴 High |

## Team Capability Fit

| Technology | Team Experience | Learning Curve | Training Needed | Risk |
|---|---|---|---|---|
| {{tech}} | ✅ Strong | Low | None | 🟢 |
| {{tech}} | ⚠️ Some exposure | Medium | 1-2 days | 🟡 |
| {{tech}} | ❌ None | High | 1-2 weeks | 🔴 |

## Cost Estimate

| Service | Free / Dev | Production | Scale (10x) |
|---|---|---|---|
| {{service}} | {{cost}} | {{cost}} | {{cost}} |
| **Total** | **$X/mo** | **$X/mo** | **$X/mo** |

## Alternatives Considered

| Technology | Category | Why Rejected |
|---|---|---|
| {{tech}} | {{category}} | {{reason}} |
