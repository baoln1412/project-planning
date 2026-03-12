# Implementation Plan: Dashboard Hub

This plan outlines the build process from project initialization to a functioning Phase 1 (MVP) and Phase 2 (AI Generation) application.

## Milestone 1: Foundation & Authentication
**Goal:** Scaffold the app, set up the database, and implement Google OAuth to secure the `drive.file` scope.
**Estimated Effort:** M/L (Complex Auth Flow)

- [ ] Initialize Next.js 15 (App Router) with Tailwind CSS and Tremor.
- [ ] Set up the Supabase project and run initial Drizzle migrations for `users`, `dashboards`, and `purchases` tables.
- [ ] Configure Auth.js with the Google Provider.
- [ ] **Critical:** Configure the Google OAuth scope to specifically request `https://www.googleapis.com/auth/drive.file`.
- [ ] Create a secure server utility to retrieve the Google Access Token from the current session for use in backend fetches.
- [ ] Build the basic homepage and navigation shell.

## Milestone 2: Storefront & Manual Purchases
**Goal:** Build the B2C storefront so users can browse available dashboards and see what they have purchased.
**Estimated Effort:** M

- [ ] Seed the Supabase `dashboards` table with 1-2 manual entries (e.g., a dummy "Habit Tracker").
- [ ] Build the `/store` page to list all active dashboards.
- [ ] Build the `/product/[slug]` page showing dashboard details and a "Buy" button (linking to a Stripe Checkout / Payment Link).
- [ ] Build the `/dashboard` index page showing the user's purchased templates (reading from `purchases` table).
- [ ] Create the Admin UI to manually toggle the `is_approved` flag on purchases.

## Milestone 3: The Dashboard Engine & Sheets Integration
**Goal:** Prove the core value proposition — reading from a user's cloned Google Sheet and rendering it beautifully with Tremor.
**Estimated Effort:** L (Complex Data Parsing)

- [ ] Create a Master Sheet template for a sample Habit Tracker with a specific column schema.
- [ ] Provide instructions in the web app on how users can clone the Master Sheet and paste their new Sheet URL into the app.
- [ ] Implement the Google Sheets API server-side fetcher using the user's OAuth token.
- [ ] Build the Next.js page `/dashboard/[slug]` that parses the raw Google Sheets JSON array into structured TypeScript objects.
- [ ] Build Tremor components (BarCharts, DataLists) to visualize the Sheet data.
- [ ] **Critical:** Implement robust try/catch error boundaries to handle cases where the user broke the Sheet schema.

## Milestone 4: Dedicated AI Dashboard Generation Skill & Design System
**Goal:** Establish the global brand guidelines and build a dedicated Antigravity "Skill" so the IDE can autonomously research components, consume dedicated idea folders, and generate new on-brand dashboards locally.
**Estimated Effort:** S

- [ ] Create a `_design/brand/` directory for global assets (logos, colors, theme variables, general UI preferences).
- [ ] Create a `_dashboards/` directory. Each new dashboard idea will live in its own subfolder here (e.g., `_dashboards/reading-tracker/`), storing its specific image mockups, component references, and prompt descriptions.
- [ ] Document the workflow in `README.md` (e.g., "To create a new product: 1. Create a folder in `_dashboards/[name]`. 2. Put UI references there. 3. Call the `generate-dashboard` skill.").
- [ ] **Create an Antigravity Skill:** Write `.agents/skills/generate-dashboard/SKILL.md`. This skill instruction file must explicitly instruct the Antigravity IDE to:
  1. Read the global brand guidelines from `_design/brand/`.
  2. Read the specific dashboard idea from `_dashboards/[target-folder]/`.
  3. Research and implement appropriate Tremor components.
  4. Write the Next.js page route securely.
  5. Output the required Google Columns and Apps Script formulas for a new Master Sheet.

---

## Next Steps
Once this plan is approved, we can run the `/export-project` workflow to compile these documents into a final `handoff.md` file designed specifically for a development workspace to execute.
