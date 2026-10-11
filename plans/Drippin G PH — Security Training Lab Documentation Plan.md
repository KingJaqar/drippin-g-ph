# Drippin G PH — Security Training Lab: Documentation Plan

## Context

The goal is a **teaching resource** (Markdown documentation + code listings, not a running app) for an
isolated, deliberately-vulnerable e-commerce security-training lab called **Drippin G PH**. It exists so a
learner can study five classic web vulnerabilities *paired with their fixes*, in the style of established
teaching projects (OWASP Juice Shop, DVWA, WebGoat). The deliverable is a set of documents that a developer
(or a coding agent) can follow to build two parallel apps — `vulnerable-lab/` (flaws present) and
`secure-main/` (flaws remediated) — that are identical except for the security controls.

This plan covers the **"Docs/guide only"** deliverable chosen by the user — but the docs are **build-complete**:
every phase ships **complete, copy-pasteable code** (full React/Next.js components, full Tailwind, full SQL
migrations and seed, full route handlers / server actions, config files), not just specifications. Following
the guide top to bottom produces the **entire working website — frontend, database, and backend** — for both
`secure-main/` and `vulnerable-lab/`. No app is run or deployed inside this task; **all seven phases are
written to completion.**

### Completeness bar (what "fully functional" means here)
A developer who copies the listings in order gets, with no gaps to fill in:
- **Frontend:** every page and component renders and navigates — landing/hero, category nav + slide-out menu,
  product grid + filters/search, PDP (gallery + buy panel, pill sizes, Add to Bag, Buy Now), cart drawer,
  COD checkout, login/signup + guest mode, account/order history, admin dashboard.
- **Database:** all ten tables, relationships, indexes, RLS policies, migrations, and seed (≥12 products +
  test accounts) — runnable as given.
- **Backend:** every route handler / server action with request + response code, auth/session handling,
  cart + Buy Now logic, COD order creation, order history queries, and admin CRUD — all wired to Supabase.
- Full `package.json`, `tailwind.config`, `next.config`, Supabase client setup, and `.env.local` template.

### Safety scope baked into every document
- Local / private, access-controlled only. **Never exposed to the public internet.**
- Used only against this app itself (a system the operator owns). No guidance for attacking third-party systems.
- Seeded demo data only; no real customer data, no real payments, no production secrets, no external integrations.
- Secrets in environment variables, never committed.
- Every intentional flaw labeled as such in code comments and docs; persistent UI banner:
  *"Intentionally vulnerable application — local security training only."*
- **Supabase recommendation (reconciled with the user's pick):** the vulnerable lab defaults to **local
  Supabase CLI + Docker** (all on 127.0.0.1), because a hosted project would make the injectable endpoints
  internet-reachable and break the isolation rule. A hosted free project is documented only as an optional,
  password-protected private demo for `secure-main/`.

## Deliverable: files written to `C:\Users\User\drippin-g-ph\docs\`

Move the session to `C:\Users\User\drippin-g-ph` at the start of implementation, then create a `docs/`
tree. The documents:

1. **`README.md`** — what the lab is, the safety banner text, the two-app layout, and a reading order.
2. **`00-architecture.md`**
   - ASCII/Mermaid architecture diagram: Browser → Next.js (App Router, server actions/route handlers) →
     Supabase (Postgres / Auth / Storage) → local testing tools (Burp Suite, OWASP ZAP).
   - Tech stack rationale: Next.js + TypeScript, Supabase, Tailwind. Local-first.
   - Full folder structure for **both** `vulnerable-lab/` and `secure-main/`, shown side by side so the only
     differences are the security controls.
3. **`01-database.md`**
   - Schema for: `users`, `categories`, `products`, `product_variants`, `carts`, `cart_items`, `orders`,
     `order_items`, `reviews`, `audit_logs`. Column types, keys, relationships (ER description + Mermaid).
   - Note on RLS: `secure-main/` enables Row Level Security with policies; `vulnerable-lab/` deliberately
     leaves gaps (documented as intentional) for the broken-access-control / IDOR lessons.
   - SQL migration listings (`supabase/migrations/*.sql`) shared by both apps.
4. **`02-seed-data.md`**
   - Seed script (`supabase/seed.sql` + a TS seeding helper) with **≥12 products** across T-shirts, jackets,
     baggy pants — original names, descriptions, PHP prices, sizes, colors, stock counts.
   - Original placeholder image strategy (locally generated SVG/solid-color placeholders; no third-party or
     copyrighted photography). Test accounts (admin + regular users) with documented credentials for the lab.
5. **`03-frontend.md`** — Nike-structure, original-assets build guide
   - Design language (replicate **structure/layout/interaction only**, 100% original assets): full-bleed
     edge-to-edge hero, sticky minimal top nav with slide-out category menu, grid product cards (large image
     on neutral bg, name, category label, price, subtle hover), PDP with left gallery + right buy panel (pill
     size grid, prominent "Add to Bag"), expandable detail accordions, cart drawer, footer.
   - Typography: freely-licensed condensed bold display + neutral body sans (e.g., Google Fonts Archivo /
     Inter). Palette: black/white/neutral + one bold accent. Understated motion.
   - **Originality guardrail**, stated explicitly: no Nike logo/swoosh, brand name, slogans, proprietary
     fonts, product names, or product/marketing imagery.
   - **Component checklist** (the "exact UI build list" offered): navbar + site navigation, slide-out menu,
     landing/home hero, promo sections, product grid/card, filters + search bar, PDP gallery, PDP buy panel
     (pill sizes, **Add to Bag** + **Buy Now**), reviews block, cart drawer, **COD checkout form** (delivery
     details only), login/signup + guest-mode entry, account pages, admin dashboard shell, the persistent
     vulnerability banner.
6. **`04-api-and-actions.md`**
   - Route handler / server action specifications for every workflow: landing page / home + featured,
     site navigation (category nav + slide-out menu routes), product listing with filters (category, size,
     color, price) + search, product detail + related + reviews, **add to cart** + cart CRUD,
     **Buy Now** (direct-to-checkout path that bypasses the cart for a single item), **guest mode** (browse
     and check out without an account; order captured against a guest contact), checkout, auth
     (register/login/logout/profile/order history), and admin CRUD for products/categories/orders/users.
   - **Payment: Cash on Delivery (COD) only — no online/card payments and no payment gateway.** Checkout
     collects delivery details, records the order with `payment_method = 'COD'` and an initial status, and
     never transmits payment data anywhere. This satisfies both the "no real payment processing" safety rule
     and the user's COD-only requirement.
   - Request/response shapes, auth requirements, and where each endpoint sits in both apps.
   - **Lightweight-system note:** keep the stack minimal — App Router server actions/route handlers over the
     Supabase client, no extra state libraries or heavy dependencies; both apps stay small and easy to reason
     about for a beginner.
7. **`05-local-setup.md`**
   - Step-by-step local setup (Windows-friendly): prerequisites (Node, Docker Desktop, Supabase CLI),
     `supabase start`, env vars via `.env.local` (listed, never committed), migrations, seeding, `npm run dev`.
   - How to confirm full functionality works in `secure-main/` **before** studying any flaw (acceptance
     checklist: landing page, navigation and category browsing work; login/signup **and** guest mode work;
     add-to-cart and Buy Now both work; COD checkout produces a valid order record; order history shows it;
     admin CRUD on all entities works; no console errors).
8. **`06-private-deployment.md`**
   - Optional, access-controlled only: Vercel password-protected private demo for `secure-main/` *only*,
     with explicit warnings never to deploy `vulnerable-lab/` anywhere reachable. Keep the vulnerable lab local.
9. **`SECURITY-LAB.md`**
   - Scope & authorization statement, rules of engagement (own-system-only, local-only), test accounts,
     suggested tools (Burp Suite, OWASP ZAP) at a configuration level, and a finding/reporting template.
10. **`07-vulnerability-catalog.md`**
    - One entry per required vulnerability, each mapped to **OWASP Top 10** and **OWASP WSTG** IDs:
      authentication brute-force, SQL injection, blind SQL injection, reflected XSS, stored XSS — plus the
      optional extensions (broken access control, IDOR on orders, price/quantity manipulation at checkout,
      weak session handling, missing security headers).
    - Each entry: (1) vulnerable endpoint/page, (2) the insecure code pattern, (3) a self-contained
      demonstration/test case **run against the lab itself**, (4) potential impact, (5) the secure
      remediation with corrected code, (6) a verification test proving the fix holds.
    - Test cases are written as scoped, lab-only reproductions (e.g., Playwright/HTTP scripts that hit
      `localhost`), framed as "reproduce in the lab, then confirm the fix" — not as generalized attack
      tooling.
11. **`08-remediation-guide.md`**
    - Side-by-side insecure vs. secure code for each flaw (e.g., string-concatenated SQL vs. Supabase query
      builder / parameterized queries; `dangerouslySetInnerHTML` of raw input vs. safe React rendering +
      sanitization + CSP; no rate limiting vs. rate limit + lockout + generic errors + monitoring; missing
      authz checks vs. server-side ownership checks + RLS).
    - A short "how to diff the two apps" section so learners can compare `vulnerable-lab/` and `secure-main/`.

## Writing conventions across all docs
- **Code is complete, not illustrative.** Every component, migration, seed row, and route handler is given in
  full and labeled with its target path (e.g. `secure-main/app/(shop)/page.tsx`). No `// ... rest here`
  ellipses, no pseudo-code, no "left as an exercise." The reader pastes files, runs the setup steps, and the
  site works.
- Beginner-runnable: short steps, explicit commands in fenced `bash`/`sql` blocks, minimal magic.
- Every intentional flaw appears with a clear `// INTENTIONAL VULNERABILITY (lesson: …)` style marker in the
  listing and a matching note in prose.
- PHP currency, Philippine streetwear framing, all-original brand assets.
- Vulnerability detail kept at the level of well-documented OWASP teaching material, always paired with the
  fix and always scoped to the learner's own lab.

## Verification (of the documentation itself)
Since the deliverable is docs, "verification" means the docs are self-consistent and buildable:
- Schema in `01` matches the entities referenced by `04` API specs and `02` seed data.
- Every workflow in the Core functionality list has a corresponding API spec and a UI component.
- Each of the five required vulnerabilities has all six required parts in `07`, a matching side-by-side in
  `08`, and an OWASP mapping.
- The component checklist in `03` covers every page named in the acceptance criteria.
- Setup steps in `05` form a complete path from clone → running `secure-main/` with passing acceptance checks.

## Execution phases — full development roadmap

These phases are both (a) the order in which I will write the documents in this task, and (b) the build
order the docs prescribe. **Each phase delivers the full source for its slice** — enough that by the end of
Phase 7 the two complete, runnable apps exist entirely as copy-pasteable listings across the docs. The
governing rule is the user's constraint: **build and verify full functionality in `secure-main/` first;
introduce flaws into `vulnerable-lab/` only afterward.** All seven phases are completed in this effort.

### Phase 0 — Project setup & foundations ✅ Done (2026-10-11)
- [x] Move session to `C:\Users\User\drippin-g-ph`; create `docs/`, `secure-main/`, `vulnerable-lab/` layout.
- [x] Write `README.md` + `00-architecture.md`: stack, local-first decision, folder structures, architecture
  diagram, safety banner text.
- [x] Define shared tooling: Node/Docker/Supabase CLI prerequisites, `.env.local` contract, lightweight
  dependency list (Next.js, Supabase JS, Tailwind — nothing heavy). (Documented in `00-architecture.md` §5.)

### Phase 1 — Data layer (shared by both apps) ✅ Done (2026-10-11)
- [x] Write `01-database.md`: schema for all ten tables, relationships, ER diagram, and the migration SQL.
- [x] Write `02-seed-data.md`: ≥12 original products across the three categories (PHP prices, sizes, colors,
  stock), original placeholder-image strategy, and test accounts.
- [x] RLS posture documented: `secure-main/` policies on; `vulnerable-lab/` gaps marked intentional.

### Phase 2 — `secure-main/` frontend (full, working, no flaws) ✅ Done (2026-10-11)
- [x] Write `03-frontend.md`: Nike-structure/original-assets design system + the component checklist.
- [x] Build order of the UI: layout + sticky nav + slide-out menu → landing/home (hero, featured, promos) →
  product grid/cards + filters/search → PDP (gallery + buy panel with pill sizes, Add to Bag, Buy Now) →
  cart drawer → COD checkout → auth (login/signup) + guest-mode entry → account/order history →
  admin dashboard shell.

### Phase 3 — `secure-main/` backend & workflows (full functionality)
- Write `04-api-and-actions.md`: every route handler / server action, with COD-only checkout and Buy Now.
- Wire each UI piece to data: browsing, search/filter, add-to-cart + Buy Now, guest + authenticated
  checkout → order record, order history, admin CRUD on all entities.
- Security controls present from the start (parameterized queries, output encoding/CSP, authz + RLS checks,
  rate limiting) — this is the "correct" reference implementation.

### Phase 4 — Verify `secure-main/` end to end
- Write `05-local-setup.md`: clone → `supabase start` → migrate → seed → `npm run dev`.
- Run the acceptance checklist (landing/nav, login/signup + guest, add-to-cart + Buy Now, COD order
  created, order history, admin CRUD, no console errors). **Gate: nothing proceeds to Phase 5 until this
  passes.**

### Phase 5 — Create `vulnerable-lab/` by regressing controls
- Copy `secure-main/` to `vulnerable-lab/`; keep features, schema, and seed data identical.
- Introduce the five required flaws (+ optional extensions), each clearly marked in code comments and docs:
  auth brute-force (remove rate limit/lockout), SQL injection (string-built SQL), blind SQL injection
  (condition-sensitive lookup), reflected XSS (unsafe render of input), stored XSS (review/profile HTML).
- Add the persistent "Intentionally vulnerable application — local security training only" banner and the
  local-only guardrails (no deploy).

### Phase 6 — Security documentation
- Write `SECURITY-LAB.md` (scope, rules of engagement, test accounts, tool config, reporting template).
- Write `07-vulnerability-catalog.md`: per flaw — endpoint, insecure pattern, lab-only reproduction, impact,
  remediation code, verification test — mapped to OWASP Top 10 + WSTG.
- Write `08-remediation-guide.md`: side-by-side insecure vs. secure code + "how to diff the two apps."

### Phase 7 — Consistency pass (documentation verification)
- Run the Verification checks below: schema ↔ API ↔ seed ↔ UI alignment; each of the five vulnerabilities
  has all six parts, a side-by-side, and an OWASP mapping; component checklist covers every acceptance page;
  setup path is complete.
- Optional `06-private-deployment.md`: password-protected private Vercel demo for `secure-main/` only.

## Status

| Phase | Status | Notes |
| --- | --- | --- |
| 0 — Setup & foundations | ✅ Done 2026-10-11 | `docs/README.md`, `docs/00-architecture.md`, placeholder READMEs in `secure-main/` and `vulnerable-lab/`, root `.gitignore`. Re-verified 2026-10-11: npm pins re-checked against the registry, 0 broken anchors, no secrets; fixed `public/images/` missing from the §4 tree, added the config-only differences table (`package.json`, `.env.example`, `config.toml`), and added TRAINING-ONLY labels. |
| 1 — Data layer | ✅ Done 2026-10-11 | `docs/01-database.md` (10 tables, ER diagram, migrations `0001`–`0008`, secure `0007` RLS, per-app `config.toml`) + `docs/02-seed-data.md` (13 products / 42 variants / 6 reviews, SVG placeholder generator, 3 test accounts). Scripted checks: 13 products (5/4/4), 42 unique SKUs, slug↔image↔generator alignment, 0 broken doc links, no secrets. Decisions used: `public.users`↔`auth.users` trigger, guest carts/orders via SECURITY DEFINER token-scoped fns (no anon blanket access), failed logins counted from `audit_logs`, added `orders.guest_token` for guest lookup. |
| 2 — `secure-main/` frontend | ✅ Done 2026-10-11 | `docs/03-frontend.md`: 25 components, 49 labeled code listings, `package.json` + configs, complete design system in `globals.css` `@theme`, all 13 pages. 0 broken anchors, no secrets. Tailwind 4 CSS-configured as planned. |
| 3 — `secure-main/` backend | Not started | Next.js 16: session refresh lives in `proxy.ts` (formerly `middleware.ts`). |
| 4 — Verify `secure-main/` | Not started | Add a check that every listener is on `127.0.0.1`. |
| 5 — `vulnerable-lab/` | Not started | SQLi via lab-only migration `9001_lab_only_unsafe_functions.sql` + `supabase.rpc()`. |
| 6 — Security docs | Not started | |
| 7 — Consistency pass | Not started | |

## Out of scope for this task
- Building/running the Next.js apps or the Supabase instance (this is the docs-only deliverable; the phases
  above are written into the guide as the prescribed build order).
- Any exploit guidance targeting systems other than the learner's own local lab.
