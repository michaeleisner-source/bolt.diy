# Architecture

How the platform is put together, the layers, where code lives, and who pays for what.
Companion to `CLAUDE.md` and `DECISIONS.md`.

## The two layers (do not mix them)

1. **The platform** — *this repo*, our fork of bolt.diy. The thing a user logs into and builds
   inside. We host it. Adding new **AI providers** happens here.
2. **The customer's project** — whatever the user builds *inside* the platform. Its code lives on
   the **customer's** GitHub; it deploys to the **customer's** Vercel/Netlify; any tools it uses
   (Supabase, Firecrawl, Resend, …) use **that project's own** keys. Never added to the platform.

This split is the whole cost model: our inference cost is zero (BYOK), and the customer owns their
code, hosting, and keys.

## The platform stack

| Concern | Choice | Notes |
| --- | --- | --- |
| App framework | **Remix** targeting **Cloudflare Pages** | `@remix-run/cloudflare`, `functions/[[path]].ts`, `wrangler.toml` |
| Hosting | **Cloudflare Pages** (Git integration) | auto-deploy from `main`; live at `bolt-diy-8bq.pages.dev`. See D-008 |
| AI | **Vercel AI SDK** (`ai`, `@ai-sdk/*`) | streams SSE from the model to the browser |
| Model (default) | **Anthropic Claude Sonnet 4.5** | BYOK — user's key, billed to the user |
| Live preview / runtime | **StackBlitz WebContainers** | runs the built app in-browser. Commercial license needed to resell — D-005 |
| Platform data (accounts, encrypted keys, project metadata) | **Supabase** *(planned, Stage 2)* | not yet wired for platform auth; bolt.diy's existing Supabase hooks are for *customer projects* |
| Customer code | **GitHub** (customer's account) | |
| Customer live sites | **Vercel / Netlify** (customer's account) | one-click deploy from the builder |

## How a chat request flows (why the host mattered)

Browser → `app/routes/api.chat.ts` (`action`) → `streamText` (AI SDK) → Anthropic API →
tokens stream back as SSE → browser renders files/preview. This streaming path is what broke under
Railway's `wrangler pages dev` (`workerd` emulator) with `Custom error: internal error; reference =
...`. On real Cloudflare Pages the same code runs fine (D-008). The `onError` handler in that route
wraps any failure as `Custom error: <message>` — so that prefix in the UI means "read the rest of
the message for the real cause."

## Runtimes: dev vs production (worth remembering)

- `pnpm run dev` (`remix vite:dev`) → app runs in **plain Node.js** (CF bindings emulated via
  `remixCloudflareDevProxy`). Streaming works.
- `wrangler pages dev` (the old Railway image's `dockerstart`) → app runs in **`workerd`**. This is
  what broke streaming in a plain container.
- **Cloudflare Pages (production)** → `workerd` running natively on Cloudflare. Works.

## Keys & secrets

- **Today (single-user):** `ANTHROPIC_API_KEY` is a **Cloudflare Pages Production secret** (server
  side), which also lets the live model list load. Users can alternatively enter a key in the UI
  (stored in the browser) — the BYOK design.
- **Stage 2 (multi-tenant):** each user's key must be stored **encrypted, server-side only, never
  exposed to the browser** (CLAUDE.md core rule). That's a Supabase-backed change, not built yet.

## Where the code lives / update loop

Source of truth = this GitHub repo. Edit → commit → push to a branch → merge to `main` →
Cloudflare auto-builds → live. Revert = check out an earlier commit. `main` = production.

## Build config (Cloudflare Pages)

Framework preset `None`; build command `pnpm run build`; output dir `build/client`; build env
`NODE_VERSION=22`. `wrangler.toml` supplies `compatibility_flags = ["nodejs_compat"]` and the
compatibility date, which Cloudflare Pages honors because `pages_build_output_dir` is set.

## Open architectural items

- **WebContainer commercial license** (or swap to E2B/Daytona/Fly) before reselling — D-005.
- **Supabase for platform auth + per-user encrypted keys + project isolation** — Stage 2.
- Retire Railway once nothing depends on it — D-008.
