# BUILD LOG

A running journal of what we're building, what we decided, and why. Newest entry on top.
Companion to `CLAUDE.md` (which holds the stable "what/who/how"). This file holds the
**timeline**: problems, investigations, decisions, and next steps.

---

## 2026-07-20 — Chat request fails on Railway ("Custom error: internal error; reference = ...")

### Symptom
- Deployed on Railway via prebuilt image `ghcr.io/stackblitz-labs/bolt.diy:latest`
  (`RUNNING_IN_DOCKER=true`, `PORT=5173`). App loads fine at its Railway URL.
- Sending a chat prompt fails. Browser console:
  `ERROR Chat chat request failed Error: Custom error: internal error; reference = ...`
- Server logs healthy: Wrangler reports `Ready on http://0.0.0.0:5173`, only harmless
  "sideEffects" warnings. Anthropic key set & funded; model = Sonnet.

### Investigation (what the code shows)
- **How the prod image runs the app:** `Dockerfile` → `CMD ["pnpm", "run", "dockerstart"]`
  → `wrangler pages dev ./build/client --ip 0.0.0.0 --port 5173`. That is Cloudflare's
  **local dev emulator (`workerd`)**, not a real production Node server, and not
  Cloudflare's real edge network.
- **Where the error string comes from:** `app/routes/api.chat.ts` `onError` returns
  ``Custom error: ${errorMessage}``. So the real underlying message is
  `internal error; reference = ...` — the signature of a **`workerd` internal error**.
- **Why it works in dev but not in the prod image:** `vite.config.ts` uses
  `remixCloudflareDevProxy()`, so `pnpm run dev` runs the app in **plain Node.js** (CF
  bindings emulated). The prod image instead runs it in **`workerd`**. The long-lived
  **streaming** LLM response is what `workerd` chokes on when it's running inside a plain
  container instead of on Cloudflare's network.
- **Clue that a dev-mode attempt was started before:** `vite.config.ts` line ~24 already
  has `allowedHosts: ['boltdiy-production-0d8a.up.railway.app']`. That setting is **only**
  read by the Vite dev server — never by `wrangler pages dev`. So someone previously began
  wiring up "run dev mode on Railway" but didn't finish it.
- **Env plumbing is fine:** `bindings.sh` converts Railway env vars (from
  `worker-configuration.d.ts`) into wrangler `--binding` flags, so `ANTHROPIC_API_KEY`
  does reach the app. The failure is the **runtime**, not a missing key.

### Diagnosis (plain language)
Michael's theory is essentially correct. bolt.diy's production image runs the app through
Cloudflare's Wrangler emulator (`workerd`). `workerd` is happy on Cloudflare's real
network but unstable for long streaming AI responses when crammed into a plain Railway
container — it throws an opaque `internal error; reference = ...`, which surfaces in the
browser as the `Custom error: ...` above. The app loads (static/simple requests are fine);
only the **streaming chat** path trips the emulator.

### Options weighed (a / b / c)
- **(a) Adjust how it runs on Railway (tweak the emulator):** Give Railway more
  memory / tweak wrangler flags while still using `wrangler pages dev`. *Trade-off:* still
  the wrong runtime; may reduce frequency but won't reliably remove the root cause.
  A robust version would swap to a real Node server adapter — a nontrivial change bolt.diy
  doesn't officially support.
- **(b) Deploy to Cloudflare Pages instead:** Run bolt.diy where it's designed to run.
  `workerd` is native there, so this class of streaming error largely disappears.
  *Trade-off:* move off Railway; set up a Cloudflare deploy + move secrets; live within
  Cloudflare's request/subrequest limits. This is the platform-with-the-grain, product-grade
  answer.
- **(c) Run the dev-mode server (with more memory) on Railway:** Change the container to
  run `pnpm run dev --host` (plain **Node**, no `workerd`) instead of `wrangler pages dev`.
  *Trade-off:* it's a *development* server (unminified, heavier, not hardened for
  multi-user public traffic) — great as an unblock for "just me" now, not the thing you'd
  ship to other customers later. Smallest, most reversible change (essentially the start
  command). The half-finished `allowedHosts` entry suggests this was the earlier intended
  path.

### Recommendation
- **Fastest working fix today:** **(c)** — flip Railway to the Node dev server so the
  `workerd` internal error disappears and Michael is unblocked to keep building.
- **Best long-term / product-grade:** **(b)** — Cloudflare Pages, when ready to harden for
  other users.
- **(a)** is the weakest as pure tweaks: doesn't remove the root cause.

### Decision
- ✅ **Michael chose Option (b): deploy to Cloudflare Pages.** Fastest fix (c) was
  available but he opted straight for the durable, product-grade home.

### Deployment plan (Cloudflare Pages, via Git integration)
Repo is already Pages-ready (`wrangler.toml` with `pages_build_output_dir`,
`functions/[[path]].ts`, `nodejs_compat` flag, `pnpm run deploy`). Expected steps:
1. Cloudflare account (free tier is fine) + connect the GitHub repo
   (`michaeleisner-source/bolt.diy`) as a Pages project via Git.
2. Build config — build command `pnpm run build`, output dir `build/client`,
   production branch (likely `main`). Pin Node version if the builder default causes issues.
3. Set the `nodejs_compat` compatibility flag + secrets (`ANTHROPIC_API_KEY`) in the
   Pages project.
4. Deploy, then test a chat prompt on the `*.pages.dev` URL.
5. Once verified: optionally attach a custom domain and decommission the Railway service.

### 2026-07-20 (later) — Direct-upload from the assistant's build env is blocked
- Tried **Plan B** (assistant builds + `wrangler pages deploy` from this environment).
  - ✅ Installed deps and built the app successfully here. (Had to temporarily strip
    Electron desktop-build tooling because this env's egress policy blocks
    `codeload.github.com`, which hosts `@electron/node-gyp`. `package.json` +
    `pnpm-lock.yaml` were restored afterward — repo left pristine. App code imports no
    Electron packages, so the web build is unaffected.)
  - ❌ **Deploy blocked:** this environment's egress policy also blocks `api.cloudflare.com`
    (proxy returns 403 CONNECT). Per policy we do not route around it. So the assistant
    **cannot** push the deploy from here. The upload must run somewhere allowed to reach
    Cloudflare.
- **Two viable paths that avoid this env entirely:**
  1. **Cloudflare Pages Git integration** (recommended end-state): Cloudflare pulls the
     repo and builds/deploys on *their* servers — unaffected by this firewall — and
     auto-deploys on every push. Blocker to clear: the earlier "unable to install on
     GitHub account" error (fix = uninstall/reinstall the Cloudflare GitHub App).
  2. **Local `wrangler` deploy from Michael's own computer**: `wrangler login` (browser)
     then `pnpm run deploy`. No GitHub App needed; more terminal steps.

### 2026-07-20 (resolved) — ✅ Live on Cloudflare Pages, chat works
- **Chosen path:** Cloudflare Pages via **Git integration** (Michael's pick).
- **Deploy setup that worked:**
  - Cleared the earlier "unable to install on GitHub account" error by removing/reinstalling
    the Cloudflare Pages GitHub App on the personal account.
  - Used the **Pages** create flow (not the Workers flow) →
    `dash.cloudflare.com/<acct>/workers-and-pages/create/pages` → Import Git repo →
    `michaeleisner-source/bolt.diy`.
  - **Build settings:** Framework preset `None`; build command `pnpm run build`; build
    output directory `build/client`; build env var `NODE_VERSION=22`.
  - Result: deployed to **`bolt-diy-8bq.pages.dev`**. Auto-redeploys on push to `main`.
- **Original bug confirmed fixed:** the `Custom error: internal error; reference = ...`
  (workerd) no longer appears — that was purely the Railway/Wrangler emulator. On real
  Cloudflare Pages the streaming chat runs fine.
- **Two follow-on issues hit and fixed during first chat:**
  1. `Custom error: model: claude-3-5-sonnet-20241022` — Anthropic **retired** that model
     snapshot. The app's 3 built-in (static) models are all stale, and it defaults to a
     retired one (`DEFAULT_MODEL = 'claude-3-5-sonnet-latest'` in `app/utils/constants.ts`).
  2. The **live** model list wasn't loading (only 3–4 models shown), because the key was
     only in the browser cookie, not on the server. **Fix:** added `ANTHROPIC_API_KEY` as a
     **Production secret** in Cloudflare Pages (Settings → Variables and Secrets), then
     **Retry deployment**. After that the real model list loaded, a current Sonnet was
     selected, and prompts build successfully.

### Environment / infrastructure notes learned
- The assistant's build environment is **firewalled from Cloudflare** (`api.cloudflare.com`
  *and* `*.pages.dev` return 403 CONNECT) and from `codeload.github.com`. So the assistant
  cannot deploy, test the live site, or install GitHub-tarball deps from here. Deploys must
  run on Cloudflare's own build servers (Git integration) — which is what we set up.

### Open / next (nice-to-have, not blocking)
- **Durable model fix (proposed):** update `DEFAULT_MODEL` + the Anthropic `staticModels`
  list to current, non-retired model IDs so first-load never errors and future users don't
  have to hand-pick a model. Waiting on Michael to confirm which current model he selected.
- Consider decommissioning the old Railway service once he's happy with Cloudflare.

### Status
- ✅ **DONE:** bolt.diy is live on Cloudflare Pages and chat builds apps. Docs live on
  branch `claude/bolt-chat-request-railway-8t0y4x`.
