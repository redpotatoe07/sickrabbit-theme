# Decisions

Architectural and design decisions. What was decided, why, what was rejected.

## 2026-04-19: Fork Dawn, restyle — don't build from scratch or go headless

**Decision:** Use Shopify's Dawn as the base theme, fork it, restyle it. Deploy via Shopify's GitHub integration.

**Why:** Dawn is the reference OS 2.0 theme — performance-tuned, actively maintained by Shopify, free, well-understood. All standard e-commerce plumbing (cart, checkout, PDP, variants, search) comes for free. The only problem left to solve is styling.

**Rejected:**
- Headless / Hydrogen — explicitly moving away from the prior Astro direction. Adds complexity for no benefit at this scale.
- Build from scratch — 4–8 weeks of reinventing cart logic nobody asked for.
- Paid theme — pay to then fight its opinions. Dawn is freer, cleaner, updates from Shopify.

## 2026-04-19: Retrofit the Dev Project Playbook structure onto Dawn (don't overwrite)

**Decision:** Layer `CLAUDE.md`, `docs/`, `planning/`, `.claude/skills/` alongside Dawn's existing `assets/`, `sections/`, `snippets/`, `templates/` etc. Do not rename, delete, or restructure Dawn's files.

**Why:** Dawn is tracked as `upstream`. Every rename or structural change we make becomes a merge conflict forever. Keep changes additive.

**Rejected:** Restructuring Dawn into a "tidier" layout. Not worth the perpetual merge cost.

## 2026-04-19: Invert the Build Order — visual first, functional already done

**Decision:** Skip the playbook's "functional-first, visual later" phase. Go straight to brand-token extraction and section restyling.

**Why:** The Build Order principle assumes you're building features from scratch. Here, Dawn ships a fully-functional storefront on day one. Functional work is maintenance only (upstream merges, occasional bug fixes). The whole point of forking Dawn is that we skip that phase.

**Rejected:** Running functional audits or feature-building passes before visual work. There's nothing to audit — it works.

## 2026-04-19: Products-first, collections-later

**Decision:** Populate the catalogue by creating products and tagging them as we go. Build collections — likely rule-based / automatic — once there's enough product data to organise.

**Why:** Tapstitch is the fulfillment source and the catalogue is being built from scratch. Designing collections against zero products is speculative. Tags are cheap; collections can be recomputed any time.

**Rejected:** Designing collection structure up front.

## 2026-04-20: Use Shopify AI Toolkit instead of benjaminsehl/liquid-skills

**Decision:** Install the official [Shopify AI Toolkit](https://github.com/Shopify/Shopify-AI-Toolkit) as the Liquid / Shopify Agent Skill source, not `benjaminsehl/liquid-skills` as originally planned in Kiro's `CONTEXT.md`.

**Why:** The liquid-skills repo's own README now redirects people to Shopify's official toolkit ("No longer needed — go check out the official Shopify AI Toolkit"). The official toolkit adds doc search, GraphQL/Liquid validation, and store management via the CLI's execute capability, with auto-updates.

**Rejected:** Cloning the deprecated `benjaminsehl/liquid-skills`. Would work for the three skill files, but we'd miss the validator, doc search, and ongoing updates, and we'd own a dead dependency.

**Install:** Inside Claude Code, run `/plugin marketplace add Shopify/shopify-ai-toolkit` then `/plugin install shopify-plugin@shopify-ai-toolkit`.

## 2026-04-20: Fonts — hybrid Shopify font_picker + Google Fonts link

**Decision:** Load Sick Rabbit fonts through a hybrid system based on what Shopify's font library actually carries.

**Shopify `font_picker` (4 fonts)** — auto-subsetted, edge-cached, merchant-editable:
- `type_body_font` → `anonymous_pro_n4` (body, UI)
- `type_display_special_font` → `unifraktur_maguntia_n4` (H1, buttons)
- `type_logo_font` → `unifraktur_cook_n7` (logo, bold only)
- `type_hero_title_font` → `outfit_n4` (hero)

**Google Fonts `<link>` (2 fonts)** — hardcoded in `layout/theme.liquid`:
- `Pirata One` → `--font-heading-family` (H2)
- `Cinzel` → `--font-display-family` (collection names — stand-in for Nordica Plus)

**Why hybrid:** Pirata One and Cinzel are not in Shopify's curated font library. They're both on Google Fonts, so a single `<link>` in `<head>` (with preconnect) gets both with `font-display: swap`. Everything else goes through Shopify's native pipeline for best performance. `type_header_font` picker is kept with Dawn's default `assistant_n4` but its effect on `--font-heading-family` is overridden in `theme.liquid` — the setting is retained so upstream Dawn merges stay clean.

**Nordica Plus gap:** The original brand display font for collection names is not in Shopify's library OR Google Fonts (paid foundry font). Cinzel is the stand-in — medieval-leaning display serif, sits in the same gothic band as Pirata One and UnifrakturCook. Licensing + self-hosting Nordica Plus is a **post-launch decision** — logged in `planning/ideas.md`.

**Initial attempt that failed:** Tried to put all 6 fonts through `font_picker` on the assumption that Shopify's library = full Google Fonts catalogue. It isn't. Shopify's library is curated. Check the [available fonts](https://shopify.dev/docs/storefronts/themes/architecture/settings/fonts) before defaulting any `font_picker` to a non-standard family.

**Rejected:**
- Self-hosting all six fonts via `@font-face` — slower first paint, bigger repo, font files to license/manage.
- Finding in-library substitutes for Pirata One and Cinzel — the brand tokens call these out specifically; substitution is last-resort, not a convenience.
- Licensing Nordica Plus now — blocks Phase 2 on an external purchase; Cinzel is a strong-enough stand-in to move forward.

## 2026-04-20: Dawn colour scheme mapping — Sick Rabbit palette

**Decision:** Map the five Sick Rabbit brand tokens onto Dawn's five built-in colour schemes, keeping Dawn's scheme conventions (cards default to scheme-2, sold-out badges to scheme-3, sale badges to scheme-5).

| Scheme | Role | Background | Text | Button |
|---|---|---|---|---|
| 1 | Sand Dune — primary / default | `#e5dcc5` | `#2d2d2a` | `#903636` burnt rose |
| 2 | Graphite — cards | `#2d2d2a` | `#e5dcc5` | `#903636` burnt rose |
| 3 | Slate — sold-out badge | `#1c4d4f` | `#e5dcc5` | `#903636` burnt rose |
| 4 | Rose — featured / hero accent | `#903636` | `#e5dcc5` | `#2d2d2a` graphite |
| 5 | Copper — sale badge | `#a77b5f` | `#2d2d2a` | `#2d2d2a` graphite |

**Why:**
- Scheme-1 = Sand Dune (light) matches the old Astro site's default page background — most of the storefront renders on this.
- Dark graphite cards (scheme-2) on sand backgrounds give strong contrast — core Sick Rabbit feel.
- Burnt rose is the button colour across light schemes so CTAs pop without per-section overrides.
- Slate → sold-out and Copper → sale leans on Dawn's existing `sale_badge_color_scheme` / `sold_out_badge_color_scheme` conventions so merchants and upstream Dawn merges keep working.
- `--color-faded-copper` was flagged as "reserved / special uses" — sale badges are rare enough to qualify as a special use.

**Rejected:**
- Fewer schemes (e.g. 3) — would break Dawn's admin picker UX and break presets.
- Burnt rose as scheme-1 background — too aggressive for the default surface; reserved for featured moments instead.
- Keeping Copper entirely out of the scheme system — would leave scheme-5 doing nothing useful.

## 2026-04-20: Button system — bevelled primary on burnt rose, flat secondary/tertiary

**Decision:** Define `.button` / `.button--primary` / `.button--secondary` / `.button--tertiary` in `assets/chrome-restyle.css` using the existing Phase 2 tokens. Primary uses `--font-display-special-family` (UnifrakturMaguntia), burnt-rose fill, sand-dune label, and the `--shadow-bevel-*` trio for raised/hover/pressed states — so a CTA reads as a physical analogue toggle rather than a flat web button. Secondary is an outlined monospace button; tertiary is text-only.

**Why:** Phase 3 is the first time `.button` styling actually ships — Dawn's default is neutral and boxy. The Sick Rabbit brand leans on three legs (archaic typography + analogue gear physicality + playful irreverence), and the CTA is where all three converge. The bevel gives the physicality, UnifrakturMaguntia carries the archaic typography onto UI, and burnt rose is the brand-locked accent. Making the choice now, while the cart-drawer checkout button is being styled, avoids writing a one-off checkout button that later has to be reconciled with a general button system.

**Implementation note:** Dawn paints `.button` fills via a `::after` pseudo-element (box-shadow stack, z-index 1) rather than a `background-color` on `.button` itself. The first pass overrode `.button { background-color: ... }` and saw no change at rest — Dawn's `::after` was covering it. The fix extends Dawn's `::after` box-shadow list with the `--shadow-bevel-*` tokens so Sick Rabbit's bevel rides on top of Dawn's scheme-bound fill/border. This also means every `.button` variant gets the bevel automatically (primary, secondary, anything else Dawn classes as `button`), which matches the "physicality on every control" direction — Dawn-demoted secondaries (e.g. PDP "Add to cart" when PayPal is enabled) still feel analogue instead of looking flat next to a primary checkout button.

**Rejected:**
- Extend Dawn's `.button` with a single colour override and leave the neutral shape — misses the physicality leg entirely; all three brand legs are load-bearing.
- Build a standalone `.checkout-button` just for the drawer — cheaper now, but locks in duplication every time a CTA appears elsewhere (PDP add-to-cart, hero CTA, newsletter submit).
- Put the button styles in a separate `buttons.css` file — fragments the Phase 3 override layer and adds another `stylesheet_tag`. `chrome-restyle.css` already covers the surfaces where buttons appear first (header cart, cart drawer, footer newsletter); later phases can split if the file balloons.
- Bevel on primary only, secondary flat — tried first pass, produced visible inconsistency between primary CTAs (cart drawer checkout) and Dawn-demoted secondaries (PDP add-to-cart when dynamic checkout is on). The demotion is about hierarchy, not brand voice — all controls should feel analogue.

## 2026-04-19: Copy source is the old Astro repo

**Decision:** Reuse homepage copy, about page, collection descriptions from `C:\Users\redpo\repos\sickrabbit-website` (`src/pages/`, `src/content/`). Only rewrite when the old copy doesn't fit the new context.

**Why:** The voice is already right. Don't redo the voice work just because the stack changed.

**Rejected:** Writing copy from scratch for the Shopify version.

## 2026-06-05: Switch PR review from Greptile to Margins

**Decision:** Discontinue Greptile and review PRs with [Margins](https://github.com/redpotatoe07/margins) instead. Removed the `.greptile/` directory and the `greptile-api` MCP server in `.mcp.json`; added `.margins.md` (per-repo review rules) at the repo root and `.github/workflows/margins-review.yml` (the review Action). The `pr-commit` / `pr-comments` skills and all docs were retargeted to Margins; their machinery (open-PR, schedule the 8-minute polling cron, fetch-fix-push loop, quiet-after-2-rounds ship signal) is reviewer-agnostic and unchanged.

**Why:** Greptile ran as a local MCP proxy requiring a per-machine API key and a running Claude Code session to poll. Margins runs server-side as a GitHub Action on `pull_request` events — no local key, no MCP, reviews fire automatically on push, and review rules are versioned in-repo via `.margins.md`. Only secret is `ANTHROPIC_API_KEY` in GitHub Actions secrets.

**Rejected:**
- Keeping both — redundant cost and two review voices on every PR.
- Deleting the `pr-comments` / `pr-commit` skills outright — their loop works for any reviewer that posts PR comments; only the Greptile-specific naming needed swapping.

## 2026-06-05: Product card image ratio — portrait, not Dawn's "adapt"

**Decision:** Set `image_ratio` to `portrait` on the collection grid (`templates/collection.json`) and the homepage `featured-collection` sections, overriding Dawn's default of `adapt`.

**Why:** `adapt` sizes each card to its own image's aspect ratio, so a grid of mixed-shape photos renders ragged (uneven tile heights, rows that don't line up). `portrait` forces every tile to a uniform tall rectangle — essential for the merch-table look where dozens of products sit in a tight grid. Apparel photography is portrait-leaning anyway.

**Upstream-merge note:** `image_ratio` is a frequently-churned field — Dawn's "Update from Shopify" merges repeatedly reset it back to `adapt`. **After any `git merge upstream/main`, re-check `image_ratio` in `templates/collection.json` (and the homepage featured sections) and restore `portrait` if upstream flipped it.** Logged here so the choice isn't silently reverted. (Surfaced during PR #10's review — below the comment-posting threshold, recorded here instead.)

**Rejected:**
- `adapt` — ragged grid, defeats the merch-table density.
- `square` — viable, but crops apparel awkwardly; portrait suits tees/hoodies better.
