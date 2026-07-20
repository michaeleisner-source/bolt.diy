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

### Status / next step
- ⏳ **Awaiting Michael's choice** before making any change. No fix applied yet.
- Docs `CLAUDE.md` + `docs/BUILD_LOG.md` created and committed on branch
  `claude/bolt-chat-request-railway-8t0y4x`.
