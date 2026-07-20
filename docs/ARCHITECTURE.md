# Architecture

## The core idea

Two completely separate things share the screen and must never be confused:

1. **The platform** — the builder app customers log into (this repo, our bolt.diy fork).
2. **The customer's project** — the site/app a customer builds inside the platform. It runs in
   the customer's browser (WebContainers) and deploys to the customer's own hosting.

Keeping these separate is the entire cost model: everything variable sits on the customer's
accounts, so our price can be a flat fee with almost no cost of goods.

## The six layers, and who pays for each

| # | Layer | What it is | Whose account / who pays |
| --- | --- | --- | --- |
| 1 | Builder UI + AI agent | The platform itself | **Us** (this repo) — the thing we brand and sell |
| 2 | AI brain | Claude / GPT / etc. via API | **Customer's API key**, billed per token to them |
| 3 | Runtime / live preview | StackBlitz WebContainers (runs in browser) | Free for personal use; **we license it for resale** (or swap for our own sandbox) |
| 4 | Code storage | Git repo of the customer's project | **Customer's GitHub** |
| 5 | Deployed site hosting | Where the finished live site runs | **Customer's Vercel / Netlify** |
| 6 | Project database | If the built app needs one | **Customer's Supabase** |

Layers 2, 4, 5, 6 are bring-your-own. The only cost that is truly ours is Layer 3 at resale —
which is why the WebContainer license (or building our own sandbox) is the one real infrastructure
cost of the business.

## Where OUR platform code lives (4 places, 4 jobs)

- **Source of truth →** GitHub repo (our bolt.diy fork). This is where the code *is*.
- **Editing →** local clone on the PC, or Claude Code pointed at the repo.
- **Running →** **Cloudflare Pages**, auto-deploying from `main` on push (live at
  `bolt-diy-8bq.pages.dev`). *(Originally planned for Railway/Render/Coolify; moved to Cloudflare
  Pages because Railway's image ran the app in the `workerd` emulator, which crashed streaming
  chat — see DECISIONS D-008.)*
- **Platform data →** Supabase (accounts, encrypted customer keys, project metadata). This is
  state, not code — deliberately separate from the repo. *(Planned for Stage 2; not wired yet.)*

## The update loop

```
edit (locally or via Claude Code)  ->  commit  ->  push to GitHub
      ->  Cloudflare auto-builds & deploys  ->  live in ~1-2 min
```

- History + revert is just git: every update is a commit; rolling back is checking out an earlier one.
- `main` = production. Changes go on a branch, get tested, then merge. Cloudflare gives a preview URL
  per branch so you never edit the live thing while a customer is mid-build.

## Multi-tenant additions (the "assume customers" work)

A fresh bolt.diy fork is single-user. To serve others we add:
- **Auth** (Supabase Auth or Clerk) — login and per-user isolation
- **Per-user encrypted API key storage** — server-side only, never sent to the browser
- **Per-user deploy target** — connect the customer's Vercel
- **Billing** (Stripe) — added last, only after someone says yes

This wrapper is the custom engineering we own. It is also literally "the scaffolding" that is the
product.

---

## Current implementation notes (2026-07-20)

Concrete facts about how the platform actually runs today, so the conceptual model above stays
honest:

- **Host:** Cloudflare Pages (Git integration, auto-deploy from `main`). Build config: framework
  preset `None`, build command `pnpm run build`, output dir `build/client`, build env
  `NODE_VERSION=22`. `wrangler.toml` provides `compatibility_flags = ["nodejs_compat"]` + a
  compatibility date, which Pages honors because `pages_build_output_dir` is set.
- **Chat flow:** browser → `app/routes/api.chat.ts` (`action`) → `streamText` (Vercel AI SDK) →
  Anthropic API → SSE tokens stream back → files/preview render. Its `onError` wraps failures as
  `Custom error: <message>` (so that prefix means "read the rest for the real cause").
- **Runtime lesson (why the host mattered):** `pnpm run dev` runs the app in **plain Node.js**
  (bindings emulated) and streaming works; the old Railway image's `wrangler pages dev` ran it in
  the **`workerd` emulator** and streaming crashed; **Cloudflare Pages** runs `workerd` natively and
  it works.
- **Keys today (single-user):** `ANTHROPIC_API_KEY` is a Cloudflare Pages **Production secret**
  (also lets the live model list load). Default model: **Claude Sonnet 4.5**.
- **Keys at Stage 2 (multi-tenant):** each user's key stored **encrypted, server-side only, never
  exposed to the browser** — the core security rule.
