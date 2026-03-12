---
name: Tech Stack Advisor
description: Evaluate and recommend technologies for a project with comparison tables, scoring matrices, and rationale
---

# Tech Stack Advisor Skill

## Purpose

Evaluate technology options for a project and produce a `tech-stack.md` with recommendations, comparison tables, and clear rationale for each choice.

## When to Use

- During the tech stack phase of `/generate-plan`
- When the user asks for technology recommendations
- During `/add-feature` if new technologies are needed

## Evaluation Framework

### Categories to Evaluate

For each project, evaluate technologies across these categories:

| Category | Examples |
|---|---|
| **Language** | TypeScript, Python, Go, Rust, Java |
| **Frontend Framework** | React, Vue, Svelte, Next.js, Nuxt |
| **Backend Framework** | Express, FastAPI, Gin, Actix, Spring Boot |
| **Database** | PostgreSQL, MySQL, MongoDB, Redis, Supabase |
| **ORM / Data Layer** | Prisma, Drizzle, SQLAlchemy, GORM |
| **Authentication** | Supabase Auth, Clerk, Auth0, Firebase Auth |
| **Hosting** | Vercel, Railway, AWS, GCP, Fly.io, DigitalOcean |
| **CI/CD** | GitHub Actions, GitLab CI, CircleCI |
| **Monitoring** | Sentry, Datadog, Grafana, New Relic |
| **Storage** | S3, Cloudflare R2, Supabase Storage |

### Scoring Criteria

Score each option 1–5 on these dimensions:

| Criterion | Weight | Description |
|---|---|---|
| **Maturity** | 15% | How battle-tested is it? Stable releases? |
| **Community** | 15% | Size of community, ecosystem of plugins/extensions |
| **Scalability** | 20% | Can it handle growth? Horizontal scaling support? |
| **Developer Experience** | 15% | Documentation quality, tooling, learning curve |
| **Performance** | 15% | Speed, resource efficiency |
| **Cost** | 10% | Pricing at different scales (free tier → production) |
| **Ecosystem Fit** | 10% | How well does it integrate with other stack choices? |

### Comparison Table Format

For each category, produce a table like:

```markdown
### Frontend Framework

| Criterion | Next.js | Nuxt | SvelteKit |
|---|---|---|---|
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Community | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Scalability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| DX | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Cost | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Ecosystem Fit | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Weighted Total** | **4.35** | **4.10** | **3.95** |

**Recommendation**: Next.js — [rationale]
```

## Process

1. **Read `overview.md` and `research.md`** to understand requirements
2. **Identify which categories are relevant** (not all projects need all categories)
3. **Use web search** to check latest versions, pricing, and any recent changes
4. **Build comparison tables** for each category with 2–4 options
5. **Present to the user** and ask:
   - Any technologies you're already familiar with?
   - Any hard requirements or exclusions?
   - Budget constraints for hosted services?
6. **Finalize recommendations** based on user feedback

## Reasoning Approach

Use the `sequentialthinking` MCP tool when building comparison tables. For each technology category:
- **Thought 1–N**: Score each option across the 7 criteria, one at a time
- **Revision thoughts**: Revisit earlier scores as ecosystem fit between chosen technologies becomes clearer (e.g., choosing Supabase for DB may boost Supabase Auth's ecosystem fit score)
- **Final thought**: Calculate weighted totals and state your recommendation with rationale

This prevents scoring bias from evaluating all criteria simultaneously and allows course correction as the full stack picture emerges.

## Output Format

Write to `projects/<project-slug>/tech-stack.md`:

```markdown
# Tech Stack: <Project Name>

## Summary
<!-- 1-paragraph overview of the recommended stack -->

## Recommended Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Language | TypeScript | 5.x | [why] |
| Frontend | Next.js | 14.x | [why] |
| Backend | Next.js API Routes | — | [why] |
| Database | PostgreSQL (Supabase) | 15.x | [why] |
| Auth | Supabase Auth | — | [why] |
| Hosting | Vercel | — | [why] |
| CI/CD | GitHub Actions | — | [why] |

## Detailed Evaluation
### [Category 1]
<!-- comparison table + analysis -->

### [Category 2]
<!-- comparison table + analysis -->

## Cost Estimate

| Service | Free Tier | Production Est. | Scale Est. |
|---|---|---|---|
| [service] | [free tier details] | [$/mo] | [$/mo] |

## Alternatives Considered
<!-- Brief note on rejected options and why -->
```

## Important Notes

- Always check for the **latest stable versions** via web search
- Consider the user's existing skills and preferences
- Factor in **ecosystem compatibility** — the stack should work well together
- Include **cost estimates** at 3 scales: free/hobby, production, and scale
- If the user has a strong preference, respect it unless there's a technical blocker
