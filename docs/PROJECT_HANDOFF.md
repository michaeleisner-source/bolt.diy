# PROJECT HANDOFF — BYOK Builder

> Durable summary of the whole venture (business + technical). Companion to `CLAUDE.md`
> (how we work) and `BUILD_LOG.md` (dated timeline). Paste-able into a fresh chat to resume.

## ⚠️ Status reconciliation (2026-07-20)
The original handoff below said the platform was live on **Railway**. That is **superseded**:
the app is now live on **Cloudflare Pages** at **`bolt-diy-8bq.pages.dev`** (Git-integration
auto-deploys from `main`). We moved off Railway because its prebuilt image runs the app
through Cloudflare's `wrangler pages dev` / `workerd` **emulator**, which crashed the
streaming chat with `Custom error: internal error; reference = ...`. On real Cloudflare
Pages that bug is gone and chat builds apps. **Railway is being retired** — see the
housekeeping list at the bottom. Treat any Railway URL/step below as historical.

---

## What we're building
A hosted, **bring-your-own-key (BYOK)** AI app builder — a Lovable/Bolt-style "describe it
and it builds it" platform, built on the open-source **bolt.diy**. Michael is **customer #1**;
the setup is built to be repeatable so it can be sold to others later.

## The business, in one line
The software is a commodity (bolt.diy is free). The product we sell is the **done-for-you
setup + managed hosting**, for non-technical people who find "get an API key and wire it up"
intimidating. Pitch: same experience as Lovable, no marked-up AI credits (you pay the
provider directly), and we set it all up.

## Rules / decisions locked in
- **BYOK is the model.** Users bring their own AI API key; no per-token markup; flat setup fee.
- **API key ≠ Max subscription.** bolt.diy calls the **Anthropic API** (pay-per-token,
  prepaid credits at console.anthropic.com), separate from a Claude Max plan. This is NOT
  Claude Code — it's the Claude model via the API.
- **Build on Sonnet, not Opus, for everyday work.** Opus burns credit fast; Sonnet is the
  sweet spot. Switch to Opus only when Sonnet gets stuck. (Current pick: **Sonnet 4.5**.)
- **"Cheaper than Lovable" is conditional** — true for light/moderate use on Sonnet; can
  flip if heavy on Opus. For customers it's always true (they pay their own tokens directly).
- **Platform vs. project are separate layers.** bolt.diy is the workshop (add AI providers
  here). Tools like Firecrawl, Supabase, Resend belong to the individual apps you BUILD,
  using that app's own keys — not added to bolt.diy itself.
- **Reselling gate:** bolt.diy code is MIT (rebrandable), but it runs on **WebContainers**,
  which needs a **commercial license from StackBlitz** for for-profit resale (personal
  use/prototypes are free). Resolve before charging customers. *Not legal advice.*
- **Keep state in the repo/knowledge base, not in chat.** End each session with a BUILD_LOG
  entry.
- **Security:** treat API keys like passwords; set a spending limit; when multi-tenant, store
  customer keys **encrypted server-side, never exposed to the browser**.

## What's DONE
1. GitHub account (`michaeleisner-source`) ✅
2. Forked bolt.diy → `github.com/michaeleisner-source/bolt.diy` ✅
3. **Live on Cloudflare Pages** at `bolt-diy-8bq.pages.dev`, chat builds apps ✅
   *(Replaces the earlier Railway deploy, which had the streaming bug.)*
4. Anthropic API key created, funded, set as a **Cloudflare Pages Production secret** ✅
5. Auto-deploy: every push to `main` rebuilds & redeploys ✅

## What's NEXT
See **`docs/ROADMAP.md`** for the phased game plan (Lovable-style dashboard → accounts →
multi-tenant/sellable). High-level near-term:
- [ ] Set **Sonnet 4.5** as the app's default model (code fix, so first-load never errors)
- [ ] Retire old **Railway** services (stop paying for the broken ones)
- [ ] Confirm Anthropic **spending limit** set / auto-reload OFF
- [ ] Pick a **brand name** (replace "bolt.diy"/"BoltKey" placeholder)
- [ ] Optional: **custom domain** on Cloudflare Pages
- [ ] Then: **Phase 1** — the Lovable-style Projects dashboard (see ROADMAP)

## Housekeeping / cleanup
- **Railway:** retire the old services (the GitHub-connected `bolt.diy` service and the
  Docker-image service). Delete any stray/empty Railway projects (e.g. "pure-recreation").
  Goal: stop paying for anything now replaced by Cloudflare Pages (free tier).
- **GitHub repos to keep:** `gothamvendingadmin` (Gotham Vending), `nimble-spark-builder`
  (= "Property Ledger"). (`westfieldx-api` already deleted.)
