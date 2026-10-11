# 02 — Seed data

> **Intentionally vulnerable application — local security training only.**
> (Applies to `vulnerable-lab/`. See [README](README.md#safety-rules-apply-to-every-document-and-every-listing).)

This document provides the seed data shared by both apps: **13 original products** across the three
categories, their variants and stock, a few plain-text reviews, an original SVG placeholder-image generator,
and the test accounts. All of it is runnable as given against the schema in
[`01-database.md`](01-database.md).

Two pieces make up the seed:

1. **`supabase/seed.sql`** — catalogue rows (categories, products, variants, reviews). Runs automatically on
   `npx supabase db reset`.
2. **`scripts/seed.ts`** — creates the Supabase **Auth** accounts (admin + shoppers). Auth users cannot be
   created from plain SQL, so this short Node script uses the secret key once. Run after `db reset`.

The placeholder images are generated once by **`scripts/gen-placeholders.ts`** into `public/images/`.

## 1. Originality and safety

- **All assets are original.** Product names, descriptions and images are invented for this lab. There is no
  third-party logo, brand name, slogan, proprietary font, or product photography anywhere.
- **Images are generated locally** as plain SVG (a coloured field with the product's name and category). No
  file is downloaded; nothing external is referenced.
- **No real data.** Prices are made up, in PHP. Test-account emails use the reserved `example.test` domain and
  passwords are obvious lab placeholders. None of these are real credentials; they exist only in the local
  stack and are safe to document because the stack is never exposed (see the safety rules in the
  [README](README.md#safety-rules-apply-to-every-document-and-every-listing)).

## 2. `supabase/seed.sql`

Idempotent: it clears the catalogue tables first so `db reset` (and re-runs) always produce the same state.
It does **not** touch `auth.users`, `public.users`, `orders`, or `audit_logs` — accounts come from
`scripts/seed.ts`, and orders/logs are produced by using the app.

```sql
-- supabase/seed.sql
-- Catalogue seed: 3 categories, 13 products, their variants, and sample reviews.
-- Shared verbatim by secure-main/ and vulnerable-lab/.
-- Runs on `npx supabase db reset`. All content is original; prices are in PHP.

begin;

-- Start clean so re-runs are deterministic. Cascades clear variants/reviews.
truncate table
  public.reviews,
  public.cart_items,
  public.order_items,
  public.product_variants,
  public.products,
  public.categories
restart identity cascade;

-- Categories ----------------------------------------------------------------
insert into public.categories (slug, name, description, sort_order) values
  ('tees',    'T-Shirts',    'Heavyweight and boxy tees cut for the Manila heat.', 1),
  ('jackets', 'Jackets',     'Coach jackets, puffers and windbreakers for the -ber months.', 2),
  ('pants',   'Baggy Pants', 'Wide, pleated and cargo fits with room to move.', 3);

-- Products ------------------------------------------------------------------
-- price_php is the shelf price; image_path points at a generated SVG (§4).
with cat as (select id, slug from public.categories)
insert into public.products
  (slug, category_id, name, description, price_php, image_path, is_featured)
select p.slug, cat.id, p.name, p.description, p.price_php, p.image_path, p.is_featured
from (values
  -- T-shirts
  ('alon-oversized-tee',      'tees',    'Alon Oversized Tee',
     'Drop-shoulder heavyweight cotton with a wave-ripple chest print.',
     749.00::numeric,  '/images/alon-oversized-tee.svg',      true),
  ('sampaguita-boxy-tee',     'tees',    'Sampaguita Boxy Tee',
     'Short boxy cut, garment-dyed, with a small embroidered bloom.',
     699.00::numeric,  '/images/sampaguita-boxy-tee.svg',     false),
  ('tsinelas-club-tee',       'tees',    'Tsinelas Club Graphic Tee',
     'Mid-weight tee with a tongue-in-cheek "slipper club" back print.',
     820.00::numeric,  '/images/tsinelas-club-tee.svg',       true),
  ('palengke-heavyweight-tee','tees',    'Palengke Heavyweight Tee',
     '240gsm cotton built to survive the wet market and the wash.',
     890.00::numeric,  '/images/palengke-heavyweight-tee.svg',false),
  ('jeepney-rush-tee',        'tees',    'Jeepney Rush Tee',
     'Relaxed tee with a route-sign inspired front strip.',
     760.00::numeric,  '/images/jeepney-rush-tee.svg',        false),
  -- Jackets
  ('habagat-coach-jacket',    'jackets', 'Habagat Coach Jacket',
     'Water-resistant coach jacket with snap front and lined hood.',
     2450.00::numeric, '/images/habagat-coach-jacket.svg',    true),
  ('amihan-puffer',           'jackets', 'Amihan Puffer',
     'Lightweight recycled-fill puffer for cool northeast-monsoon nights.',
     3990.00::numeric, '/images/amihan-puffer.svg',           true),
  ('lahar-work-jacket',       'jackets', 'Lahar Work Jacket',
     'Boxy canvas work jacket, triple-stitched, ash-grey wash.',
     2990.00::numeric, '/images/lahar-work-jacket.svg',       false),
  ('signal-no5-windbreaker',  'jackets', 'Signal No. 5 Windbreaker',
     'Packable windbreaker named for the strongest storm signal.',
     1990.00::numeric, '/images/signal-no5-windbreaker.svg',  false),
  -- Baggy pants
  ('estero-baggy-denim',      'pants',   'Estero Baggy Denim',
     'Rigid wide-leg denim with a deep rinse and roomy thigh.',
     2190.00::numeric, '/images/estero-baggy-denim.svg',      true),
  ('lakbay-cargo-pants',      'pants',   'Lakbay Cargo Pants',
     'Six-pocket ripstop cargos for the long haul.',
     1890.00::numeric, '/images/lakbay-cargo-pants.svg',      false),
  ('balikbayan-wide-pants',   'pants',   'Balikbayan Wide Pants',
     'Soft twill wide-leg with an elastic-back waist.',
     1750.00::numeric, '/images/balikbayan-wide-pants.svg',   false),
  ('tambay-pleated-trouser',  'pants',   'Tambay Pleated Trouser',
     'Double-pleated trouser that hangs loose for all-day tambay.',
     1650.00::numeric, '/images/tambay-pleated-trouser.svg',  false)
) as p(slug, cat_slug, name, description, price_php, image_path, is_featured)
join cat on cat.slug = p.cat_slug;

-- Variants (size + colour + stock + SKU) ------------------------------------
with prod as (select id, slug from public.products)
insert into public.product_variants (product_id, size, color, sku, stock)
select prod.id, v.size, v.color, v.sku, v.stock
from (values
  -- Alon Oversized Tee
  ('alon-oversized-tee','S','Bone','ALON-S-BON',24),
  ('alon-oversized-tee','M','Bone','ALON-M-BON',30),
  ('alon-oversized-tee','L','Bone','ALON-L-BON',18),
  ('alon-oversized-tee','L','Black','ALON-L-BLK',12),
  ('alon-oversized-tee','XL','Black','ALON-XL-BLK',9),
  -- Sampaguita Boxy Tee
  ('sampaguita-boxy-tee','S','White','SAMP-S-WHT',20),
  ('sampaguita-boxy-tee','M','White','SAMP-M-WHT',22),
  ('sampaguita-boxy-tee','L','Sage','SAMP-L-SAG',14),
  -- Tsinelas Club Graphic Tee
  ('tsinelas-club-tee','M','Black','TSIN-M-BLK',26),
  ('tsinelas-club-tee','L','Black','TSIN-L-BLK',21),
  ('tsinelas-club-tee','XL','Rust','TSIN-XL-RST',8),
  -- Palengke Heavyweight Tee
  ('palengke-heavyweight-tee','M','Clay','PALE-M-CLY',16),
  ('palengke-heavyweight-tee','L','Clay','PALE-L-CLY',16),
  ('palengke-heavyweight-tee','XL','Charcoal','PALE-XL-CHR',10),
  -- Jeepney Rush Tee
  ('jeepney-rush-tee','S','Yellow','JEEP-S-YEL',18),
  ('jeepney-rush-tee','M','Yellow','JEEP-M-YEL',24),
  ('jeepney-rush-tee','L','Navy','JEEP-L-NVY',15),
  -- Habagat Coach Jacket
  ('habagat-coach-jacket','M','Black','HABA-M-BLK',12),
  ('habagat-coach-jacket','L','Black','HABA-L-BLK',10),
  ('habagat-coach-jacket','XL','Olive','HABA-XL-OLV',6),
  -- Amihan Puffer
  ('amihan-puffer','M','Graphite','AMIH-M-GRP',8),
  ('amihan-puffer','L','Graphite','AMIH-L-GRP',7),
  ('amihan-puffer','L','Sand','AMIH-L-SND',5),
  -- Lahar Work Jacket
  ('lahar-work-jacket','M','Ash','LAHA-M-ASH',9),
  ('lahar-work-jacket','L','Ash','LAHA-L-ASH',9),
  ('lahar-work-jacket','XL','Ash','LAHA-XL-ASH',4),
  -- Signal No. 5 Windbreaker
  ('signal-no5-windbreaker','S','Red','SIGN-S-RED',14),
  ('signal-no5-windbreaker','M','Red','SIGN-M-RED',14),
  ('signal-no5-windbreaker','L','Black','SIGN-L-BLK',11),
  -- Estero Baggy Denim
  ('estero-baggy-denim','30','Rinse','ESTE-30-RIN',10),
  ('estero-baggy-denim','32','Rinse','ESTE-32-RIN',12),
  ('estero-baggy-denim','34','Rinse','ESTE-34-RIN',9),
  ('estero-baggy-denim','36','Rinse','ESTE-36-RIN',5),
  -- Lakbay Cargo Pants
  ('lakbay-cargo-pants','30','Khaki','LAKB-30-KHK',11),
  ('lakbay-cargo-pants','32','Khaki','LAKB-32-KHK',13),
  ('lakbay-cargo-pants','34','Black','LAKB-34-BLK',8),
  -- Balikbayan Wide Pants
  ('balikbayan-wide-pants','S','Stone','BALI-S-STN',15),
  ('balikbayan-wide-pants','M','Stone','BALI-M-STN',17),
  ('balikbayan-wide-pants','L','Chocolate','BALI-L-CHO',9),
  -- Tambay Pleated Trouser
  ('tambay-pleated-trouser','30','Slate','TAMB-30-SLT',10),
  ('tambay-pleated-trouser','32','Slate','TAMB-32-SLT',12),
  ('tambay-pleated-trouser','34','Slate','TAMB-34-SLT',7)
) as v(prod_slug, size, color, sku, stock)
join prod on prod.slug = v.prod_slug;

-- Reviews (plain text; author_name is a display name only, no account needed) -
with prod as (select id, slug from public.products)
insert into public.reviews (product_id, author_name, rating, body)
select prod.id, r.author_name, r.rating, r.body
from (values
  ('alon-oversized-tee',   'Ria from QC',    5, 'Fit is perfect oversized, super thick cotton. Lakas maka-tambay.'),
  ('alon-oversized-tee',   'Migs',           4, 'Great tee, runs a little long but I like it that way.'),
  ('habagat-coach-jacket', 'Dondon',         5, 'Kept me dry on the commute. Snaps feel solid.'),
  ('amihan-puffer',        'Kaye',           4, 'Warm and light. Wish it came in more colours.'),
  ('estero-baggy-denim',   'Paolo',          5, 'Proper wide leg, not skinny pretending to be baggy.'),
  ('lakbay-cargo-pants',   'Trish',          3, 'Good pockets but the ripstop is a bit stiff at first.')
) as r(prod_slug, author_name, rating, body)
join prod on prod.slug = r.prod_slug;

commit;
```

**Totals:** 3 categories, 13 products (5 tees, 4 jackets, 4 pants), 42 variants, 6 reviews — comfortably over
the ≥12-product bar, with every category and every featured flag used.

## 3. Test accounts — `scripts/seed.ts`

Auth accounts are created through the Supabase Admin API (the trigger in `0001` then creates each profile in
`public.users`). The admin account is promoted to `role = 'admin'` afterwards.

### Documented lab credentials

These are **placeholders for the local lab only**. They are not real accounts and must never be reused
anywhere that is reachable.

| Role | Email | Password | Notes |
| --- | --- | --- | --- |
| Admin | `admin@example.test` | `Lab-Admin-123!` | Full admin dashboard access |
| Shopper | `shopper@example.test` | `Lab-Shopper-123!` | Regular customer; owns order history |
| Shopper (2nd) | `shopper2@example.test` | `Lab-Shopper2-123!` | Second customer — used to demonstrate the IDOR lesson (one shopper must not read the other's order) |

Guests need no account — the guest-mode entry (Phase 2) and the token-scoped functions in
[`01-database.md` §5](01-database.md#5-guest-access-without-a-hole-in-rls) cover that path.

### The script

```ts
// scripts/seed.ts
// Creates the lab's Auth accounts and promotes the admin. Run AFTER `db reset`.
//   node scripts/seed.ts
// Reads NEXT_PUBLIC_SUPABASE_URL and SUPABASE_SECRET_KEY from .env.local.
// The secret key bypasses RLS and is used ONLY here, never in app code.
import { readFileSync } from 'node:fs';
import { createClient } from '@supabase/supabase-js';

// --- tiny .env.local loader (no extra dependency) --------------------------
function loadEnv(path = '.env.local'): Record<string, string> {
  const out: Record<string, string> = {};
  let text = '';
  try { text = readFileSync(path, 'utf8'); } catch { return out; }
  for (const line of text.split('\n')) {
    const m = line.match(/^\s*([A-Z0-9_]+)\s*=\s*(.*)\s*$/);
    if (m) out[m[1]] = m[2].replace(/^["']|["']$/g, '');
  }
  return out;
}

const env = { ...loadEnv(), ...process.env };
const url = env.NEXT_PUBLIC_SUPABASE_URL;
const secret = env.SUPABASE_SECRET_KEY;

if (!url || !secret) {
  console.error('Missing NEXT_PUBLIC_SUPABASE_URL or SUPABASE_SECRET_KEY in .env.local');
  process.exit(1);
}

// Guard: this script only ever runs against a loopback stack.
if (!/^https?:\/\/(127\.0\.0\.1|localhost)/.test(url)) {
  console.error(`Refusing to seed a non-loopback Supabase URL: ${url}`);
  process.exit(1);
}

const admin = createClient(url, secret, {
  auth: { autoRefreshToken: false, persistSession: false },
});

type Account = {
  email: string;
  password: string;
  display_name: string;
  phone: string;
  role: 'customer' | 'admin';
};

// Lab-only placeholder credentials. Mirrored in docs/02-seed-data.md §3.
const accounts: Account[] = [
  { email: 'admin@example.test',    password: 'Lab-Admin-123!',    display_name: 'Lab Admin',   phone: '0917-000-0001', role: 'admin'    },
  { email: 'shopper@example.test',  password: 'Lab-Shopper-123!',  display_name: 'Lab Shopper', phone: '0917-000-0002', role: 'customer' },
  { email: 'shopper2@example.test', password: 'Lab-Shopper2-123!', display_name: 'Lab Shopper 2', phone: '0917-000-0003', role: 'customer' },
];

async function main() {
  for (const acc of accounts) {
    // Create (or find) the auth user with the email already confirmed.
    const { data, error } = await admin.auth.admin.createUser({
      email: acc.email,
      password: acc.password,
      email_confirm: true,
      user_metadata: { display_name: acc.display_name, phone: acc.phone },
    });

    let userId = data?.user?.id ?? null;

    if (error) {
      // Already exists from a previous run: look it up and reset the password.
      const { data: list } = await admin.auth.admin.listUsers();
      const existing = list?.users.find((u) => u.email === acc.email);
      if (!existing) { console.error(`Could not create or find ${acc.email}:`, error.message); continue; }
      userId = existing.id;
      await admin.auth.admin.updateUserById(userId, {
        password: acc.password,
        user_metadata: { display_name: acc.display_name, phone: acc.phone },
      });
    }

    if (!userId) continue;

    // Promote admins (the 0001 trigger always starts everyone as 'customer').
    const { error: roleErr } = await admin
      .from('users')
      .update({ role: acc.role, display_name: acc.display_name, phone: acc.phone })
      .eq('id', userId);
    if (roleErr) console.error(`Role update failed for ${acc.email}:`, roleErr.message);

    console.log(`seeded ${acc.email} (${acc.role})`);
  }
  console.log('Done. Test accounts are documented in docs/02-seed-data.md §3.');
}

main().catch((e) => { console.error(e); process.exit(1); });
```

## 4. Placeholder images — `scripts/gen-placeholders.ts`

One original SVG per product, written to `public/images/<slug>.svg`. Each is a deterministic coloured field
with the product name and category — no external asset, no download. Re-run any time; output is stable.

```ts
// scripts/gen-placeholders.ts
// Generates one original SVG placeholder per product slug into public/images/.
//   node scripts/gen-placeholders.ts
import { mkdirSync, writeFileSync } from 'node:fs';
import { join } from 'node:path';

// slug -> [display name, category label]. Must match seed.sql §2.
const products: Array<[string, string, string]> = [
  ['alon-oversized-tee', 'Alon Oversized Tee', 'T-Shirt'],
  ['sampaguita-boxy-tee', 'Sampaguita Boxy Tee', 'T-Shirt'],
  ['tsinelas-club-tee', 'Tsinelas Club Graphic Tee', 'T-Shirt'],
  ['palengke-heavyweight-tee', 'Palengke Heavyweight Tee', 'T-Shirt'],
  ['jeepney-rush-tee', 'Jeepney Rush Tee', 'T-Shirt'],
  ['habagat-coach-jacket', 'Habagat Coach Jacket', 'Jacket'],
  ['amihan-puffer', 'Amihan Puffer', 'Jacket'],
  ['lahar-work-jacket', 'Lahar Work Jacket', 'Jacket'],
  ['signal-no5-windbreaker', 'Signal No. 5 Windbreaker', 'Jacket'],
  ['estero-baggy-denim', 'Estero Baggy Denim', 'Baggy Pants'],
  ['lakbay-cargo-pants', 'Lakbay Cargo Pants', 'Baggy Pants'],
  ['balikbayan-wide-pants', 'Balikbayan Wide Pants', 'Baggy Pants'],
  ['tambay-pleated-trouser', 'Tambay Pleated Trouser', 'Baggy Pants'],
];

// Deterministic neutral background per slug (no third-party palette).
function bg(slug: string): string {
  let h = 0;
  for (const ch of slug) h = (h * 31 + ch.charCodeAt(0)) % 360;
  return `hsl(${h} 18% 86%)`;
}

function escapeXml(s: string): string {
  return s.replace(/[<>&'"]/g, (c) =>
    ({ '<': '&lt;', '>': '&gt;', '&': '&amp;', "'": '&apos;', '"': '&quot;' }[c]!));
}

function svg(name: string, category: string, slug: string): string {
  const title = escapeXml(name);
  const cat = escapeXml(category.toUpperCase());
  return `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 1000" role="img" aria-label="${title}">
  <rect width="800" height="1000" fill="${bg(slug)}"/>
  <rect x="60" y="60" width="680" height="880" fill="none" stroke="#111" stroke-width="3" opacity="0.25"/>
  <text x="400" y="120" text-anchor="middle" font-family="Arial, sans-serif" font-size="28" letter-spacing="6" fill="#111" opacity="0.55">${cat}</text>
  <text x="400" y="520" text-anchor="middle" font-family="Arial, sans-serif" font-weight="800" font-size="52" fill="#111">${title}</text>
  <text x="400" y="580" text-anchor="middle" font-family="Arial, sans-serif" font-size="22" fill="#111" opacity="0.5">DRIPPIN G PH · placeholder</text>
</svg>
`;
}

const outDir = join('public', 'images');
mkdirSync(outDir, { recursive: true });
for (const [slug, name, category] of products) {
  writeFileSync(join(outDir, `${slug}.svg`), svg(name, category, slug), 'utf8');
  console.log(`wrote public/images/${slug}.svg`);
}
console.log(`Generated ${products.length} placeholder images.`);
```

A matching `brand-mark.svg` (an original wordmark used by the navbar/footer) and an `og-placeholder.svg` are
added alongside these in Phase 2 when the layout needs them; they follow the same "original SVG only" rule.

## 5. Order of operations

Full setup is in `05-local-setup.md` (Phase 4). The seed-relevant steps, in order:

```bash
npx supabase start            # bring up the local stack (Docker)
npx supabase db reset         # apply migrations 0001–0008 + run seed.sql (catalogue)
node scripts/gen-placeholders.ts   # write public/images/*.svg
node scripts/seed.ts          # create the Auth test accounts + promote admin
```

After this the catalogue has 13 products with images, and the three documented accounts exist. Orders and
audit-log rows appear only once you use the app.

## 6. Consistency with other phases

- Every `slug`, `size`, and `color` used here is referenced by the product pages and filters in
  `03-frontend.md` (Phase 2) and the catalogue/search endpoints in `04-api-and-actions.md` (Phase 3).
- The three accounts back the Phase 4 acceptance checks (admin CRUD as `admin@example.test`; order history as
  `shopper@example.test`) and the IDOR lesson in Phase 5/6 (`shopper2@example.test` must not read
  `shopper@example.test`'s order).
- `image_path` values exactly match the files `gen-placeholders.ts` writes.
```
