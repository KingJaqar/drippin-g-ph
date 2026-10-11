# vulnerable-lab/ — TRAINING-ONLY

> **Intentionally vulnerable application — local security training only.**

This app will contain **deliberately insecure code** for learning. Do not deploy it, do not expose it to any
network, do not reuse its code in a real project, and do not put real data in it. It runs on `127.0.0.1`
only (`http://127.0.0.1:3001`, Supabase on `127.0.0.1:55321`).

This folder stays empty until **Phase 5**, and Phase 5 may start only after `secure-main/` passes the
Phase 4 acceptance checklist. Phase 5 copies `secure-main/` here and then removes specific controls, each
marked `INTENTIONAL VULNERABILITY (lesson: …)` in the code.

- What differs from `secure-main/` and why: [`../docs/00-architecture.md`](../docs/00-architecture.md#files-that-differ-the-security-controls)
- Rules of engagement: `../docs/SECURITY-LAB.md` (Phase 6)
