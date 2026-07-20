# CLAUDE.md — Project Context for AI Assistants

> Read this first. It captures **what this project is**, **who it's for**, and **how we work together** so context is never lost between sessions.

## What this project is

This is a fork of **bolt.diy** (the open-source, community fork of StackBlitz's bolt.new)
being turned into a **hosted, bring-your-own-key (BYOK) AI app builder** — a
Lovable-style "describe it and it builds it" platform.

- **Owner:** Michael (michaeleisner@gmail.com). Michael is **customer #1**.
- **Long-term goal:** grow this from "just me" into a product that Michael can set up
  and run for other users.
- **Today's reality:** single-user (Michael), used to validate the idea and keep building.

## Who Michael is (how to work with him)

- **Semi-technical.** Explain in **plain language**, avoid unexplained jargon.
- **Go step by step, ONE step at a time.** Do **not** batch a pile of changes together.
- **Explain what and why before making any change**, and let Michael choose before
  editing anything. Reads (opening/searching files) are fine to do freely; changes pause
  for approval.
- When giving options, give a clear recommendation — don't just list choices.

## The stack (as deployed today)

- **App:** bolt.diy — a Remix app **built to target Cloudflare Pages** (uses
  `@remix-run/cloudflare`, `functions/[[path]].ts`, `wrangler.toml`).
- **AI:** Vercel AI SDK (`ai`, `@ai-sdk/*`). Chat streams Server-Sent Events (SSE) from
  the LLM back to the browser. Current model: **Anthropic Sonnet**. Keys are BYOK
  (entered in the UI and/or set as server env vars).
- **Hosting (current):** **Cloudflare Pages** via **Git integration** — Cloudflare builds
  and deploys automatically on every push to `main`. Live at **`bolt-diy-8bq.pages.dev`**.
  - Build config: framework preset `None`, build command `pnpm run build`, output dir
    `build/client`, build env var `NODE_VERSION=22`.
  - `ANTHROPIC_API_KEY` is set as a **Production secret** in the Pages project (Settings →
    Variables and Secrets). This is what makes the live model list load + chat work.
- **Hosting (old, being retired):** Railway, running `ghcr.io/stackblitz-labs/bolt.diy:latest`
  via `pnpm run dockerstart` → `wrangler pages dev` (the `workerd` emulator). That emulator
  is what broke streaming chat; moving to real Cloudflare Pages fixed it.

## Key architecture facts worth remembering

- **Two different runtimes depending on how it's started:**
  - `pnpm run dev` (`remix vite:dev`) → app code runs in **plain Node.js** (Cloudflare
    bindings are *emulated* via `remixCloudflareDevProxy`). Streaming chat works reliably.
  - `pnpm run dockerstart` (`wrangler pages dev`) → app code runs inside **`workerd`**,
    Cloudflare's Workers runtime emulator. This is what the Railway production image uses.
- The chat endpoint is `app/routes/api.chat.ts`. Its `onError` handler wraps any failure
  as `"Custom error: <message>"`. So a browser error of
  `Custom error: internal error; reference = ...` means the underlying message was
  `internal error; reference = ...` — which is a **`workerd` runtime internal error**.
- `bindings.sh` turns server env vars (listed in `worker-configuration.d.ts`) into
  `--binding NAME=VALUE` flags for wrangler, so `ANTHROPIC_API_KEY` from Railway does
  reach the app. (Env plumbing is fine; the failure is the runtime, not a missing key.)

## Status (as of 2026-07-20): ✅ RESOLVED

The original bug — chat failing with `Custom error: internal error; reference = ...` — was
the Railway/`workerd` emulator choking on the streaming AI request. **Fixed by moving to
Cloudflare Pages** (where `workerd` runs natively). bolt.diy is now live at
`bolt-diy-8bq.pages.dev` and chat builds apps successfully.

Two smaller issues surfaced and were fixed during the first working chat: a **retired
default Anthropic model** and the **API key needing to be server-side** (added as a Pages
secret) so the live model list loads. See `docs/BUILD_LOG.md` for the full story.

**Known nice-to-have (not blocking):** the app still *defaults* to a retired model
(`DEFAULT_MODEL` in `app/utils/constants.ts`) and ships stale `staticModels` in
`app/lib/modules/llm/providers/anthropic.ts`. Worth updating to current model IDs so
first-load never errors — pending Michael's go-ahead.

## Working agreements / conventions

- **Branch:** develop on `claude/bolt-chat-request-railway-8t0y4x`. Commit with clear
  messages; push when a unit of work is complete. Never push to another branch without
  explicit permission. Do not open a PR unless Michael asks.
- **Keep these docs current.** After any meaningful decision or change, update
  `docs/BUILD_LOG.md` (and this file if the project's shape changes).
