# Handoff: {{PROJECT_NAME}}

> This document contains everything needed to start development in a separate workspace.

## Project Overview

{{1-paragraph summary — what is being built, for whom, and why}}

**Planning Documents**: See the full project folder at `projects/{{project-slug}}/`

---

## Repository Structure

```
{{project-slug}}/
├── src/
│   ├── {{component 1}}/
│   ├── {{component 2}}/
│   └── ...
├── tests/
├── docs/
├── .env.example
├── .gitignore
├── README.md
└── package.json / requirements.txt / go.mod
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| {{layer}} | {{tech}} | {{version}} |

See [tech-stack.md](./tech-stack.md) for full evaluation and rationale.

---

## Environment Setup

### Prerequisites

- {{prerequisite 1}}
- {{prerequisite 2}}

### Setup Commands

```bash
# Clone and install
git clone {{repo-url}}
cd {{project-slug}}
{{install command}}

# Configure environment
cp .env.example .env
# Edit .env with your values

# Run development server
{{dev command}}
```

### Environment Variables

| Variable | Description | Required |
|---|---|---|
| `{{VAR_NAME}}` | {{description}} | Yes/No |

### External Services

| Service | Purpose | Setup Link |
|---|---|---|
| {{service}} | {{purpose}} | {{link}} |

---

## Architecture Reference

See [architecture.md](./architecture.md) for full diagrams.

**Key Decisions**:
- {{decision 1 and rationale}}
- {{decision 2 and rationale}}

---

## First Sprint / MVP Scope

### Goals

{{What the MVP should achieve}}

### Features to Build

- [ ] {{Feature 1}} — {{brief description}}
- [ ] {{Feature 2}} — {{brief description}}

### Acceptance Criteria

- [ ] {{criteria 1}}
- [ ] {{criteria 2}}

See [implementation-plan.md](./implementation-plan.md) for the full milestone breakdown.

---

## CI/CD Recommendations

| Stage | Tool | Purpose |
|---|---|---|
| Build | {{tool}} | {{purpose}} |
| Test | {{tool}} | {{purpose}} |
| Deploy | {{tool}} | {{purpose}} |

---

## References

- [Project Overview](./overview.md)
- [Research](./research.md)
- [Tech Stack](./tech-stack.md)
- [Architecture](./architecture.md)
- [Implementation Plan](./implementation-plan.md)
