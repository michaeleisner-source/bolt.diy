# ROADMAP — Making bolt.diy feel like Lovable

> Goal: turn this bolt.diy fork into a Lovable-style product — **log in, see all your
> projects, pick one to continue or start a new one** — used by Michael first, built so it
> can serve other customers later. Companion to `PROJECT_HANDOFF.md` (the venture) and
> `BUILD_LOG.md` (dated timeline). Work **one phase at a time**; update BUILD_LOG as we go.

## Where we are today (baseline)
- ✅ Live on Cloudflare Pages, chat builds apps, auto-deploys on push to `main`.
- bolt.diy **already** saves every project and has a slide-out **history sidebar**
  (`app/components/sidebar/Menu.client.tsx`) — but projects are stored **in the browser only**
  (IndexedDB, `app/lib/persistence/`), and there is **no login / no accounts**.

## The gap vs. Lovable
| Lovable has | bolt.diy today |
|---|---|
| Log in from any device | No accounts at all |
| Cloud-stored projects per user | Projects saved in this browser only |
| A projects **dashboard** (grid of cards) | A slide-out history sidebar |
| Open a project → builder | ✅ already works |

---

## Phase 0 — Foundation & housekeeping (quick wins)
Small, low-risk items that stabilize the base before we build.
- [ ] **Default model → Sonnet 4.5** (edit `DEFAULT_MODEL` in `app/utils/constants.ts` +
      refresh stale `staticModels` in `app/lib/modules/llm/providers/anthropic.ts`) so a
      fresh load never hits the retired-model error.
- [ ] **Retire Railway** (stop paying for the old broken services).
- [ ] **Anthropic spend limit** set / auto-reload OFF.
- [ ] **Brand name** chosen (replace "bolt.diy"/"BoltKey" placeholders in the UI).
- [ ] Optional: **custom domain** on Cloudflare Pages.

## Phase 1 — Make it *feel* like Lovable (single-user, no login yet)
Goal: opening the site lands on a **Projects dashboard** (grid of your projects + "New
Project"), styled like Lovable — instead of a blank chat.
- [ ] Build a **dashboard home screen** that lists projects from the existing local history
      (reuse `app/lib/persistence` + the sidebar's data).
- [ ] Project **cards**: name, last-edited, open button; a **New Project** button.
- [ ] Light **rebranding / visual polish** toward the Lovable look.
- **Why first:** delivers the Lovable *experience* fast, front-end only, **no risky backend**.
- **Limitation (accepted for now):** projects still live in the browser — fine for one
  person on one machine.

## Phase 2 — Real accounts + cloud projects (the product backbone)
Goal: log in anywhere and your projects follow you.
- [ ] **Authentication** (login/signup). Recommended: **Supabase Auth** (Supabase is already
      partially wired into bolt.diy, so it pairs naturally with the database).
- [ ] **Cloud database** (Supabase Postgres) storing projects **per user** (files + chat
      history), replacing/backing the browser-only storage.
- [ ] Dashboard reads/writes the **cloud**, keyed to the logged-in user.
- **Biggest lift** — break into small sub-steps; each shippable on its own.

## Phase 3 — Sellable to others (multi-tenant)
Only after real people say they'd pay.
- [ ] **Per-user API keys stored encrypted server-side**, never exposed to the browser.
- [ ] **Onboarding flow** — guide a new customer to add their key + set a spend limit.
- [ ] **Billing** (e.g. Stripe) for the setup fee / subscription.
- [ ] ⚠️ **WebContainer commercial license** from StackBlitz — **legally required before
      charging customers** (our own locked-in rule). Resolve, or swap the runtime.
- [ ] Full branding / polish.

---

## Recommended order
1. **Phase 0 quick wins** (start with the Sonnet 4.5 default — small, immediate).
2. **Phase 1 dashboard** — the first real "feels like Lovable" build.
3. Plan **Phase 2** carefully once Phase 1 is in your hands.

## Decision log for this roadmap
- *(add dated decisions here as we make them — e.g. chosen auth provider, DB schema, brand
  name — so the plan and the reasons stay in the repo, not in chat.)*
