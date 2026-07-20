# CLAUDE.md — project context (auto-loaded by Claude Code)

> This file is the single source of truth for what this project is and where it stands.
> If you are an AI assistant resuming work: read this, then read `docs/BUILD_LOG.md` for the
> latest session, then continue. Do not assume anything not written here or in the docs.

## What we are building

A bring-your-own-key (BYOK) AI app builder — a Lovable/Bolt-style "describe it and it builds it"
platform where each user supplies their own AI API key. Built on a fork of **bolt.diy**.
The seller's value is the setup + managed hosting, not the AI. Michael (Matis) is customer #1,
but the infrastructure is built multi-tenant from the start so it can serve others.

## How to work with Michael (important)

- **Semi-technical.** Explain in **plain language**; avoid unexplained jargon.
- **One step at a time.** Give a single step, wait until he confirms it's done, then the next.
  Do NOT batch a pile of changes together.
- **Explain what + why before any change**, and let him choose before editing. Reads
  (opening/searching files) are free; changes pause for approval.
- When giving options, give a clear **recommendation**, not just a list.

## Guiding principles

- **Customer-shaped architecture, thin scope.** Build multi-user foundations now; do NOT build
  billing, team seats, or fancy features until a second real person asks to pay.
- **State lives in the repo, not in chat.** Every decision goes in `docs/DECISIONS.md`; every
  session ends with an entry in `docs/BUILD_LOG.md`.
- **The platform code and the customer's project code are separate trees.** Never mix them.
- **Never expose a stored API key to the browser.** Keys are handled server-side only.

## The stack (see docs/ARCHITECTURE.md for detail)

- Platform UI + agent: our fork of **bolt.diy** (MIT code) — this repo
- AI brain: **Anthropic (or other) API key** — brought by the user, billed to the user
  (current model: **Claude Sonnet 4.5**)
- Runtime / live preview: **StackBlitz WebContainers** — free for personal use, **commercial
  license required before reselling** (open item, see DECISIONS)
- Platform data (accounts, encrypted keys, project metadata): **Supabase**
- Customer code storage: **GitHub** (customer's account)
- Customer deployed sites: **Vercel / Netlify** (customer's account)
- **Platform hosting: Cloudflare Pages** — live at **`bolt-diy-8bq.pages.dev`**, auto-deploys
  from `main`. (We moved here from Railway; the Railway prebuilt image ran the app through
  Cloudflare's `wrangler pages dev` / `workerd` emulator, which crashed the streaming chat.
  On real Cloudflare Pages that bug is gone. See DECISIONS D-008 + BUILD_LOG.)

## Where the code lives

- Source of truth: this **GitHub repo** (`michaeleisner-source/bolt.diy`, our bolt.diy fork)
- Editing: local clone on Michael's PC, or Claude Code pointed at this repo
- Running (live platform): Cloudflare Pages, auto-deploying from `main` on push
- Update loop: edit → commit → push → Cloudflare auto-builds → live. Revert = check out an earlier commit.
- Convention: `main` = production. Make changes on a branch, test, then merge.
- Active working branch this session: `claude/bolt-chat-request-railway-8t0y4x`.

## Conventions

- Small commits, clear messages.
- Document a decision the moment it's made (DECISIONS.md), don't wait.
- Put a spending limit on the Anthropic API account so a runaway loop can't drain it.

## Current stage

**Stage 0 (foundations) and Stage 1 (hosting) are essentially DONE** — the fork is live on
Cloudflare Pages and chat builds apps. **Next is Stage 2 — the multi-tenant skeleton
(auth + per-user encrypted keys + per-user project isolation).** See `docs/ROADMAP.md` for the
exact next step.
