# secure-main/

The **reference implementation** of Drippin G PH: full shop functionality with all security controls on.
It is built and verified first. `vulnerable-lab/` is created from it only after Phase 4 passes.

This folder is empty on purpose until Phase 2. Its contents are created by copying the listings in
`../docs/` in order:

| Phase | Document | Creates |
| --- | --- | --- |
| 1 | `01-database.md`, `02-seed-data.md` | `supabase/migrations/`, `supabase/seed.sql`, `supabase/config.toml`, `scripts/seed.ts`, `scripts/gen-placeholders.ts`, `public/images/` |
| 2 | `03-frontend.md` | `package.json`, config files, `app/`, `components/` |
| 3 | `04-api-and-actions.md` | `lib/`, `app/api/`, `proxy.ts` |
| 4 | `05-local-setup.md` | (verification only) |

Folder structure and `.env.local` contract: [`../docs/00-architecture.md`](../docs/00-architecture.md).
Runs on `http://127.0.0.1:3000`.
