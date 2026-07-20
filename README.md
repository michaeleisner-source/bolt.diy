# BYOK Builder (working name)

A bring-your-own-key AI app builder. Same experience as Lovable/Bolt — describe a change,
the AI builds it, live preview, revert, deploy — except each user brings their own AI API key,
so there is no per-token markup. We sell the **setup and the managed platform**, not the AI.

Built on a fork of **bolt.diy** (open-source Bolt.new). We are building it customer-shaped from
day one; **Michael (Matis) is customer #1.**

> The original bolt.diy project README is preserved at `docs/UPSTREAM_bolt.diy_README.md`.

---

## What this repo is

This is the **platform** — the thing customers log into. It is NOT the projects customers build
inside it (those are separate and live on the customer's own accounts). Keep that distinction
clear; it is the whole cost model.

## The one-line business

The tech is a commodity (bolt.diy is free and open source). Awareness is low and setup friction
is real. Our product is: **"It's already wired up — you just bring a key."** The durable margin is
the onboarding + managed hosting, not the software.

---

## How to navigate these docs

| File | What it holds |
| --- | --- |
| `README.md` | This overview |
| `CLAUDE.md` | Context file auto-loaded by Claude Code. The single source of truth for resuming work. |
| `docs/ARCHITECTURE.md` | The stack, the layers, where code lives, who pays for what |
| `docs/DECISIONS.md` | Why we chose what we chose (decision log) |
| `docs/ROADMAP.md` | The stages and step-by-step checklist, with current status |
| `docs/BUILD_LOG.md` | Running log of every work session |
| `docs/CUSTOMER_EXPLAINER.md` | Plain-language "how it works / what you need" for future customers |

---

## How to resume work without losing context (READ THIS)

The state of this project lives in **these files and the git history — never in a chat window.**
A chat can fill up and reset; the repo cannot. To pick up where we left off in any fresh session:

1. **In Claude Code (recommended):** open this repo. `CLAUDE.md` loads automatically, so it already
   knows the plan. Say "read BUILD_LOG.md and continue from the last entry."
2. **In a fresh claude.ai chat:** paste `CLAUDE.md` (or the latest `BUILD_LOG.md` entry) and say
   "we're resuming this project."

**Rule: end every work session by adding an entry to `docs/BUILD_LOG.md`.** That entry is the
save point.

---

## Current status

**Live.** The platform is deployed on **Cloudflare Pages** at **`bolt-diy-8bq.pages.dev`**,
auto-deploying from `main`, and chat builds apps successfully (Anthropic key set server-side).
Stages 0 (foundations) and 1 (hosting) are essentially done — **next up is Stage 2, the
multi-tenant skeleton (auth + per-user data)**. See `docs/ROADMAP.md` for the exact next step and
`docs/BUILD_LOG.md` for how we got here (including the Railway→Cloudflare fix).
