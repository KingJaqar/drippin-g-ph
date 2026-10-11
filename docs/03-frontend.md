# 03 — Frontend

> **Intentionally vulnerable application — local security training only.**
> (Applies to `vulnerable-lab/`. See [README](README.md#safety-rules-apply-to-every-document-and-every-listing).)

This document builds the **complete `secure-main/` frontend**: every page, every component, every config
file, in the order they should be created. Copy each listing into the path shown above it; by the end, the
app renders and navigates with no errors. Backend wiring (data fetching, server actions) is in
[`04-api-and-actions.md`](04-api-and-actions.md) (Phase 3).

Build order:

1. [Project config](#1-project-config) — `package.json`, PostCSS, TypeScript, Next.js config
2. [Design system](#2-design-system) — `globals.css` with Tailwind 4 `@theme`, palette, typography
3. [Root layout](#3-root-layout) — `app/layout.tsx`
4. [Layout components](#4-layout-components) — Navbar, SlideOutMenu, Footer
5. [Home / landing](#5-home--landing) — Hero, FeaturedGrid, PromoSection
6. [Product listing](#6-product-listing) — ProductCard, ProductGrid, Filters, SearchBar, category page
7. [Product detail (PDP)](#7-product-detail-page) — Gallery, BuyPanel, SizePills, DetailAccordion, Reviews
8. [Cart](#8-cart-drawer) — CartDrawer, CartLine
9. [Checkout](#9-cod-checkout) — CodCheckoutForm, success page
10. [Auth](#10-auth) — LoginForm, SignupForm, GuestEntry
11. [Account](#11-account-pages) — profile, order history, order detail
12. [Admin](#12-admin-dashboard) — AdminShell, DataTable, EntityForm, CRUD pages
13. [Shared types](#13-shared-types) — `lib/types.ts`

## Originality guardrail

This app **replicates structure, layout and interaction patterns** seen on modern sneaker/streetwear sites
(full-bleed hero, sticky nav, product grid on a neutral background, pill-based size selector, cart drawer).
All **assets are 100% original**:

- **No** Nike swoosh, logo, brand name, slogans, proprietary fonts, product names or photography.
- **Brand name:** Drippin G PH — a fictional Philippine streetwear shop.
- **Typography:** freely licensed (Google Fonts Archivo Black for display, Inter for body).
- **Palette:** black / white / neutral + one bold accent (defined in `globals.css`).
- **Product imagery:** locally generated SVG placeholders (see [`02-seed-data.md` §4](02-seed-data.md#4-placeholder-images--scriptsgen-placeholdersts)).

---

## 1. Project config

### `secure-main/package.json`

```jsonc
{
  "name": "drippin-secure",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev -H 127.0.0.1 -p 3000",
    "build": "next build",
    "start": "next start -H 127.0.0.1 -p 3000",
    "db:start": "npx supabase start",
    "db:stop": "npx supabase stop",
    "db:reset": "npx supabase db reset",
    "seed": "node scripts/seed.ts"
  },
  "dependencies": {
    "next": "16.4.0",
    "react": "19.3.0",
    "react-dom": "19.3.0",
    "@supabase/supabase-js": "2.117.3",
    "@supabase/ssr": "0.12.7"
  },
  "devDependencies": {
    "typescript": "5.9.3",
    "@types/react": "19.3.0",
    "@types/react-dom": "19.3.0",
    "@types/node": "24.19.2",
    "tailwindcss": "4.3.3",
    "@tailwindcss/postcss": "4.3.3",
    "supabase": "2.120.0"
  }
}
```

### `secure-main/postcss.config.mjs`

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

### `secure-main/tsconfig.json`

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["DOM", "DOM.Iterable", "ES2022"],
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "jsx": "preserve",
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "incremental": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    },
    "plugins": [{ "name": "next" }]
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### `secure-main/next.config.ts`

Security headers are set here. The lab version (Phase 5) removes them.

```ts
// next.config.ts  (secure-main)
import type { NextConfig } from "next";

const securityHeaders = [
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "X-Frame-Options", value: "DENY" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" },
  {
    key: "Content-Security-Policy",
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-inline'",         // Next.js needs inline scripts
      "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
      "font-src 'self' https://fonts.gstatic.com",
      "img-src 'self' data:",
      "connect-src 'self' http://127.0.0.1:*",     // local Supabase
      "frame-ancestors 'none'",
    ].join("; "),
  },
];

const config: NextConfig = {
  async headers() {
    return [{ source: "/(.*)", headers: securityHeaders }];
  },
};

export default config;
```

### `secure-main/.env.example`

```bash
# .env.example  (secure-main)
# Copy to .env.local and fill in values from `npx supabase status`.
# .env.local is git-ignored and must never be committed.

NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your-anon-key-here
SUPABASE_SECRET_KEY=your-service-role-key-here
NEXT_PUBLIC_SITE_URL=http://127.0.0.1:3000
NEXT_PUBLIC_LAB_MODE=secure
LOGIN_MAX_ATTEMPTS=5
LOGIN_LOCKOUT_MINUTES=15
```

---

## 2. Design system

Tailwind 4 is configured in CSS, not `tailwind.config`. The `@theme` block in `globals.css` defines the
design tokens; utilities like `bg-primary`, `text-accent` reference them.

### `secure-main/app/globals.css`

```css
/* app/globals.css */
@import "tailwindcss";

/* Google Fonts — Archivo Black (display) + Inter (body) */
@import url("https://fonts.googleapis.com/css2?family=Archivo+Black&family=Inter:wght@400;500;600;700&display=swap");

@theme {
  /* Palette: black/white/neutral + one bold accent */
  --color-primary: #111111;
  --color-primary-light: #2a2a2a;
  --color-surface: #ffffff;
  --color-surface-alt: #f5f5f5;
  --color-border: #e5e5e5;
  --color-muted: #737373;
  --color-accent: #e63946;          /* bold red — call-to-action buttons */
  --color-accent-hover: #c1121f;
  --color-success: #2d6a4f;
  --color-warning: #e9c46a;
  --color-error: #d62828;

  /* Typography */
  --font-display: "Archivo Black", "Arial Black", sans-serif;
  --font-body: "Inter", ui-sans-serif, system-ui, sans-serif;

  /* Spacing */
  --gutter: 1rem;                   /* 16px side gutter — phone-safe */
  --max-content: 80rem;             /* 1280px content cap */

  /* Transitions */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
}

/* Base resets */
*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
  -webkit-font-smoothing: antialiased;
}

body {
  margin: 0;
  font-family: var(--font-body);
  color: var(--color-primary);
  background-color: var(--color-surface);
  line-height: 1.5;
}

/* Display headings */
.heading-display {
  font-family: var(--font-display);
  text-transform: uppercase;
  letter-spacing: 0.04em;
  line-height: 1.1;
}

/* Utility: full-bleed container */
.container-content {
  width: 100%;
  max-width: var(--max-content);
  margin-inline: auto;
  padding-inline: var(--gutter);
}

/* Utility: screen-reader only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Smooth slide for drawers & menus */
.slide-in {
  transform: translateX(100%);
  transition: transform 0.35s var(--ease-out-expo);
}
.slide-in.open {
  transform: translateX(0);
}

/* Pill size buttons */
.size-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 3rem;
  padding: 0.5rem 0.75rem;
  border: 1.5px solid var(--color-border);
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s ease;
  background: var(--color-surface);
  color: var(--color-primary);
}
.size-pill:hover {
  border-color: var(--color-primary);
}
.size-pill.selected {
  background: var(--color-primary);
  color: var(--color-surface);
  border-color: var(--color-primary);
}
.size-pill.out-of-stock {
  opacity: 0.35;
  cursor: not-allowed;
  text-decoration: line-through;
}
```

---

## 3. Root layout

### `secure-main/app/layout.tsx`

```tsx
// app/layout.tsx  (secure-main)
import type { Metadata } from "next";
import "./globals.css";
import { Navbar } from "@/components/layout/Navbar";
import { Footer } from "@/components/layout/Footer";

export const metadata: Metadata = {
  title: "Drippin G PH",
  description: "Philippine streetwear — T-shirts, jackets, baggy pants. Local security training reference build.",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className="flex min-h-screen flex-col">
        <Navbar />
        <main className="flex-1">{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

---

## 4. Layout components

### `secure-main/components/layout/Navbar.tsx`

Sticky top nav: wordmark left, category links centre, icons right (search, cart, account). A hamburger toggles
the `SlideOutMenu` on mobile.

```tsx
// components/layout/Navbar.tsx
"use client";

import Link from "next/link";
import { useState } from "react";
import { SlideOutMenu } from "./SlideOutMenu";

const categories = [
  { name: "T-Shirts", href: "/c/tees" },
  { name: "Jackets", href: "/c/jackets" },
  { name: "Baggy Pants", href: "/c/pants" },
];

export function Navbar() {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <>
      <header className="sticky top-0 z-40 border-b border-border bg-surface/95 backdrop-blur">
        <nav className="container-content flex h-14 items-center justify-between gap-4">
          {/* Hamburger — mobile only */}
          <button
            className="mr-2 flex flex-col gap-1 md:hidden"
            onClick={() => setMenuOpen(true)}
            aria-label="Open menu"
          >
            <span className="block h-0.5 w-5 bg-primary" />
            <span className="block h-0.5 w-5 bg-primary" />
            <span className="block h-0.5 w-3.5 bg-primary" />
          </button>

          {/* Wordmark */}
          <Link href="/" className="heading-display text-lg tracking-widest whitespace-nowrap">
            DRIPPIN G PH
          </Link>

          {/* Desktop category links */}
          <ul className="hidden md:flex items-center gap-6 text-sm font-medium uppercase tracking-wider">
            {categories.map((c) => (
              <li key={c.href}>
                <Link href={c.href} className="hover:text-accent transition-colors">
                  {c.name}
                </Link>
              </li>
            ))}
          </ul>

          {/* Icon cluster */}
          <div className="flex items-center gap-3">
            <Link href="/products?q=" aria-label="Search" className="hover:text-accent transition-colors">
              {/* Magnifying glass icon (inline SVG, no dependency) */}
              <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24">
                <circle cx="11" cy="11" r="8" />
                <path d="m21 21-4.3-4.3" />
              </svg>
            </Link>
            <Link href="/account" aria-label="Account" className="hover:text-accent transition-colors">
              <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24">
                <circle cx="12" cy="8" r="4" />
                <path d="M4 21v-1a6 6 0 0 1 12 0v1" />
              </svg>
            </Link>
            {/* Cart icon — placeholder count; wired in Phase 3 */}
            <button aria-label="Open cart" className="relative hover:text-accent transition-colors">
              <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24">
                <path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z" />
                <path d="M3 6h18" />
                <path d="M16 10a4 4 0 0 1-8 0" />
              </svg>
            </button>
          </div>
        </nav>
      </header>

      <SlideOutMenu open={menuOpen} onClose={() => setMenuOpen(false)} categories={categories} />
    </>
  );
}
```

### `secure-main/components/layout/SlideOutMenu.tsx`

Full-screen slide-out from the left on mobile. Closes on backdrop click, Escape, or the × button.

```tsx
// components/layout/SlideOutMenu.tsx
"use client";

import Link from "next/link";
import { useEffect, useRef } from "react";

type Props = {
  open: boolean;
  onClose: () => void;
  categories: { name: string; href: string }[];
};

export function SlideOutMenu({ open, onClose, categories }: Props) {
  const panelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    function handleKey(e: KeyboardEvent) {
      if (e.key === "Escape") onClose();
    }
    if (open) {
      document.addEventListener("keydown", handleKey);
      document.body.style.overflow = "hidden";
    }
    return () => {
      document.removeEventListener("keydown", handleKey);
      document.body.style.overflow = "";
    };
  }, [open, onClose]);

  return (
    <>
      {/* Backdrop */}
      {open && (
        <div
          className="fixed inset-0 z-50 bg-black/40 backdrop-blur-sm"
          onClick={onClose}
          aria-hidden
        />
      )}

      {/* Panel */}
      <div
        ref={panelRef}
        className={`fixed inset-y-0 left-0 z-50 w-72 bg-surface shadow-xl transition-transform duration-300 ${
          open ? "translate-x-0" : "-translate-x-full"
        }`}
        style={{ transitionTimingFunction: "var(--ease-out-expo)" }}
      >
        <div className="flex h-14 items-center justify-between px-4 border-b border-border">
          <span className="heading-display text-sm tracking-widest">MENU</span>
          <button onClick={onClose} aria-label="Close menu" className="p-1 hover:text-accent">
            <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24">
              <path d="M18 6 6 18M6 6l12 12" />
            </svg>
          </button>
        </div>

        <nav className="flex flex-col p-4 gap-2">
          <Link href="/" onClick={onClose} className="py-2 text-sm font-semibold uppercase tracking-wider hover:text-accent">
            Home
          </Link>
          {categories.map((c) => (
            <Link
              key={c.href}
              href={c.href}
              onClick={onClose}
              className="py-2 text-sm font-semibold uppercase tracking-wider hover:text-accent"
            >
              {c.name}
            </Link>
          ))}
          <Link href="/products" onClick={onClose} className="py-2 text-sm font-semibold uppercase tracking-wider hover:text-accent">
            All Products
          </Link>

          <hr className="my-3 border-border" />

          <Link href="/login" onClick={onClose} className="py-2 text-sm font-medium hover:text-accent">
            Sign In
          </Link>
          <Link href="/signup" onClick={onClose} className="py-2 text-sm font-medium hover:text-accent">
            Create Account
          </Link>
        </nav>
      </div>
    </>
  );
}
```

### `secure-main/components/layout/Footer.tsx`

```tsx
// components/layout/Footer.tsx
import Link from "next/link";

export function Footer() {
  return (
    <footer className="border-t border-border bg-primary text-surface">
      <div className="container-content py-10">
        <div className="grid gap-8 md:grid-cols-3">
          {/* Brand */}
          <div>
            <p className="heading-display text-lg tracking-widest mb-2">DRIPPIN G PH</p>
            <p className="text-sm text-surface/60">
              Philippine streetwear. T-shirts, jackets, baggy pants. Cash on Delivery only.
            </p>
          </div>

          {/* Shop links */}
          <div>
            <p className="text-xs font-semibold uppercase tracking-wider text-surface/50 mb-3">Shop</p>
            <ul className="space-y-1 text-sm">
              <li><Link href="/c/tees" className="hover:text-accent transition-colors">T-Shirts</Link></li>
              <li><Link href="/c/jackets" className="hover:text-accent transition-colors">Jackets</Link></li>
              <li><Link href="/c/pants" className="hover:text-accent transition-colors">Baggy Pants</Link></li>
            </ul>
          </div>

          {/* Info */}
          <div>
            <p className="text-xs font-semibold uppercase tracking-wider text-surface/50 mb-3">Info</p>
            <ul className="space-y-1 text-sm">
              <li><Link href="/account" className="hover:text-accent transition-colors">My Account</Link></li>
              <li><Link href="/login" className="hover:text-accent transition-colors">Sign In</Link></li>
            </ul>
          </div>
        </div>

        <div className="mt-8 border-t border-surface/10 pt-4 text-xs text-surface/40 text-center">
          Drippin G PH &copy; {new Date().getFullYear()} &middot; Local training reference build &middot;
          Not a real store
        </div>
      </div>
    </footer>
  );
}
```

---

## 5. Home / landing

### `secure-main/app/(shop)/page.tsx`

The home page: a full-bleed hero, a featured product grid, and a promo section.

```tsx
// app/(shop)/page.tsx
import { Hero } from "@/components/home/Hero";
import { FeaturedGrid } from "@/components/home/FeaturedGrid";
import { PromoSection } from "@/components/home/PromoSection";

export default function HomePage() {
  return (
    <>
      <Hero />
      <FeaturedGrid />
      <PromoSection />
    </>
  );
}
```

### `secure-main/app/(shop)/layout.tsx`

The shop layout — wraps all shop pages with the common structure.

```tsx
// app/(shop)/layout.tsx
export default function ShopLayout({ children }: { children: React.ReactNode }) {
  return <>{children}</>;
}
```

### `secure-main/components/home/Hero.tsx`

Full-bleed hero with a large display heading and a CTA button.

```tsx
// components/home/Hero.tsx
import Link from "next/link";

export function Hero() {
  return (
    <section className="relative flex min-h-[70vh] items-center justify-center bg-primary text-surface overflow-hidden">
      {/* Background pattern — purely decorative, original */}
      <div
        className="absolute inset-0 opacity-[0.04]"
        style={{
          backgroundImage:
            "repeating-linear-gradient(45deg, transparent, transparent 40px, currentColor 40px, currentColor 41px)",
        }}
        aria-hidden
      />

      <div className="relative z-10 text-center px-4 max-w-3xl">
        <p className="text-xs font-semibold uppercase tracking-[0.3em] text-surface/50 mb-4">
          Philippine Streetwear
        </p>
        <h1 className="heading-display text-5xl sm:text-7xl md:text-8xl mb-6">
          DRIPPIN G PH
        </h1>
        <p className="text-lg sm:text-xl text-surface/70 max-w-md mx-auto mb-8">
          Heavyweight tees, coaches &amp; puffers, and wide-leg pants.
          All original. Cash on Delivery only.
        </p>
        <Link
          href="/products"
          className="inline-block bg-accent hover:bg-accent-hover text-surface font-semibold uppercase tracking-wider text-sm px-8 py-3 rounded transition-colors"
        >
          Shop All
        </Link>
      </div>
    </section>
  );
}
```

### `secure-main/components/home/FeaturedGrid.tsx`

Shows featured products in a 2×2 (mobile) / 4-col grid. Products are fetched from the database in Phase 3;
this static version uses the slugs from seed data so the grid is visually complete immediately.

```tsx
// components/home/FeaturedGrid.tsx
import Link from "next/link";

// Placeholder data — replaced by a server fetch in Phase 3.
const featured = [
  { slug: "alon-oversized-tee", name: "Alon Oversized Tee", price: 749, image: "/images/alon-oversized-tee.svg" },
  { slug: "tsinelas-club-tee", name: "Tsinelas Club Graphic Tee", price: 820, image: "/images/tsinelas-club-tee.svg" },
  { slug: "habagat-coach-jacket", name: "Habagat Coach Jacket", price: 2450, image: "/images/habagat-coach-jacket.svg" },
  { slug: "amihan-puffer", name: "Amihan Puffer", price: 3990, image: "/images/amihan-puffer.svg" },
  { slug: "estero-baggy-denim", name: "Estero Baggy Denim", price: 2190, image: "/images/estero-baggy-denim.svg" },
];

function formatPHP(n: number) {
  return `₱${n.toLocaleString("en-PH", { minimumFractionDigits: 2 })}`;
}

export function FeaturedGrid() {
  return (
    <section className="container-content py-12">
      <h2 className="heading-display text-2xl sm:text-3xl mb-8">Featured</h2>

      <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4">
        {featured.map((p) => (
          <Link
            key={p.slug}
            href={`/products/${p.slug}`}
            className="group block"
          >
            <div className="aspect-[3/4] bg-surface-alt rounded overflow-hidden mb-3">
              {/* eslint-disable-next-line @next/next/no-img-element */}
              <img
                src={p.image}
                alt={p.name}
                className="h-full w-full object-contain p-4 transition-transform duration-300 group-hover:scale-105"
              />
            </div>
            <p className="text-sm font-semibold leading-tight mb-1">{p.name}</p>
            <p className="text-sm text-muted">{formatPHP(p.price)}</p>
          </Link>
        ))}
      </div>
    </section>
  );
}
```

### `secure-main/components/home/PromoSection.tsx`

A text-image split promo block. Content is original.

```tsx
// components/home/PromoSection.tsx
import Link from "next/link";

export function PromoSection() {
  return (
    <section className="bg-surface-alt">
      <div className="container-content py-16 grid md:grid-cols-2 gap-8 items-center">
        <div>
          <p className="text-xs font-semibold uppercase tracking-[0.2em] text-muted mb-2">New Drop</p>
          <h2 className="heading-display text-3xl sm:text-4xl mb-4">Monsoon Ready</h2>
          <p className="text-muted mb-6 max-w-md">
            Water-resistant coaches, packable windbreakers and heavyweight tees that shrug off the rain.
            Built for the commute.
          </p>
          <Link
            href="/c/jackets"
            className="inline-block border-2 border-primary text-primary font-semibold uppercase tracking-wider text-sm px-6 py-2.5 rounded hover:bg-primary hover:text-surface transition-colors"
          >
            Shop Jackets
          </Link>
        </div>
        <div className="aspect-[4/3] bg-surface rounded overflow-hidden flex items-center justify-center">
          {/* eslint-disable-next-line @next/next/no-img-element */}
          <img
            src="/images/habagat-coach-jacket.svg"
            alt="Habagat Coach Jacket"
            className="h-3/4 w-3/4 object-contain"
          />
        </div>
      </div>
    </section>
  );
}
```

---

## 6. Product listing

### `secure-main/components/product/ProductCard.tsx`

A single product card used in grids: large image on neutral bg, name, category, price, subtle hover scale.

```tsx
// components/product/ProductCard.tsx
import Link from "next/link";
import type { Product } from "@/lib/types";

function formatPHP(n: number) {
  return `₱${n.toLocaleString("en-PH", { minimumFractionDigits: 2 })}`;
}

export function ProductCard({ product }: { product: Product }) {
  return (
    <Link href={`/products/${product.slug}`} className="group block">
      <div className="aspect-[3/4] bg-surface-alt rounded overflow-hidden mb-3">
        {/* eslint-disable-next-line @next/next/no-img-element */}
        <img
          src={product.image_path || "/images/placeholder.svg"}
          alt={product.name}
          className="h-full w-full object-contain p-4 transition-transform duration-300 group-hover:scale-105"
        />
      </div>
      <p className="text-xs font-semibold uppercase tracking-wider text-muted mb-1">
        {product.category_name}
      </p>
      <p className="text-sm font-semibold leading-tight mb-1">{product.name}</p>
      <p className="text-sm text-muted">{formatPHP(product.price_php)}</p>
    </Link>
  );
}
```

### `secure-main/components/product/ProductGrid.tsx`

Renders a grid of `ProductCard`s.

```tsx
// components/product/ProductGrid.tsx
import { ProductCard } from "./ProductCard";
import type { Product } from "@/lib/types";

export function ProductGrid({ products }: { products: Product[] }) {
  if (products.length === 0) {
    return <p className="py-12 text-center text-muted">No products found.</p>;
  }
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 sm:gap-6">
      {products.map((p) => (
        <ProductCard key={p.id} product={p} />
      ))}
    </div>
  );
}
```

### `secure-main/components/product/Filters.tsx`

Sidebar/top-bar filters: category, size, colour, price range. Values hardcoded to match seed data. In Phase 3
these drive search-param-based filtering.

```tsx
// components/product/Filters.tsx
"use client";

import { useRouter, useSearchParams } from "next/navigation";
import { useCallback } from "react";

const sizes = ["S", "M", "L", "XL", "30", "32", "34", "36"];
const colors = ["Black", "Bone", "White", "Sage", "Rust", "Clay", "Charcoal", "Yellow", "Navy",
                "Olive", "Graphite", "Sand", "Ash", "Red", "Rinse", "Khaki", "Stone", "Chocolate", "Slate"];
const categoryOptions = [
  { slug: "", label: "All" },
  { slug: "tees", label: "T-Shirts" },
  { slug: "jackets", label: "Jackets" },
  { slug: "pants", label: "Baggy Pants" },
];

export function Filters() {
  const router = useRouter();
  const params = useSearchParams();

  const set = useCallback(
    (key: string, value: string) => {
      const next = new URLSearchParams(params.toString());
      if (value) next.set(key, value);
      else next.delete(key);
      router.push(`/products?${next.toString()}`);
    },
    [router, params],
  );

  const active = (key: string) => params.get(key) ?? "";

  return (
    <aside className="space-y-6 text-sm">
      {/* Category */}
      <div>
        <p className="font-semibold uppercase tracking-wider text-xs text-muted mb-2">Category</p>
        <ul className="space-y-1">
          {categoryOptions.map((c) => (
            <li key={c.slug}>
              <button
                onClick={() => set("category", c.slug)}
                className={`w-full text-left py-1 px-2 rounded hover:bg-surface-alt transition-colors ${
                  active("category") === c.slug ? "font-bold" : ""
                }`}
              >
                {c.label}
              </button>
            </li>
          ))}
        </ul>
      </div>

      {/* Size */}
      <div>
        <p className="font-semibold uppercase tracking-wider text-xs text-muted mb-2">Size</p>
        <div className="flex flex-wrap gap-1.5">
          {sizes.map((s) => (
            <button
              key={s}
              onClick={() => set("size", active("size") === s ? "" : s)}
              className={`size-pill text-xs ${active("size") === s ? "selected" : ""}`}
            >
              {s}
            </button>
          ))}
        </div>
      </div>

      {/* Colour */}
      <div>
        <p className="font-semibold uppercase tracking-wider text-xs text-muted mb-2">Colour</p>
        <div className="flex flex-wrap gap-1.5">
          {colors.slice(0, 8).map((c) => (
            <button
              key={c}
              onClick={() => set("color", active("color") === c ? "" : c)}
              className={`text-xs px-2 py-1 rounded border transition-colors ${
                active("color") === c
                  ? "border-primary bg-primary text-surface"
                  : "border-border hover:border-primary"
              }`}
            >
              {c}
            </button>
          ))}
        </div>
      </div>

      {/* Sort */}
      <div>
        <p className="font-semibold uppercase tracking-wider text-xs text-muted mb-2">Sort</p>
        <select
          value={active("sort")}
          onChange={(e) => set("sort", e.target.value)}
          className="w-full border border-border rounded px-2 py-1.5 text-sm bg-surface"
        >
          <option value="">Newest</option>
          <option value="price_asc">Price: Low → High</option>
          <option value="price_desc">Price: High → Low</option>
          <option value="name">Name A → Z</option>
        </select>
      </div>
    </aside>
  );
}
```

### `secure-main/components/product/SearchBar.tsx`

```tsx
// components/product/SearchBar.tsx
"use client";

import { useRouter, useSearchParams } from "next/navigation";
import { useState } from "react";

export function SearchBar() {
  const router = useRouter();
  const params = useSearchParams();
  const [value, setValue] = useState(params.get("q") ?? "");

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const next = new URLSearchParams(params.toString());
    if (value.trim()) next.set("q", value.trim());
    else next.delete("q");
    router.push(`/products?${next.toString()}`);
  }

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <input
        type="search"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Search products…"
        className="flex-1 border border-border rounded px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-accent/30"
      />
      <button
        type="submit"
        className="bg-primary text-surface font-semibold text-sm px-4 py-2 rounded hover:bg-primary-light transition-colors"
      >
        Search
      </button>
    </form>
  );
}
```

### `secure-main/app/(shop)/products/page.tsx`

Product listing page with filters, search, and grid. In the **secure** version, the search term is rendered
as safe React text — never `dangerouslySetInnerHTML`. The lab replaces this with an unsafe render (Phase 5).

```tsx
// app/(shop)/products/page.tsx  (secure-main)
import { Suspense } from "react";
import { ProductGrid } from "@/components/product/ProductGrid";
import { Filters } from "@/components/product/Filters";
import { SearchBar } from "@/components/product/SearchBar";
import type { Product } from "@/lib/types";

// Placeholder data — replaced by a server fetch in Phase 3.
const allProducts: Product[] = [
  { id: "1", slug: "alon-oversized-tee", name: "Alon Oversized Tee", description: "", price_php: 749, image_path: "/images/alon-oversized-tee.svg", category_name: "T-Shirts", is_featured: true, is_active: true },
  { id: "2", slug: "sampaguita-boxy-tee", name: "Sampaguita Boxy Tee", description: "", price_php: 699, image_path: "/images/sampaguita-boxy-tee.svg", category_name: "T-Shirts", is_featured: false, is_active: true },
  { id: "3", slug: "tsinelas-club-tee", name: "Tsinelas Club Graphic Tee", description: "", price_php: 820, image_path: "/images/tsinelas-club-tee.svg", category_name: "T-Shirts", is_featured: true, is_active: true },
  { id: "4", slug: "habagat-coach-jacket", name: "Habagat Coach Jacket", description: "", price_php: 2450, image_path: "/images/habagat-coach-jacket.svg", category_name: "Jackets", is_featured: true, is_active: true },
  { id: "5", slug: "estero-baggy-denim", name: "Estero Baggy Denim", description: "", price_php: 2190, image_path: "/images/estero-baggy-denim.svg", category_name: "Baggy Pants", is_featured: true, is_active: true },
];

export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ q?: string; category?: string; size?: string; color?: string; sort?: string }>;
}) {
  const params = await searchParams;
  const q = params.q ?? "";

  // Placeholder client-side filter — replaced by a Supabase query in Phase 3.
  let products = allProducts;
  if (q) {
    const lower = q.toLowerCase();
    products = products.filter(
      (p) => p.name.toLowerCase().includes(lower) || p.description.toLowerCase().includes(lower),
    );
  }

  return (
    <div className="container-content py-8">
      <h1 className="heading-display text-2xl sm:text-3xl mb-6">
        {q ? (
          <>
            Results for &ldquo;<span className="text-accent">{q}</span>&rdquo;
          </>
        ) : (
          "All Products"
        )}
      </h1>

      <div className="mb-6">
        <Suspense>
          <SearchBar />
        </Suspense>
      </div>

      <div className="grid md:grid-cols-[220px_1fr] gap-8">
        <Suspense>
          <Filters />
        </Suspense>
        <ProductGrid products={products} />
      </div>
    </div>
  );
}
```

### `secure-main/app/(shop)/c/[category]/page.tsx`

Category listing page — reuses the same components.

```tsx
// app/(shop)/c/[category]/page.tsx
import { notFound } from "next/navigation";
import { ProductGrid } from "@/components/product/ProductGrid";
import type { Product } from "@/lib/types";

const categoryMeta: Record<string, string> = {
  tees: "T-Shirts",
  jackets: "Jackets",
  pants: "Baggy Pants",
};

// Placeholder data — replaced by a server fetch in Phase 3.
const allProducts: Product[] = [
  { id: "1", slug: "alon-oversized-tee", name: "Alon Oversized Tee", description: "", price_php: 749, image_path: "/images/alon-oversized-tee.svg", category_name: "T-Shirts", is_featured: true, is_active: true },
  { id: "2", slug: "habagat-coach-jacket", name: "Habagat Coach Jacket", description: "", price_php: 2450, image_path: "/images/habagat-coach-jacket.svg", category_name: "Jackets", is_featured: true, is_active: true },
  { id: "3", slug: "estero-baggy-denim", name: "Estero Baggy Denim", description: "", price_php: 2190, image_path: "/images/estero-baggy-denim.svg", category_name: "Baggy Pants", is_featured: true, is_active: true },
];

export default async function CategoryPage({
  params,
}: {
  params: Promise<{ category: string }>;
}) {
  const { category } = await params;
  const label = categoryMeta[category];
  if (!label) notFound();

  // Placeholder filter — replaced by Supabase query in Phase 3.
  const products = allProducts.filter((p) => p.category_name === label);

  return (
    <div className="container-content py-8">
      <h1 className="heading-display text-2xl sm:text-3xl mb-6">{label}</h1>
      <ProductGrid products={products} />
    </div>
  );
}
```

---

## 7. Product detail page

### `secure-main/app/(shop)/products/[slug]/page.tsx`

```tsx
// app/(shop)/products/[slug]/page.tsx
import { notFound } from "next/navigation";
import { Gallery } from "@/components/product/Gallery";
import { BuyPanel } from "@/components/product/BuyPanel";
import { DetailAccordion } from "@/components/product/DetailAccordion";
import { Reviews } from "@/components/product/Reviews";
import type { ProductDetail, Variant, Review } from "@/lib/types";

// Placeholder data — replaced by server fetch in Phase 3.
const placeholderProduct: ProductDetail = {
  id: "1", slug: "alon-oversized-tee", name: "Alon Oversized Tee",
  description: "Drop-shoulder heavyweight cotton with a wave-ripple chest print.",
  price_php: 749, image_path: "/images/alon-oversized-tee.svg",
  category_name: "T-Shirts", is_featured: true, is_active: true,
};

const placeholderVariants: Variant[] = [
  { id: "v1", size: "S", color: "Bone", sku: "ALON-S-BON", stock: 24 },
  { id: "v2", size: "M", color: "Bone", sku: "ALON-M-BON", stock: 30 },
  { id: "v3", size: "L", color: "Bone", sku: "ALON-L-BON", stock: 18 },
  { id: "v4", size: "L", color: "Black", sku: "ALON-L-BLK", stock: 12 },
  { id: "v5", size: "XL", color: "Black", sku: "ALON-XL-BLK", stock: 0 },
];

const placeholderReviews: Review[] = [
  { id: "r1", author_name: "Ria from QC", rating: 5, body: "Fit is perfect oversized, super thick cotton. Lakas maka-tambay.", created_at: "2026-09-15" },
  { id: "r2", author_name: "Migs", rating: 4, body: "Great tee, runs a little long but I like it that way.", created_at: "2026-09-20" },
];

export default async function PDPPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  // Phase 3 replaces this with a real Supabase query.
  if (slug !== placeholderProduct.slug) notFound();

  const product = placeholderProduct;
  const variants = placeholderVariants;
  const reviews = placeholderReviews;

  return (
    <div className="container-content py-8">
      <div className="grid md:grid-cols-2 gap-8 lg:gap-12">
        <Gallery images={[product.image_path]} name={product.name} />
        <BuyPanel product={product} variants={variants} />
      </div>

      <DetailAccordion description={product.description} />

      <Reviews reviews={reviews} productId={product.id} />
    </div>
  );
}
```

### `secure-main/components/product/Gallery.tsx`

Left-side image gallery. Single image for now (seed products have one SVG each); the component handles
multiple images for future extensibility.

```tsx
// components/product/Gallery.tsx
"use client";

import { useState } from "react";

export function Gallery({ images, name }: { images: string[]; name: string }) {
  const [active, setActive] = useState(0);
  const src = images[active] || "/images/placeholder.svg";

  return (
    <div className="space-y-3">
      <div className="aspect-[3/4] bg-surface-alt rounded overflow-hidden flex items-center justify-center">
        {/* eslint-disable-next-line @next/next/no-img-element */}
        <img src={src} alt={name} className="h-4/5 w-4/5 object-contain" />
      </div>

      {images.length > 1 && (
        <div className="flex gap-2">
          {images.map((img, i) => (
            <button
              key={i}
              onClick={() => setActive(i)}
              className={`w-16 h-16 rounded border-2 overflow-hidden ${
                active === i ? "border-primary" : "border-border"
              }`}
            >
              {/* eslint-disable-next-line @next/next/no-img-element */}
              <img src={img} alt={`${name} view ${i + 1}`} className="h-full w-full object-contain" />
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

### `secure-main/components/product/SizePills.tsx`

Pill-based size selector.

```tsx
// components/product/SizePills.tsx
"use client";

import type { Variant } from "@/lib/types";

type Props = {
  variants: Variant[];
  selected: string | null;
  onSelect: (variantId: string) => void;
  selectedColor: string;
};

export function SizePills({ variants, selected, onSelect, selectedColor }: Props) {
  // Group by size for the active colour.
  const forColor = variants.filter((v) => v.color === selectedColor);

  return (
    <div className="flex flex-wrap gap-2">
      {forColor.map((v) => (
        <button
          key={v.id}
          onClick={() => v.stock > 0 && onSelect(v.id)}
          disabled={v.stock <= 0}
          className={`size-pill ${selected === v.id ? "selected" : ""} ${
            v.stock <= 0 ? "out-of-stock" : ""
          }`}
        >
          {v.size}
        </button>
      ))}
    </div>
  );
}
```

### `secure-main/components/product/BuyPanel.tsx`

Right-side buy panel: colour picker, pill size selector, prominent **Add to Bag** + **Buy Now** buttons.
Actions are stubbed here and wired to server actions in Phase 3.

```tsx
// components/product/BuyPanel.tsx
"use client";

import { useState, useMemo } from "react";
import { SizePills } from "./SizePills";
import type { ProductDetail, Variant } from "@/lib/types";

function formatPHP(n: number) {
  return `₱${n.toLocaleString("en-PH", { minimumFractionDigits: 2 })}`;
}

export function BuyPanel({ product, variants }: { product: ProductDetail; variants: Variant[] }) {
  const colors = useMemo(() => [...new Set(variants.map((v) => v.color))], [variants]);
  const [selectedColor, setSelectedColor] = useState(colors[0] ?? "");
  const [selectedVariant, setSelectedVariant] = useState<string | null>(null);

  const chosen = variants.find((v) => v.id === selectedVariant);

  function handleAddToBag() {
    if (!selectedVariant) return;
    // Phase 3: call addToCart server action
    alert(`Added to bag: ${product.name}, ${chosen?.size}, ${chosen?.color}`);
  }

  function handleBuyNow() {
    if (!selectedVariant) return;
    // Phase 3: call buyNow server action → redirect to checkout
    alert(`Buy now: ${product.name}, ${chosen?.size}, ${chosen?.color}`);
  }

  return (
    <div className="flex flex-col gap-5">
      {/* Category + name */}
      <div>
        <p className="text-xs font-semibold uppercase tracking-wider text-muted mb-1">
          {product.category_name}
        </p>
        <h1 className="heading-display text-2xl sm:text-3xl">{product.name}</h1>
      </div>

      {/* Price */}
      <p className="text-xl font-bold">{formatPHP(product.price_php)}</p>

      {/* Colour picker */}
      <div>
        <p className="text-sm font-semibold mb-2">
          Colour: <span className="font-normal text-muted">{selectedColor}</span>
        </p>
        <div className="flex gap-2">
          {colors.map((c) => (
            <button
              key={c}
              onClick={() => { setSelectedColor(c); setSelectedVariant(null); }}
              className={`px-3 py-1.5 rounded border text-sm transition-colors ${
                selectedColor === c
                  ? "border-primary bg-primary text-surface"
                  : "border-border hover:border-primary"
              }`}
            >
              {c}
            </button>
          ))}
        </div>
      </div>

      {/* Size pills */}
      <div>
        <p className="text-sm font-semibold mb-2">Size</p>
        <SizePills
          variants={variants}
          selected={selectedVariant}
          onSelect={setSelectedVariant}
          selectedColor={selectedColor}
        />
      </div>

      {/* Stock status */}
      {chosen && (
        <p className="text-sm text-muted">
          {chosen.stock > 5 ? "In stock" : chosen.stock > 0 ? `Only ${chosen.stock} left` : "Out of stock"}
        </p>
      )}

      {/* Buttons */}
      <div className="flex flex-col gap-3 pt-2">
        <button
          onClick={handleAddToBag}
          disabled={!selectedVariant || (chosen && chosen.stock <= 0)}
          className="w-full bg-primary text-surface font-semibold uppercase tracking-wider text-sm py-3.5 rounded hover:bg-primary-light disabled:opacity-40 disabled:cursor-not-allowed transition-colors"
        >
          Add to Bag
        </button>
        <button
          onClick={handleBuyNow}
          disabled={!selectedVariant || (chosen && chosen.stock <= 0)}
          className="w-full bg-accent text-surface font-semibold uppercase tracking-wider text-sm py-3.5 rounded hover:bg-accent-hover disabled:opacity-40 disabled:cursor-not-allowed transition-colors"
        >
          Buy Now
        </button>
      </div>
    </div>
  );
}
```

### `secure-main/components/product/DetailAccordion.tsx`

Expandable sections below the buy panel: Description, Shipping, Returns.

```tsx
// components/product/DetailAccordion.tsx
"use client";

import { useState } from "react";

type Section = { title: string; content: string };

const sections: Section[] = [
  { title: "Shipping", content: "Cash on Delivery (COD) only. Metro Manila 2–3 days; provincial 5–7 days. Free shipping on orders over ₱2,500." },
  { title: "Returns", content: "Exchanges within 7 days of delivery. Items must be unworn and in original packaging. Contact us via the account page." },
];

export function DetailAccordion({ description }: { description: string }) {
  const allSections: Section[] = [{ title: "Description", content: description }, ...sections];
  const [openIndex, setOpenIndex] = useState<number | null>(0);

  return (
    <div className="mt-12 border-t border-border divide-y divide-border max-w-2xl">
      {allSections.map((s, i) => (
        <div key={s.title}>
          <button
            onClick={() => setOpenIndex(openIndex === i ? null : i)}
            className="flex w-full items-center justify-between py-4 text-sm font-semibold uppercase tracking-wider"
          >
            {s.title}
            <svg
              width="16" height="16" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"
              className={`transition-transform ${openIndex === i ? "rotate-180" : ""}`}
            >
              <path d="m6 9 6 6 6-6" />
            </svg>
          </button>
          {openIndex === i && (
            <div className="pb-4 text-sm text-muted leading-relaxed">
              {/* Secure: rendered as text (React escaping). No dangerouslySetInnerHTML. */}
              {s.content}
            </div>
          )}
        </div>
      ))}
    </div>
  );
}
```

### `secure-main/components/product/Reviews.tsx`

Product reviews list. In `secure-main/`, review bodies are rendered as **safe React text**. The stored-XSS
lesson in the lab renders raw HTML instead (Phase 5).

```tsx
// components/product/Reviews.tsx  (secure-main)
"use client";

import { useState } from "react";
import type { Review } from "@/lib/types";

function Stars({ rating }: { rating: number }) {
  return (
    <span className="text-warning" aria-label={`${rating} out of 5 stars`}>
      {"★".repeat(rating)}{"☆".repeat(5 - rating)}
    </span>
  );
}

export function Reviews({ reviews, productId }: { reviews: Review[]; productId: string }) {
  const [body, setBody] = useState("");
  const [rating, setRating] = useState(5);

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    // Phase 3: call submitReview server action
    alert(`Review submitted for product ${productId}: ${rating}★ — ${body}`);
    setBody("");
  }

  return (
    <section className="mt-12 max-w-2xl">
      <h2 className="heading-display text-xl mb-6">Reviews ({reviews.length})</h2>

      <div className="space-y-6 mb-8">
        {reviews.map((r) => (
          <div key={r.id} className="border-b border-border pb-4">
            <div className="flex items-center gap-2 mb-1">
              <Stars rating={r.rating} />
              <span className="text-sm font-semibold">{r.author_name}</span>
            </div>
            {/* Secure: body is rendered as plain text. React escapes any HTML. */}
            <p className="text-sm text-muted">{r.body}</p>
          </div>
        ))}
      </div>

      {/* New review form */}
      <form onSubmit={handleSubmit} className="space-y-3 border-t border-border pt-6">
        <p className="text-sm font-semibold">Leave a review</p>
        <div className="flex items-center gap-2">
          <label className="text-sm text-muted">Rating:</label>
          <select
            value={rating}
            onChange={(e) => setRating(Number(e.target.value))}
            className="border border-border rounded px-2 py-1 text-sm"
          >
            {[5, 4, 3, 2, 1].map((n) => (
              <option key={n} value={n}>{n} star{n > 1 ? "s" : ""}</option>
            ))}
          </select>
        </div>
        <textarea
          value={body}
          onChange={(e) => setBody(e.target.value)}
          maxLength={2000}
          rows={3}
          placeholder="What did you think?"
          className="w-full border border-border rounded px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-accent/30 resize-none"
        />
        <button
          type="submit"
          className="bg-primary text-surface font-semibold text-sm px-6 py-2 rounded hover:bg-primary-light transition-colors"
        >
          Submit Review
        </button>
      </form>
    </section>
  );
}
```

---

## 8. Cart drawer

### `secure-main/components/cart/CartDrawer.tsx`

A right-side slide-out drawer. Cart state management is wired in Phase 3; this is the complete UI.

```tsx
// components/cart/CartDrawer.tsx
"use client";

import { useEffect } from "react";
import { CartLine } from "./CartLine";
import type { CartItem } from "@/lib/types";

function formatPHP(n: number) {
  return `₱${n.toLocaleString("en-PH", { minimumFractionDigits: 2 })}`;
}

type Props = {
  open: boolean;
  onClose: () => void;
  items: CartItem[];
  onUpdateQty: (variantId: string, qty: number) => void;
  onRemove: (variantId: string) => void;
};

export function CartDrawer({ open, onClose, items, onUpdateQty, onRemove }: Props) {
  useEffect(() => {
    function handleKey(e: KeyboardEvent) {
      if (e.key === "Escape") onClose();
    }
    if (open) {
      document.addEventListener("keydown", handleKey);
      document.body.style.overflow = "hidden";
    }
    return () => {
      document.removeEventListener("keydown", handleKey);
      document.body.style.overflow = "";
    };
  }, [open, onClose]);

  const subtotal = items.reduce((s, i) => s + i.unit_price_php * i.quantity, 0);

  return (
    <>
      {/* Backdrop */}
      {open && (
        <div className="fixed inset-0 z-50 bg-black/40 backdrop-blur-sm" onClick={onClose} aria-hidden />
      )}

      {/* Panel */}
      <div
        className={`fixed inset-y-0 right-0 z-50 w-full max-w-md bg-surface shadow-xl flex flex-col transition-transform duration-300 ${
          open ? "translate-x-0" : "translate-x-full"
        }`}
        style={{ transitionTimingFunction: "var(--ease-out-expo)" }}
      >
        {/* Header */}
        <div className="flex h-14 items-center justify-between px-4 border-b border-border">
          <span className="heading-display text-sm tracking-widest">YOUR BAG ({items.length})</span>
          <button onClick={onClose} aria-label="Close cart" className="p-1 hover:text-accent">
            <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24">
              <path d="M18 6 6 18M6 6l12 12" />
            </svg>
          </button>
        </div>

        {/* Items */}
        <div className="flex-1 overflow-y-auto p-4 space-y-4">
          {items.length === 0 ? (
            <p className="text-center text-muted py-12">Your bag is empty.</p>
          ) : (
            items.map((item) => (
              <CartLine
                key={item.variant_id}
                item={item}
                onUpdateQty={(qty) => onUpdateQty(item.variant_id, qty)}
                onRemove={() => onRemove(item.variant_id)}
              />
            ))
          )}
        </div>

        {/* Footer */}
        {items.length > 0 && (
          <div className="border-t border-border p-4 space-y-3">
            <div className="flex justify-between text-sm font-semibold">
              <span>Subtotal</span>
              <span>{formatPHP(subtotal)}</span>
            </div>
            <p className="text-xs text-muted">Shipping calculated at checkout. Cash on Delivery only.</p>
            <a
              href="/checkout"
              className="block w-full bg-accent text-surface text-center font-semibold uppercase tracking-wider text-sm py-3.5 rounded hover:bg-accent-hover transition-colors"
            >
              Checkout
            </a>
          </div>
        )}
      </div>
    </>
  );
}
```

### `secure-main/components/cart/CartLine.tsx`

One line item in the cart drawer.

```tsx
// components/cart/CartLine.tsx
import type { CartItem } from "@/lib/types";

function formatPHP(n: number) {
  return `₱${n.toLocaleString("en-PH", { minimumFractionDigits: 2 })}`;
}

type Props = {
  item: CartItem;
  onUpdateQty: (qty: number) => void;
  onRemove: () => void;
};

export function CartLine({ item, onUpdateQty, onRemove }: Props) {
  return (
    <div className="flex gap-3">
      <div className="w-20 h-20 bg-surface-alt rounded overflow-hidden flex-shrink-0 flex items-center justify-center">
        {/* eslint-disable-next-line @next/next/no-img-element */}
        <img src={item.image_path || ""} alt={item.product_name} className="h-16 w-16 object-contain" />
      </div>

      <div className="flex-1 min-w-0">
        <p className="text-sm font-semibold truncate">{item.product_name}</p>
        <p className="text-xs text-muted">{item.size} / {item.color}</p>
        <p className="text-sm font-semibold mt-1">{formatPHP(item.unit_price_php)}</p>

        <div className="flex items-center gap-2 mt-2">
          <button
            onClick={() => onUpdateQty(Math.max(1, item.quantity - 1))}
            className="w-7 h-7 border border-border rounded text-sm hover:bg-surface-alt"
          >
            −
          </button>
          <span className="text-sm font-medium w-6 text-center">{item.quantity}</span>
          <button
            onClick={() => onUpdateQty(item.quantity + 1)}
            className="w-7 h-7 border border-border rounded text-sm hover:bg-surface-alt"
          >
            +
          </button>
          <button onClick={onRemove} className="ml-auto text-xs text-muted hover:text-error">
            Remove
          </button>
        </div>
      </div>
    </div>
  );
}
```

---

## 9. COD checkout

### `secure-main/app/(shop)/checkout/page.tsx`

```tsx
// app/(shop)/checkout/page.tsx
import { CodCheckoutForm } from "@/components/checkout/CodCheckoutForm";

export default function CheckoutPage() {
  return (
    <div className="container-content py-8 max-w-2xl">
      <h1 className="heading-display text-2xl sm:text-3xl mb-2">Checkout</h1>
      <p className="text-sm text-muted mb-8">Cash on Delivery only — no card details needed.</p>
      <CodCheckoutForm />
    </div>
  );
}
```

### `secure-main/components/checkout/CodCheckoutForm.tsx`

Delivery-details form. No payment fields — COD only. Submission wired in Phase 3.

```tsx
// components/checkout/CodCheckoutForm.tsx
"use client";

import { useState } from "react";

export function CodCheckoutForm() {
  const [form, setForm] = useState({
    name: "", phone: "", email: "",
    address: "", city: "", province: "", postal: "",
    notes: "",
  });

  function handleChange(e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) {
    setForm({ ...form, [e.target.name]: e.target.value });
  }

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    // Phase 3: call placeOrder server action
    alert("Order placed (placeholder). Redirecting to success page.");
    window.location.href = "/checkout/success?order=DRP-000001";
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-5">
      <fieldset className="space-y-4">
        <legend className="text-sm font-semibold uppercase tracking-wider mb-2">Contact</legend>
        <input
          name="name" required value={form.name} onChange={handleChange}
          placeholder="Full name" className="w-full border border-border rounded px-3 py-2 text-sm"
        />
        <div className="grid grid-cols-2 gap-3">
          <input
            name="phone" required value={form.phone} onChange={handleChange}
            placeholder="Phone (e.g. 0917-xxx-xxxx)" className="border border-border rounded px-3 py-2 text-sm"
          />
          <input
            name="email" type="email" value={form.email} onChange={handleChange}
            placeholder="Email (optional)" className="border border-border rounded px-3 py-2 text-sm"
          />
        </div>
      </fieldset>

      <fieldset className="space-y-4">
        <legend className="text-sm font-semibold uppercase tracking-wider mb-2">Delivery Address</legend>
        <input
          name="address" required value={form.address} onChange={handleChange}
          placeholder="Street address, bldg, floor, unit" className="w-full border border-border rounded px-3 py-2 text-sm"
        />
        <div className="grid grid-cols-2 gap-3">
          <input
            name="city" required value={form.city} onChange={handleChange}
            placeholder="City / Municipality" className="border border-border rounded px-3 py-2 text-sm"
          />
          <input
            name="province" required value={form.province} onChange={handleChange}
            placeholder="Province" className="border border-border rounded px-3 py-2 text-sm"
          />
        </div>
        <input
          name="postal" value={form.postal} onChange={handleChange}
          placeholder="Postal code (optional)" className="w-full border border-border rounded px-3 py-2 text-sm"
        />
      </fieldset>

      <textarea
        name="notes" value={form.notes} onChange={handleChange}
        placeholder="Order notes (optional)" rows={2}
        className="w-full border border-border rounded px-3 py-2 text-sm resize-none"
      />

      <div className="border-t border-border pt-4 space-y-2">
        <p className="text-sm text-muted">
          Payment method: <strong>Cash on Delivery (COD)</strong>
        </p>
        <button
          type="submit"
          className="w-full bg-accent text-surface font-semibold uppercase tracking-wider text-sm py-3.5 rounded hover:bg-accent-hover transition-colors"
        >
          Place Order (COD)
        </button>
      </div>
    </form>
  );
}
```

### `secure-main/app/(shop)/checkout/success/page.tsx`

```tsx
// app/(shop)/checkout/success/page.tsx
import Link from "next/link";

export default async function SuccessPage({
  searchParams,
}: {
  searchParams: Promise<{ order?: string }>;
}) {
  const params = await searchParams;
  const orderNumber = params.order ?? "—";

  return (
    <div className="container-content py-16 text-center max-w-lg">
      <div className="text-4xl mb-4">✓</div>
      <h1 className="heading-display text-2xl mb-2">Order Placed</h1>
      <p className="text-muted mb-1">Order number:</p>
      <p className="text-xl font-bold mb-6">{orderNumber}</p>
      <p className="text-sm text-muted mb-8">
        Cash on Delivery — please prepare the exact amount. We&rsquo;ll text you when it ships.
      </p>
      <Link
        href="/"
        className="inline-block bg-primary text-surface font-semibold uppercase tracking-wider text-sm px-8 py-3 rounded hover:bg-primary-light transition-colors"
      >
        Continue Shopping
      </Link>
    </div>
  );
}
```

---

## 10. Auth

### `secure-main/app/(auth)/login/page.tsx`

```tsx
// app/(auth)/login/page.tsx
import { LoginForm } from "@/components/auth/LoginForm";
import { GuestEntry } from "@/components/auth/GuestEntry";
import Link from "next/link";

export default function LoginPage() {
  return (
    <div className="container-content py-12 max-w-sm">
      <h1 className="heading-display text-2xl mb-6 text-center">Sign In</h1>
      <LoginForm />
      <div className="flex items-center gap-3 my-6">
        <hr className="flex-1 border-border" />
        <span className="text-xs text-muted uppercase">or</span>
        <hr className="flex-1 border-border" />
      </div>
      <GuestEntry />
      <p className="text-center text-sm text-muted mt-6">
        No account? <Link href="/signup" className="font-semibold text-primary hover:text-accent">Create one</Link>
      </p>
    </div>
  );
}
```

### `secure-main/app/(auth)/signup/page.tsx`

```tsx
// app/(auth)/signup/page.tsx
import { SignupForm } from "@/components/auth/SignupForm";
import Link from "next/link";

export default function SignupPage() {
  return (
    <div className="container-content py-12 max-w-sm">
      <h1 className="heading-display text-2xl mb-6 text-center">Create Account</h1>
      <SignupForm />
      <p className="text-center text-sm text-muted mt-6">
        Already have an account? <Link href="/login" className="font-semibold text-primary hover:text-accent">Sign in</Link>
      </p>
    </div>
  );
}
```

### `secure-main/components/auth/LoginForm.tsx`

```tsx
// components/auth/LoginForm.tsx
"use client";

import { useState } from "react";

export function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);
    // Phase 3: call signIn server action
    // secure-main returns GENERIC errors: "Invalid email or password" — never
    // "no such user" vs. "wrong password" (that distinction is a brute-force hint).
    try {
      alert(`Sign-in placeholder: ${email}`);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      {error && <p className="text-sm text-error bg-error/10 rounded px-3 py-2">{error}</p>}
      <input
        type="email" required value={email} onChange={(e) => setEmail(e.target.value)}
        placeholder="Email" autoComplete="email"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <input
        type="password" required value={password} onChange={(e) => setPassword(e.target.value)}
        placeholder="Password" autoComplete="current-password"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <button
        type="submit" disabled={loading}
        className="w-full bg-primary text-surface font-semibold uppercase tracking-wider text-sm py-3 rounded hover:bg-primary-light disabled:opacity-50 transition-colors"
      >
        {loading ? "Signing in…" : "Sign In"}
      </button>
    </form>
  );
}
```

### `secure-main/components/auth/SignupForm.tsx`

```tsx
// components/auth/SignupForm.tsx
"use client";

import { useState } from "react";

export function SignupForm() {
  const [form, setForm] = useState({ displayName: "", email: "", phone: "", password: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setForm({ ...form, [e.target.name]: e.target.value });
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);
    // Phase 3: call signUp server action
    try {
      alert(`Sign-up placeholder: ${form.email}`);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      {error && <p className="text-sm text-error bg-error/10 rounded px-3 py-2">{error}</p>}
      <input
        name="displayName" required value={form.displayName} onChange={handleChange}
        placeholder="Display name" autoComplete="name"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <input
        name="email" type="email" required value={form.email} onChange={handleChange}
        placeholder="Email" autoComplete="email"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <input
        name="phone" value={form.phone} onChange={handleChange}
        placeholder="Phone (optional)" autoComplete="tel"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <input
        name="password" type="password" required minLength={8} value={form.password} onChange={handleChange}
        placeholder="Password (min 8 characters)" autoComplete="new-password"
        className="w-full border border-border rounded px-3 py-2 text-sm"
      />
      <button
        type="submit" disabled={loading}
        className="w-full bg-primary text-surface font-semibold uppercase tracking-wider text-sm py-3 rounded hover:bg-primary-light disabled:opacity-50 transition-colors"
      >
        {loading ? "Creating…" : "Create Account"}
      </button>
    </form>
  );
}
```

### `secure-main/components/auth/GuestEntry.tsx`

Guest-mode entry — continue to checkout without an account.

```tsx
// components/auth/GuestEntry.tsx
import Link from "next/link";

export function GuestEntry() {
  return (
    <div className="text-center">
      <p className="text-sm text-muted mb-3">Just want to check out? No account needed.</p>
      <Link
        href="/checkout"
        className="inline-block w-full border-2 border-primary text-primary font-semibold uppercase tracking-wider text-sm py-2.5 rounded hover:bg-primary hover:text-surface transition-colors text-center"
      >
        Continue as Guest
      </Link>
    </div>
  );
}
```

---

## 11. Account pages

### `secure-main/app/account/page.tsx`

```tsx
// app/account/page.tsx
import Link from "next/link";

// Phase 3: fetch the signed-in user's profile + recent orders from Supabase.
// Redirect to /login if not signed in.

export default function AccountPage() {
  // Placeholder until wired in Phase 3.
  const user = { display_name: "Lab Shopper", email: "shopper@example.test", phone: "0917-000-0002" };
  const recentOrders = [
    { order_number: "DRP-000001", status: "pending", total_php: 749, created_at: "2026-10-10" },
  ];

  return (
    <div className="container-content py-8 max-w-2xl">
      <h1 className="heading-display text-2xl mb-6">My Account</h1>

      <section className="mb-8">
        <h2 className="text-sm font-semibold uppercase tracking-wider text-muted mb-3">Profile</h2>
        <div className="bg-surface-alt rounded p-4 text-sm space-y-1">
          <p><strong>Name:</strong> {user.display_name}</p>
          <p><strong>Email:</strong> {user.email}</p>
          <p><strong>Phone:</strong> {user.phone}</p>
        </div>
      </section>

      <section className="mb-8">
        <h2 className="text-sm font-semibold uppercase tracking-wider text-muted mb-3">Recent Orders</h2>
        {recentOrders.length === 0 ? (
          <p className="text-sm text-muted">No orders yet.</p>
        ) : (
          <div className="border border-border rounded divide-y divide-border">
            {recentOrders.map((o) => (
              <Link
                key={o.order_number}
                href={`/account/orders/${o.order_number}`}
                className="flex items-center justify-between p-4 hover:bg-surface-alt transition-colors"
              >
                <div>
                  <p className="text-sm font-semibold">{o.order_number}</p>
                  <p className="text-xs text-muted">{o.created_at}</p>
                </div>
                <div className="text-right">
                  <p className="text-sm font-semibold">₱{o.total_php.toLocaleString("en-PH", { minimumFractionDigits: 2 })}</p>
                  <p className="text-xs text-muted capitalize">{o.status}</p>
                </div>
              </Link>
            ))}
          </div>
        )}
      </section>

      <button className="text-sm text-muted hover:text-error transition-colors">Sign Out</button>
    </div>
  );
}
```

### `secure-main/app/account/orders/[id]/page.tsx`

In `secure-main/`, this checks server-side that the signed-in user owns the order — no IDOR. The lab removes
this check (Phase 5).

```tsx
// app/account/orders/[id]/page.tsx  (secure-main)
import Link from "next/link";
import { notFound } from "next/navigation";

export default async function OrderDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  // Phase 3: fetch order from Supabase WITH ownership check:
  //   const order = await supabase.from('orders')
  //     .select('*, order_items(*)')
  //     .eq('order_number', id)
  //     .eq('user_id', session.user.id)   // ← ownership check
  //     .single();
  //   if (!order.data) notFound();

  // Placeholder until wired:
  const order = {
    order_number: id,
    status: "pending",
    total_php: 749,
    created_at: "2026-10-10",
    items: [{ product_name: "Alon Oversized Tee", size: "M", color: "Bone", quantity: 1, unit_price_php: 749 }],
    contact_name: "Lab Shopper",
    shipping_address: "123 Example St, Quezon City",
    shipping_city: "Quezon City",
    shipping_province: "Metro Manila",
  };

  return (
    <div className="container-content py-8 max-w-2xl">
      <Link href="/account" className="text-sm text-muted hover:text-accent mb-4 inline-block">
        ← Back to account
      </Link>
      <h1 className="heading-display text-2xl mb-6">Order {order.order_number}</h1>

      <div className="grid gap-6 md:grid-cols-2">
        <section className="bg-surface-alt rounded p-4">
          <h2 className="text-xs font-semibold uppercase tracking-wider text-muted mb-2">Status</h2>
          <p className="text-sm font-semibold capitalize">{order.status}</p>
          <p className="text-xs text-muted">{order.created_at}</p>
        </section>

        <section className="bg-surface-alt rounded p-4">
          <h2 className="text-xs font-semibold uppercase tracking-wider text-muted mb-2">Shipping</h2>
          <p className="text-sm">{order.contact_name}</p>
          <p className="text-sm text-muted">{order.shipping_address}</p>
          <p className="text-sm text-muted">{order.shipping_city}, {order.shipping_province}</p>
        </section>
      </div>

      <section className="mt-6">
        <h2 className="text-xs font-semibold uppercase tracking-wider text-muted mb-3">Items</h2>
        <div className="border border-border rounded divide-y divide-border">
          {order.items.map((item, i) => (
            <div key={i} className="flex justify-between p-3 text-sm">
              <div>
                <p className="font-semibold">{item.product_name}</p>
                <p className="text-xs text-muted">{item.size} / {item.color} × {item.quantity}</p>
              </div>
              <p className="font-semibold">₱{(item.unit_price_php * item.quantity).toLocaleString("en-PH", { minimumFractionDigits: 2 })}</p>
            </div>
          ))}
        </div>
        <div className="flex justify-between p-3 font-bold text-sm">
          <span>Total (COD)</span>
          <span>₱{order.total_php.toLocaleString("en-PH", { minimumFractionDigits: 2 })}</span>
        </div>
      </section>
    </div>
  );
}
```

---

## 12. Admin dashboard

### `secure-main/app/admin/layout.tsx`

The secure admin layout checks for the `admin` role server-side. The lab removes this check (Phase 5).

```tsx
// app/admin/layout.tsx  (secure-main)
// Phase 3: import the server Supabase client and check `users.role = 'admin'`.
// If not admin → redirect to /login. For now this is a layout shell.

export default function AdminLayout({ children }: { children: React.ReactNode }) {
  // Phase 3: const session = await getSession(); const user = await getProfile(session);
  // if (user.role !== 'admin') redirect('/login');

  return (
    <div className="min-h-screen bg-surface-alt">
      <div className="bg-primary text-surface">
        <div className="container-content flex h-12 items-center justify-between">
          <span className="heading-display text-sm tracking-widest">ADMIN</span>
          <nav className="flex gap-4 text-xs font-medium uppercase tracking-wider">
            <a href="/admin" className="hover:text-accent transition-colors">Dashboard</a>
            <a href="/admin/products" className="hover:text-accent transition-colors">Products</a>
            <a href="/admin/categories" className="hover:text-accent transition-colors">Categories</a>
            <a href="/admin/orders" className="hover:text-accent transition-colors">Orders</a>
            <a href="/admin/users" className="hover:text-accent transition-colors">Users</a>
          </nav>
        </div>
      </div>
      <div className="container-content py-6">{children}</div>
    </div>
  );
}
```

### `secure-main/app/admin/page.tsx`

```tsx
// app/admin/page.tsx
export default function AdminDashboard() {
  // Phase 3: fetch counts from Supabase (total products, orders, users).
  const stats = [
    { label: "Products", value: 13 },
    { label: "Orders", value: 0 },
    { label: "Users", value: 3 },
  ];

  return (
    <>
      <h1 className="heading-display text-2xl mb-6">Dashboard</h1>
      <div className="grid grid-cols-2 md:grid-cols-3 gap-4">
        {stats.map((s) => (
          <div key={s.label} className="bg-surface rounded border border-border p-5">
            <p className="text-xs font-semibold uppercase tracking-wider text-muted mb-1">{s.label}</p>
            <p className="text-3xl font-bold">{s.value}</p>
          </div>
        ))}
      </div>
    </>
  );
}
```

### `secure-main/components/admin/AdminShell.tsx`

A reusable admin page wrapper (title + optional action button).

```tsx
// components/admin/AdminShell.tsx
type Props = {
  title: string;
  action?: React.ReactNode;
  children: React.ReactNode;
};

export function AdminShell({ title, action, children }: Props) {
  return (
    <>
      <div className="flex items-center justify-between mb-6">
        <h1 className="heading-display text-2xl">{title}</h1>
        {action}
      </div>
      {children}
    </>
  );
}
```

### `secure-main/components/admin/DataTable.tsx`

Generic sortable table for admin listings.

```tsx
// components/admin/DataTable.tsx
type Column<T> = {
  key: string;
  header: string;
  render: (row: T) => React.ReactNode;
};

type Props<T> = {
  columns: Column<T>[];
  rows: T[];
  getKey: (row: T) => string;
};

export function DataTable<T>({ columns, rows, getKey }: Props<T>) {
  return (
    <div className="overflow-x-auto border border-border rounded">
      <table className="w-full text-sm">
        <thead>
          <tr className="bg-surface-alt text-left">
            {columns.map((c) => (
              <th key={c.key} className="px-3 py-2 font-semibold uppercase tracking-wider text-xs text-muted">
                {c.header}
              </th>
            ))}
          </tr>
        </thead>
        <tbody className="divide-y divide-border">
          {rows.map((row) => (
            <tr key={getKey(row)} className="hover:bg-surface-alt/50">
              {columns.map((c) => (
                <td key={c.key} className="px-3 py-2">
                  {c.render(row)}
                </td>
              ))}
            </tr>
          ))}
          {rows.length === 0 && (
            <tr>
              <td colSpan={columns.length} className="px-3 py-8 text-center text-muted">
                No data.
              </td>
            </tr>
          )}
        </tbody>
      </table>
    </div>
  );
}
```

### `secure-main/components/admin/EntityForm.tsx`

Reusable admin create/edit form shell.

```tsx
// components/admin/EntityForm.tsx
type Props = {
  title: string;
  onSubmit: (e: React.FormEvent) => void;
  children: React.ReactNode;
  submitLabel?: string;
};

export function EntityForm({ title, onSubmit, children, submitLabel = "Save" }: Props) {
  return (
    <form onSubmit={onSubmit} className="bg-surface border border-border rounded p-6 max-w-xl space-y-4">
      <h2 className="text-sm font-semibold uppercase tracking-wider">{title}</h2>
      {children}
      <button
        type="submit"
        className="bg-primary text-surface font-semibold text-sm px-6 py-2 rounded hover:bg-primary-light transition-colors"
      >
        {submitLabel}
      </button>
    </form>
  );
}
```

### Admin CRUD pages (stub shells — data wired in Phase 3)

#### `secure-main/app/admin/products/page.tsx`

```tsx
// app/admin/products/page.tsx
import { AdminShell } from "@/components/admin/AdminShell";
import { DataTable } from "@/components/admin/DataTable";

export default function AdminProducts() {
  // Phase 3: fetch all products from Supabase.
  const products = [
    { id: "1", name: "Alon Oversized Tee", price_php: 749, is_active: true },
  ];

  return (
    <AdminShell
      title="Products"
      action={<a href="/admin/products/new" className="bg-accent text-surface text-sm px-4 py-2 rounded font-semibold hover:bg-accent-hover transition-colors">+ Add</a>}
    >
      <DataTable
        columns={[
          { key: "name", header: "Name", render: (r) => r.name },
          { key: "price", header: "Price", render: (r) => `₱${r.price_php.toLocaleString()}` },
          { key: "active", header: "Active", render: (r) => (r.is_active ? "Yes" : "No") },
          { key: "actions", header: "", render: (r) => <a href={`/admin/products/${r.id}`} className="text-accent text-xs">Edit</a> },
        ]}
        rows={products}
        getKey={(r) => r.id}
      />
    </AdminShell>
  );
}
```

#### `secure-main/app/admin/categories/page.tsx`

```tsx
// app/admin/categories/page.tsx
import { AdminShell } from "@/components/admin/AdminShell";
import { DataTable } from "@/components/admin/DataTable";

export default function AdminCategories() {
  const categories = [
    { id: "1", name: "T-Shirts", slug: "tees", sort_order: 1 },
    { id: "2", name: "Jackets", slug: "jackets", sort_order: 2 },
    { id: "3", name: "Baggy Pants", slug: "pants", sort_order: 3 },
  ];

  return (
    <AdminShell title="Categories">
      <DataTable
        columns={[
          { key: "name", header: "Name", render: (r) => r.name },
          { key: "slug", header: "Slug", render: (r) => r.slug },
          { key: "order", header: "Order", render: (r) => r.sort_order },
        ]}
        rows={categories}
        getKey={(r) => r.id}
      />
    </AdminShell>
  );
}
```

#### `secure-main/app/admin/orders/page.tsx`

```tsx
// app/admin/orders/page.tsx
import { AdminShell } from "@/components/admin/AdminShell";
import { DataTable } from "@/components/admin/DataTable";

export default function AdminOrders() {
  // Phase 3: fetch all orders.
  const orders: { id: string; order_number: string; status: string; total_php: number; created_at: string }[] = [];

  return (
    <AdminShell title="Orders">
      <DataTable
        columns={[
          { key: "number", header: "Order #", render: (r) => r.order_number },
          { key: "status", header: "Status", render: (r) => <span className="capitalize">{r.status}</span> },
          { key: "total", header: "Total", render: (r) => `₱${r.total_php.toLocaleString()}` },
          { key: "date", header: "Date", render: (r) => r.created_at },
        ]}
        rows={orders}
        getKey={(r) => r.id}
      />
    </AdminShell>
  );
}
```

#### `secure-main/app/admin/users/page.tsx`

```tsx
// app/admin/users/page.tsx
import { AdminShell } from "@/components/admin/AdminShell";
import { DataTable } from "@/components/admin/DataTable";

export default function AdminUsers() {
  const users = [
    { id: "u1", display_name: "Lab Admin", email: "admin@example.test", role: "admin" },
    { id: "u2", display_name: "Lab Shopper", email: "shopper@example.test", role: "customer" },
    { id: "u3", display_name: "Lab Shopper 2", email: "shopper2@example.test", role: "customer" },
  ];

  return (
    <AdminShell title="Users">
      <DataTable
        columns={[
          { key: "name", header: "Name", render: (r) => r.display_name },
          { key: "email", header: "Email", render: (r) => r.email },
          { key: "role", header: "Role", render: (r) => <span className="capitalize">{r.role}</span> },
        ]}
        rows={users}
        getKey={(r) => r.id}
      />
    </AdminShell>
  );
}
```

---

## 13. Shared types

### `secure-main/lib/types.ts`

Type definitions referenced by the components above. These match the database schema in
[`01-database.md`](01-database.md).

```ts
// lib/types.ts

export type Product = {
  id: string;
  slug: string;
  name: string;
  description: string;
  price_php: number;
  image_path: string;
  category_name: string; // joined from categories
  is_featured: boolean;
  is_active: boolean;
};

export type ProductDetail = Product; // may gain extra fields later

export type Variant = {
  id: string;
  size: string;
  color: string;
  sku: string;
  stock: number;
};

export type CartItem = {
  variant_id: string;
  product_name: string;
  size: string;
  color: string;
  image_path: string;
  unit_price_php: number;
  quantity: number;
};

export type Review = {
  id: string;
  author_name: string;
  rating: number;
  body: string;
  created_at: string;
};

export type Order = {
  id: string;
  order_number: string;
  status: string;
  payment_method: string;
  total_php: number;
  created_at: string;
};

export type OrderItem = {
  product_name: string;
  size: string;
  color: string;
  unit_price_php: number;
  quantity: number;
  line_total_php: number;
};
```

---

## Component checklist

Every item from the plan's component list, with its location:

| Component | File | Section |
| --- | --- | --- |
| Navbar + site navigation | `components/layout/Navbar.tsx` | [§4](#4-layout-components) |
| Slide-out menu | `components/layout/SlideOutMenu.tsx` | [§4](#4-layout-components) |
| Footer | `components/layout/Footer.tsx` | [§4](#4-layout-components) |
| Landing/home hero | `components/home/Hero.tsx` | [§5](#5-home--landing) |
| Featured grid | `components/home/FeaturedGrid.tsx` | [§5](#5-home--landing) |
| Promo section | `components/home/PromoSection.tsx` | [§5](#5-home--landing) |
| Product card | `components/product/ProductCard.tsx` | [§6](#6-product-listing) |
| Product grid | `components/product/ProductGrid.tsx` | [§6](#6-product-listing) |
| Filters | `components/product/Filters.tsx` | [§6](#6-product-listing) |
| Search bar | `components/product/SearchBar.tsx` | [§6](#6-product-listing) |
| PDP gallery | `components/product/Gallery.tsx` | [§7](#7-product-detail-page) |
| PDP buy panel (Add to Bag + Buy Now) | `components/product/BuyPanel.tsx` | [§7](#7-product-detail-page) |
| Size pills | `components/product/SizePills.tsx` | [§7](#7-product-detail-page) |
| Detail accordion | `components/product/DetailAccordion.tsx` | [§7](#7-product-detail-page) |
| Reviews block | `components/product/Reviews.tsx` | [§7](#7-product-detail-page) |
| Cart drawer | `components/cart/CartDrawer.tsx` | [§8](#8-cart-drawer) |
| Cart line | `components/cart/CartLine.tsx` | [§8](#8-cart-drawer) |
| COD checkout form | `components/checkout/CodCheckoutForm.tsx` | [§9](#9-cod-checkout) |
| Login form | `components/auth/LoginForm.tsx` | [§10](#10-auth) |
| Signup form | `components/auth/SignupForm.tsx` | [§10](#10-auth) |
| Guest-mode entry | `components/auth/GuestEntry.tsx` | [§10](#10-auth) |
| Admin shell | `components/admin/AdminShell.tsx` | [§12](#12-admin-dashboard) |
| Data table | `components/admin/DataTable.tsx` | [§12](#12-admin-dashboard) |
| Entity form | `components/admin/EntityForm.tsx` | [§12](#12-admin-dashboard) |
| Vulnerability banner | `components/lab/LabBanner.tsx` | Phase 5 (lab only) |
