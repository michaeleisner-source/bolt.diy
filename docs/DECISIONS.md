# Decision Log

Newest at top. Each entry: the decision, why, and any open question it leaves.

---

### D-008 — Platform host is Cloudflare Pages, not Railway
The Railway deploy used the prebuilt image `ghcr.io/stackblitz-labs/bolt.diy:latest`, which starts
the app via `wrangler pages dev` — Cloudflare's **`workerd` emulator**. In a plain container that
emulator crashes the **streaming chat** request with `Custom error: internal error; reference = ...`.
Moving to **real Cloudflare Pages** (Git integration, auto-deploy from `main`) runs `workerd`
natively and the bug disappears; chat builds apps. This supersedes the earlier Railway/Render/Coolify
option in ARCHITECTURE for the platform host. **Open:** eventually decommission the old Railway
services so we stop paying for them.

### D-007 — Build multi-tenant from day one, but keep scope thin
Assume customers in the *architecture* (auth, per-user data, encrypted keys), but do not build
billing, seats, or extras until a second real person asks to pay. Michael is customer #1; his daily
use is the product test. **Open:** none.

### D-006 — Keep all project state in the repo, never in a chat
Chats hit context limits and reset. The knowledge base (`/docs`), `CLAUDE.md`, git history, and a
per-session `BUILD_LOG.md` are the durable memory. Heavy build work happens in **Claude Code**
(repo-as-state + auto-compaction + auto-loaded CLAUDE.md), which is included in the Max plan and
therefore costs no API tokens. **Open:** none.

### D-005 — Reselling is possible but the runtime is not free at resale
bolt.diy **code is MIT** (rebrand/modify/sell is fine). But it runs on **StackBlitz WebContainers**,
which **requires a commercial license for production for-profit use serving customers** (prototypes
and personal use are exempt). So at resale we either (a) license WebContainers from StackBlitz, or
(b) swap in our own sandbox (E2B / Daytona / Fly) and pay per-user compute. Either way, "the engine
is free" stops being true when we sell. **Open:** confirm current WebContainer commercial pricing
with StackBlitz before charging customers. (Not legal advice — verify directly.)

### D-004 — Foundation is a fork of bolt.diy, not a build-from-scratch
bolt.diy already solves the hard parts: chat + live preview, diff/revert, 19+ AI providers
(switchable per prompt), Git + Supabase integration, one-click deploy. Building from zero would
reinvent all of it. We fork and add the multi-tenant + onboarding layer on top. **Open:** none.

### D-003 — Do NOT build the business on hosted bolt.new
Hosted bolt.new is token/credit-metered (not BYOK) — it is the markup model we are undercutting,
and it is their SaaS (not resellable). Fine as a 10-minute UX test drive only. **Open:** none.

### D-002 — The product is the setup/managed service, not the software
The BYOK builder space already has free tools (bolt.diy, Dyad, OpenThorn). Technical users self-host
for free; non-technical users who would pay find "get an API key" intimidating. Our wedge is the
done-for-you setup + hosting + onboarding hand-holding. "Cheaper Lovable via BYOK" alone is not a
moat. **Open:** pick the specific first-buyer niche.

### D-001 — Bring-your-own-key is the core model
Users bring their own AI API key; we add no per-token markup and charge a flat setup/subscription
fee. Removes inference cost from our P&L entirely. Note: an Anthropic **API key** is pay-per-token
and **separate from a Max subscription**. **Open:** none.
