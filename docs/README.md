# Drippin G PH — Security Training Lab

> **Intentionally vulnerable application — local security training only.**
>
> This exact sentence is the persistent UI banner text for `vulnerable-lab/`. It must appear on every page
> of that app and must not be removed or reworded.

## What this is

Drippin G PH is a fictional Philippine streetwear shop (T-shirts, jackets, baggy pants; prices in PHP;
Cash on Delivery only) used as a **teaching resource** for web application security. It comes as two
parallel Next.js + Supabase apps that are identical except for their security controls:

| App | Purpose | Where it may run |
| --- | --- | --- |
| `secure-main/` | The correct reference implementation. Full shop functionality with security controls on. Built and verified **first**. | Local (`127.0.0.1`). Optionally a password-protected private demo (see `06-private-deployment.md`, Phase 7). |
| `vulnerable-lab/` — **TRAINING-ONLY** | A copy of `secure-main/` with selected controls deliberately removed, each one marked `INTENTIONAL VULNERABILITY`. | **Local only, `127.0.0.1` only. Never deployed, never exposed to a network.** |

Learners study each flaw in `vulnerable-lab/`, reproduce it against their own local copy, then read the fix
in `secure-main/` and confirm it holds. The style follows established teaching projects such as OWASP Juice
Shop, DVWA and WebGoat.

## Safety rules (apply to every document and every listing)

1. **Local and private only.** Both apps bind to `127.0.0.1`. `vulnerable-lab/` is never deployed and never
   reachable from the internet or a LAN.
2. **Own system only.** Every test case targets this lab on `localhost`. Nothing here is guidance for testing
   systems you do not own.
3. **Seed data only.** No real customers, no real personal data, no real payments (COD is recorded, never
   processed), no production secrets, no external integrations.
4. **Secrets stay in `.env.local`**, which is git-ignored and never committed. Only `.env.example` files with
   placeholders are committed.
5. **Every intentional flaw is labeled** in code (`// INTENTIONAL VULNERABILITY (lesson: …)`) and in prose,
   and is paired with its fix.

The full scope statement and rules of engagement live in `SECURITY-LAB.md` (Phase 6).

## Repository layout

```text
drippin-g-ph/
├── docs/             ← this guide (the deliverable)
├── plans/            ← planning documents
├── secure-main/      ← reference app, built from the listings in Phases 1–4
└── vulnerable-lab/   ← training-only app, created from secure-main/ in Phase 5
```

The full per-app folder structure is in [`00-architecture.md`](00-architecture.md#4-folder-structure).

## Reading order

Read and build in this order. The governing rule: **`secure-main/` must be fully working and verified
(Phase 4) before any flaw is introduced into `vulnerable-lab/` (Phase 5).**

| # | Document | Phase | Status |
| --- | --- | --- | --- |
| 1 | `README.md` (this file) | 0 | Written |
| 2 | [`00-architecture.md`](00-architecture.md) — stack, diagram, folder structure, shared tooling, `.env.local` contract | 0 | Written |
| 3 | [`01-database.md`](01-database.md) — schema, ER diagram, migrations, RLS posture | 1 | Written |
| 4 | [`02-seed-data.md`](02-seed-data.md) — 13 products, placeholder images, test accounts | 1 | Written |
| 5 | [`03-frontend.md`](03-frontend.md) — design system and every UI component (25 components, all config files) | 2 | Written |
| 6 | `04-api-and-actions.md` — route handlers and server actions, COD checkout, Buy Now | 3 | Not started |
| 7 | `05-local-setup.md` — clone → run → acceptance checklist | 4 | Not started |
| 8 | `SECURITY-LAB.md` — scope, rules of engagement, tools, reporting template | 6 | Not started |
| 9 | `07-vulnerability-catalog.md` — each flaw with OWASP mapping, reproduction, fix, verification | 6 | Not started |
| 10 | `08-remediation-guide.md` — insecure vs. secure side by side, how to diff the apps | 6 | Not started |
| 11 | `06-private-deployment.md` — optional private demo of `secure-main/` only | 7 | Not started |

## Conventions used in this guide

- **Listings are complete.** Each one is labeled with its target path (for example
  `secure-main/app/(shop)/page.tsx`) and contains the whole file. No `// ...` gaps.
- Commands are in fenced `bash` blocks and work in Git Bash on Windows unless a block says otherwise.
- Intentional flaws use the marker `// INTENTIONAL VULNERABILITY (lesson: <name>)` (`-- INTENTIONAL
  VULNERABILITY …` in SQL), with a matching note in the surrounding prose.
- All brand assets are original. No third-party logos, product names, slogans, fonts or photography.
