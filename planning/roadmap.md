# Roadmap

Mirrors the [Shopify Build Plan](../../Kiro) in the Kiro vault at `Projects/Sick Rabbit/Documentation/Website/Development/Shopify Build Plan.md`. That doc is the source of truth; this file is the repo-side summary.

Item-level tasks use `TSK-NNN` IDs so commits can reference them (`restyle: header nav (TSK-005)`). Managed by the `task` skill. Phase-level structure (this file's sections, transitions, audit checkpoints) is managed by the `plan` skill.

## Shipped

| Version | What | Date |
|---|---|---|
| — | Dawn forked as base theme | 2026-04-19 |
| — | Repo setup retrofit (CLAUDE.md, docs/, planning/, .claude/skills/) | 2026-04-19 |
| Phase 1 | Setup complete — Shopify CLI, PR review tooling, AI Toolkit, project skills, GitHub ruleset on `main` | 2026-04-20 |
| Phase 2 | Brand tokens extracted — palette mapped onto Dawn's 5 schemes, 6 font roles wired (4 via Shopify `font_picker`, 2 via Google Fonts; Nordica Plus deferred post-launch), full token block in `assets/base.css` | 2026-04-20 |
| Audit | Post-Phase 2 audit — clean. Token layer ready for Phase 3 consumption, no blocking debt, 0 commits behind upstream Dawn. Notes in chat, no issues logged. | 2026-04-20 |
| — | PR review switched from Greptile (MCP) to Margins (GitHub Action). `.greptile/` + MCP removed, `.margins.md` + workflow added, skills/docs retargeted. | 2026-06-05 |

## In Progress

### Phase 3 — Restyle sections (redesign: apparel-type + merch-table)

**Direction pivoted 2026-06-05** — full spec in [`docs/design/2026-06-05-apparel-type-redesign.md`](../docs/design/2026-06-05-apparel-type-redesign.md). Store reorganised by **apparel type** (Tees, Hoodies, Sweatshirts, Hats) with a Killer-Acid-style dense, type-grouped layout. Themed collections retired. Fonts kept; colour pass deferred to the end.

**PR 1 — Chrome** (`restyle/chrome`) — ✅ shipped 2026-04-21 (#6): header, cart drawer, footer, button system.

Remaining build clusters (detail in the spec):
- [x] **Cluster 1 — Catalogue scaffolding** ✅ — 4 automatic type collections (Tees/Hoodies/Sweatshirts/Hats) + 24 placeholder products, all published via Admin API
- [x] **Cluster 2 — Card + grid density** ✅ shipped (#10) — merch-table grid (5-col, monospace labels) + full-width layout
- [ ] **Cluster 3 — Homepage** (in review): hero + New Arrivals row + 4 type-section previews ("view all" each). Also: New Arrivals collection (newest-first) + Product type catalog filter enabled in Search & Discovery
- [ ] **Cluster 4 — Type pages**: densify collection grid + banner
- [ ] **Cluster 5 — Navigation**: SHOP dropdown → types, About, Contact
- [ ] **Later — Colour pass**: retune palette toward the reference mood (deliberately last)

## Next

### Phase 4 — Content + products (parallel with Phase 3)
- [ ] **TSK-011**: Tapstitch integration — app install or CSV sync
- [ ] **TSK-012**: Product creation + tagging conventions
- [ ] ~~**TSK-013**: Rule-based themed collections (Anachronism, Norse Poets Society, …)~~ — **superseded** by apparel-type collections (moved into Phase 3 redesign, 2026-06-05)
- [ ] **TSK-014**: Paste/write homepage, about, policies (copy from old Astro repo)
- [ ] **TSK-015**: Shipping zones, payment gateways, taxes (GBP)

### Audit Checkpoint (pre-launch, mandatory)
Full audit including the pre-launch checklist before Phase 5. Non-negotiable — enforced by the `audit` skill.

### Phase 5 — Launch
- [ ] **TSK-016**: Connect GitHub integration — `main` → live theme
- [ ] **TSK-016a**: Add Shopify GitHub app to the `main` ruleset bypass list (only appears in the picker after TSK-016 installs it). Without this, merchant admin edits will fail to commit.
- [ ] **TSK-017**: Point `sickrabbit.com` DNS at Shopify (DNS-only cutover from Vercel)
- [ ] **TSK-018**: SSL auto-provision
- [ ] **TSK-019**: Password-protect for final QA
- [ ] **TSK-020**: Remove password → live
