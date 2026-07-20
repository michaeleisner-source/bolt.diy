# Roadmap

Pace: one step at a time. Do a step, confirm it works, log it in BUILD_LOG.md, then move on.
`[ ]` = not started, `[~]` = in progress, `[x]` = done.

> **Status note (2026-07-20):** we are further along than the original plan and the host changed.
> Stage 0 and Stage 1 are essentially done, and the platform host is **Cloudflare Pages** (not
> Railway — see DECISIONS D-008). The Lovable-style **projects dashboard** the user wants lives
> naturally inside **Stage 2** (per-user projects), so it is folded in there.

---

## Stage 0 — Foundations (repo + docs + running)  ✅ essentially done

- [x] 0.1  Set up this knowledge base (these files)
- [x] 0.2  Fork `stackblitz-labs/bolt.diy` → `michaeleisner-source/bolt.diy`
- [x] 0.3  Add `/docs` + `CLAUDE.md` + `README.md` to the fork
- [x] 0.5  App builds and runs (built successfully; live on Cloudflare — see Stage 1)
- [x] 0.6  Anthropic API key added, first test build done (you = customer #1)
- [ ]  0.7  Put a **spending limit** on the Anthropic API account  ← confirm this is set
- [ ]  0.8  Set **Sonnet 4.5** as the app default model (`DEFAULT_MODEL` in
      `app/utils/constants.ts` + refresh stale `staticModels` in the Anthropic provider) so a
      fresh load never hits the retired-model error

## Stage 1 — Host it (log in from anywhere)  ✅ done (on Cloudflare Pages)

- [x] 1.1  Host chosen: **Cloudflare Pages** (moved off Railway; fixed the streaming-chat bug)
- [x] 1.2  GitHub fork connected; **auto-deploy from `main`** enabled
- [x] 1.3  Live + reachable at `bolt-diy-8bq.pages.dev` (works from any device/browser)
- [x] 1.4  `main` = production; changes on a branch then merge
- [ ]  1.5  Housekeeping: **retire old Railway services** (stop paying); optional custom domain

## Stage 2 — Multi-tenant skeleton (the "assume customers" layer)  ← WE ARE HERE

This is also where the **Lovable-style experience** gets built: log in → a **projects dashboard**
(grid of your projects) → open one or start new. bolt.diy already saves projects and has a history
sidebar, but only in the browser and with no login; Stage 2 makes it real.
- [ ]  2.1  Add **auth** (Supabase Auth or Clerk) — login/signup
- [ ]  2.2  Per-user **encrypted** API key storage (server-side only, never to the browser)
- [ ]  2.3  Per-user **project isolation** + a **projects dashboard** home screen
- [ ]  2.4  Connect a customer's own Vercel/Netlify for their deploys

## Stage 3 — Productize & sell

- [ ]  3.1  Onboarding flow: guided "how to get your API key in 3 clicks"
- [ ]  3.2  Polish `CUSTOMER_EXPLAINER.md` into real onboarding/sales material
- [ ]  3.3  Resolve the runtime: **WebContainer commercial license** or own sandbox
- [ ]  3.4  Validate with 3–5 real people BEFORE building billing
- [ ]  3.5  Billing (Stripe), only after validation
- [ ]  3.6  Domain + brand name (replace "bolt.diy"/"BoltKey" placeholder)

---

## Parallel / ongoing

- [ ] Register a domain and lock in a brand name early
- [ ] Keep BUILD_LOG.md updated every session (the save point)
- [ ] Keep a demo of your own real project (built in the platform) as the sales asset
