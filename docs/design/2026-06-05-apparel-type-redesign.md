# Redesign — apparel-type structure + merch-table layout

**Date:** 2026-06-05
**Status:** Approved direction (brainstorm complete). Build sequenced into PR clusters below.
**Reference:** [killeracid.com](https://killeracid.com/) — dense "merch-table" product grid, type-grouped sections, bold hero, minimal chrome.

## What changes

A structural + visual redesign of the storefront. Two things move together:

1. **Structure** — retire the themed/curated collections (Anachronism, Norse Poets Society, Digital Occult, Major Arcana) and organise the store by **apparel type**: Tees, Hoodies, Sweatshirts, Caps.
2. **Look** — adopt Killer Acid's *layout and density* (tight product grid, type-grouped sections, bold hero, minimal card chrome). Keep Sick Rabbit's **fonts** (blackletter display + monospace UI) and **warm-dark palette for now**; a dedicated colour pass comes later.

## Decisions

- **Collections become apparel-type, automatic.** Four automatic (rule-based) collections — Tees, Hoodies, Sweatshirts, Caps — populated by product **type**. No hand-curating. The themed collections are removed from navigation. ("Scrap collections" = scrap the *themed* ones; the collection mechanism stays — it's how Shopify powers a type grid.)
- **Homepage = hybrid long-scroll.** Bold hero, then a **New Arrivals** row, then the four type sections stacked, each a **preview** (one 5-across row — `products_to_show: 5` to match the Cluster 2 grid) under a big blackletter header with a **"View all <type> →"** link to that type's full page. Built from native Dawn `featured-collection` sections (New Arrivals + one per type, `show_view_all` on) + an `image-banner` hero. Footer. *(New Arrivals added during Cluster 3 — a newest-first automatic collection so the freshest stock leads the page.)*
- **Type pages are the real shop layer.** `/collections/tees` etc. exist as full collection-grid pages with Dawn's filtering / sorting / pagination — the "browse + search within a type" surface.
- **Navigation stripped down.** `SHOP` (dropdown → the four types), `About`, `Contact`, plus search / account / cart icons. No themed-collection menu items.
- **Product cards densified.** More columns, tighter gaps, minimal card (image, title, price). Same card on homepage previews and type pages. Restyle `snippets/card-product.liquid` + grid CSS; do **not** fork the snippet.
- **Colour later.** Keep the current warm-dark schemes; the Killer-Acid colour mood is a separate future step.

## Practical constraint — store data vs theme code

Theme markup/CSS lives in this repo and ships via PR. But **collections, products, and the nav menu are store objects** (in `sick-rabbit-store`), not theme files — so the merch-table look can't be *seen* until products exist (Phase 4 hasn't run). To build and validate the design we create a handful of **placeholder products** (a few per type, with the right product type set) so the grids populate. These get swapped for real products later. Creating them needs Shopify admin / Admin API access, not a code change.

## Build sequence (PR clusters)

Mirrors the Phase 3 approach — shared patterns land once, in order.

- **Cluster 1 — Catalogue scaffolding (store data):** create the four automatic collections (by product type); create placeholder products spread across the four types; point the nav menu at the new structure. Enables everything downstream to render.
- **Cluster 2 — Product card + grid density:** restyle `card-product.liquid` and the grid to the merch-table density. Visible on both homepage previews and type pages.
- **Cluster 3 — Homepage:** rebuild `templates/index.json` — hero (`image-banner`) + **New Arrivals** row + four `featured-collection` type sections (preview + view-all) + footer. Also enable the Product type catalog filter (Search & Discovery).
- **Cluster 4 — Type pages:** densify `main-collection-product-grid` / `main-collection-banner` to match.
- **Cluster 5 — Navigation:** `SHOP` dropdown → four types, `About`, `Contact`; remove themed-collection links.
- **Later — Colour pass:** retune the palette toward the reference's mood (separate, deliberately deferred).

## Out of scope (for now)

- Colour overhaul (deferred by decision).
- Real product photography / copy (Phase 4 content).
- PDP redesign beyond what the card/grid changes touch.
