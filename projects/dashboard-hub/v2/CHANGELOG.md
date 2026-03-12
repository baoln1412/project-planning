# CHANGELOG — Dashboard Hub v2

**Version**: v2 (pre-update snapshot)  
**Date**: 2026-03-11  
**Status at snapshot**: 🟢 Ready for Development

## What was in this version

- Storefront & Product Pages with browse/purchase flow
- Manual Purchase Approval (admin toggles `is_approved` flag)
- Google Sheets Integration (direct API calls, no caching)
- Tremor + Tailwind 3.x dashboard UI
- AI Dashboard Generation as Milestone 4
- Master Data Sheet Cloning (manual instructions)
- `_design/` brand system for AI consistency
- 4 milestones: Foundation → Storefront → Dashboard Engine → AI Generation

## Reason for update

**Feature optimization review** — identified several anti-patterns and missing critical features:

1. **Manual Purchase Approval** was a conversion killer → replaced with Stripe Webhooks auto-approval
2. **Tremor** dependency risk → replaced with shadcn/ui + Recharts (zero vendor lock-in)
3. **No free tier / demo mode** → added free starter dashboard + sample data preview
4. **No data caching** → added Supabase `sheet_cache` layer to solve rate limits, fragility, and speed
5. **AI Generation premature** → deferred to Phase 2 (hand-craft first dashboards to validate market)
6. **No SEO strategy** → added SEO-optimized product pages
7. **Sheet cloning friction** → added one-click deep link cloning
8. **Tailwind 3.x** → updated to 4.x

---

## Guide: Using shadcn/ui + Recharts with Antigravity

### Setup

**shadcn/ui** is NOT an npm package — it's a CLI that copies component source code into your project. This means zero vendor lock-in.

```bash
# Initialize shadcn/ui in your Next.js project
npx shadcn@latest init

# Add individual components as needed
npx shadcn@latest add button card dialog table input tabs avatar badge
```

**Recharts** is a standard npm package:

```bash
npm install recharts
```

### Key Concept: shadcn/ui is Copy-Paste

Unlike Tremor (which is `npm install @tremor/react`), shadcn/ui copies actual component files into `src/components/ui/`. This means:
- **You own the code** — no breaking updates from upstream
- **Full customization** — edit the component source directly
- **AI-friendly** — Antigravity can read AND modify these components
- Components live at: `src/components/ui/button.tsx`, `src/components/ui/card.tsx`, etc.

### Prompting Antigravity for Dashboard Components

When asking Antigravity to generate dashboard UI, use prompts like:

**For a metric card:**
> "Create a dashboard metric card using shadcn/ui Card component. Show a title, a large number, and a percentage change badge. Use the existing components from `src/components/ui/`."

**For a chart:**
> "Create a revenue trend chart using Recharts BarChart. Data comes from the `sheet_cache` Supabase table. Use shadcn/ui Card as the container. Follow the brand guidelines in `_design/brand/`."

**For a full dashboard page:**
> "Create a new dashboard page at `src/app/dashboard/[slug]/page.tsx`. Use Server Components to fetch data from Supabase `sheet_cache`. Build the layout with shadcn/ui Card grid + Recharts charts. Reference `_design/brand/` for colors and styling."

### Common Component Patterns

**Dashboard Card (shadcn/ui):**
```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"

<Card>
  <CardHeader>
    <CardTitle>Total Habits</CardTitle>
  </CardHeader>
  <CardContent>
    <p className="text-3xl font-bold">24</p>
    <p className="text-sm text-muted-foreground">+12% from last week</p>
  </CardContent>
</Card>
```

**Bar Chart (Recharts):**
```tsx
import { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts"

<ResponsiveContainer width="100%" height={300}>
  <BarChart data={data}>
    <XAxis dataKey="date" />
    <YAxis />
    <Tooltip />
    <Bar dataKey="completed" fill="hsl(var(--primary))" radius={[4, 4, 0, 0]} />
  </BarChart>
</ResponsiveContainer>
```

**Line Chart (Recharts):**
```tsx
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts"

<ResponsiveContainer width="100%" height={300}>
  <LineChart data={data}>
    <XAxis dataKey="date" />
    <YAxis />
    <Tooltip />
    <Line type="monotone" dataKey="streak" stroke="hsl(var(--primary))" strokeWidth={2} />
  </LineChart>
</ResponsiveContainer>
```

### Connecting to Tailwind 4.x Theme

shadcn/ui uses CSS variables that map to your Tailwind theme. In Recharts, reference them with `hsl(var(--primary))`, `hsl(var(--muted))`, etc. This keeps charts consistent with your dashboard UI.

### Tips for the AI Generation Skill

When building the `/generate-dashboard` Antigravity Skill (Milestone 5), instruct it to:

1. **Always import from `@/components/ui/`** — never install new UI packages
2. **Use `ResponsiveContainer`** — wrap all Recharts charts for responsive sizing
3. **Use `hsl(var(--...))` colors** — inherit theme colors, never hardcode hex values
4. **Read `_design/brand/`** — check color palette, spacing, and typography rules before generating
5. **Use Server Components** — fetch data from Supabase in the component, not in client-side hooks

---

## Guide: Using Google Stitch MCP for Dashboard Design

### What is Google Stitch?

[Google Stitch](https://stitch.withgoogle.com) is an AI-powered UI design tool by Google Labs (powered by Gemini 2.5 Pro/Flash). It generates professional UI designs from text prompts or image mockups and exports them as **HTML/CSS or React components**. It has an MCP server (`stitch-mcp`) that lets Antigravity directly interact with Stitch projects.

### Why It's Valuable for Dashboard Hub

| Without Stitch | With Stitch |
|---|---|
| Describe UI in text → AI generates code blindly | **See the design first** → export React → adapt code |
| Brand consistency relies on text instructions | **Design DNA** auto-extracts fonts, colors, layouts from your first screen |
| Hard to preview before coding | **Prototyping** — multi-screen flows with interaction hotspots |
| One design option | **Multiple variants** — Stitch generates alternatives to pick from |

### Setup

```bash
# Install Stitch MCP CLI
npx @_davideast/stitch-mcp init

# This will:
# - Detect/install Google Cloud SDK
# - Handle OAuth authentication
# - Enable Stitch API on your Google Cloud project
# - Configure credentials
```

**Prerequisites:**
- Google Cloud project
- Google Cloud SDK installed
- Stitch API enabled (`stitch.googleapis.com`)

### Key Stitch MCP Tools

| Tool | What It Does |
|---|---|
| `create_screen` | Generate a UI screen from a text prompt |
| `extract_design_context` | Extract "Design DNA" (fonts, colors, layouts) from an existing screen |
| `fetch_screen_code` | Download the raw HTML/React code for a screen |
| `build_site` | Build an entire site by mapping screens to routes |

### Upgraded Dashboard Generation Workflow

**Old workflow** (text-only):
```
Creator writes text description → Antigravity generates code → Manual iteration
```

**New workflow** (Stitch + Antigravity):
```
Step 1: Design in Stitch (visual)
   └── Prompt: "Create a reading tracker bookshelf with dark theme, 
        book cover grid, sidebar navigation"
   └── Stitch generates 3 design variants
   └── Pick the best one

Step 2: Extract Design DNA
   └── Stitch MCP extracts: fonts, colors, spacing, layout patterns
   └── Save to _design/brand/ for future consistency

Step 3: Export React code
   └── fetch_screen_code → gets HTML/React output
   └── Save to _dashboards/reading-tracker/stitch-export/

Step 4: Antigravity adapts to shadcn/ui + Recharts
   └── Reads Stitch export + _design/brand/ guidelines
   └── Converts to Next.js App Router page component
   └── Replaces generic HTML with shadcn/ui Card, Button, etc.
   └── Adds Recharts charts where data visualization is needed
   └── Connects to Supabase sheet_cache for data fetching
```

### Practical Example: Designing the Reading Tracker

```bash
# Step 1: Create the bookshelf screen in Stitch
npx @_davideast/stitch-mcp create_screen \
  -p <project-id> \
  --prompt "A digital bookshelf dashboard for tracking reading. 
   Dark theme. Shows book covers in a grid on wooden shelf styling.
   Sidebar with nav: Library, Progress, Schedule, Challenges.
   Top stats: Books Read, Current Streak, Hours Read."

# Step 2: Extract Design DNA from the result
npx @_davideast/stitch-mcp extract_design_context \
  -p <project-id> -s <screen-id>

# Step 3: Get the React code
npx @_davideast/stitch-mcp fetch_screen_code \
  -p <project-id> -s <screen-id>

# Step 4: Serve locally to preview
npx @_davideast/stitch-mcp serve -p <project-id>
```

### Stitch + Antigravity Prompt Template

When adapting Stitch exports in Antigravity, use this prompt:

> "I have a Stitch UI export at `_dashboards/reading-tracker/stitch-export/`. Please:
> 1. Read the export and the Design DNA from `_design/brand/`
> 2. Convert the HTML to a Next.js 15 App Router Server Component at `src/app/dashboard/reading-tracker/page.tsx`
> 3. Replace generic HTML elements with shadcn/ui components (Card, Button, Badge, etc.)
> 4. Replace static charts with Recharts components connected to the `sheet_cache` data
> 5. Add TypeScript types for the Google Sheet data model
> 6. Ensure all colors use `hsl(var(--...))` theme variables"

### Also Consider: Google Sheets MCP

A Google Sheets MCP server can complement Stitch by letting Antigravity:
- **Create Master Sheet templates** directly (no manual sheet creation)
- **Populate sample data** for demo mode automatically
- **Read sheet schemas** to generate TypeScript types
- **Validate sheet structure** to catch broken schemas

This would close the loop: **Stitch designs the UI → Sheets MCP creates the data template → Antigravity wires them together.**
