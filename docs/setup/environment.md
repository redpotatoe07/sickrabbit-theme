# Environment

Shopify themes don't use a `.env` file for theme code — there are no runtime env vars baked into Liquid. Authentication is handled per-machine by the Shopify CLI.

## Prerequisites

- **Node.js** — 18 LTS or newer
- **Shopify CLI** — `npm install -g @shopify/cli` (theme commands are bundled since CLI 3.59.0)
- **[Shopify Liquid VS Code extension](https://shopify.dev/docs/storefronts/themes/tools/shopify-liquid-vscode)** — live theme-check diagnostics, autocomplete for tags/filters/objects/settings, hover docs, Liquid Prettier formatting. Catches most issues at save-time before they ever reach a commit.
- **Git** — standard
- **Access to the Sick Rabbit store** — owner or staff account on `sick-rabbit-store.myshopify.com`

## Enable the git pre-commit hook (per-machine, do this once)

The repo ships a pre-commit hook at `.githooks/pre-commit` that refuses direct commits to `main` (which would auto-deploy to the live theme). Enable it:

```sh
git config core.hooksPath .githooks
chmod +x .githooks/pre-commit
```

The `chmod +x` is needed on Windows clients with `core.fileMode=false` (common default) and anywhere else the executable bit may not have survived checkout. Without it, git silently skips the hook and direct commits to `main` go through.

Verify: `git checkout main && git commit --allow-empty -m "test"` should fail with a clear error message. Then `git checkout -` to return to wherever you were. See `docs/setup/branching.md` for the full branch-discipline context.

## First-time auth

```sh
shopify theme dev --store=sick-rabbit-store.myshopify.com
```

The first run opens a browser for login and stores credentials in the CLI's config directory (`~/.config/shopify/` on macOS/Linux, `%APPDATA%\shopify\` on Windows). After that the CLI re-auths silently.

To switch accounts: `shopify auth logout` then re-run `shopify theme dev`.

## Claude Code local permissions (per-machine)

`.claude/settings.local.json` holds per-machine bash permissions for Claude Code (which commands it can run without prompting). It's gitignored — every clone starts fresh. If Claude Code prompts you for permission on commands you expect to allow (e.g. `where python`, `uv --version`, etc.), approve them once and they'll be saved locally. Shared team-wide conventions live in `.claude/settings.json` (which IS committed); local overrides live in the `.local.json` file which isn't.

## Margins PR review

PRs are reviewed automatically by [Margins](https://github.com/redpotatoe07/margins), a GitHub Action defined in `.github/workflows/margins-review.yml`. Per-repo review rules live in `.margins.md` at the repo root. There's no local setup — the Action runs in CI on every PR, so nothing needs configuring per machine.

The only secret it needs is `ANTHROPIC_API_KEY`, stored in the repo's GitHub Actions secrets (never in the repo). To set it once:

```sh
gh secret set ANTHROPIC_API_KEY -R redpotatoe07/sickrabbit-theme
```

Paste the key from [console.anthropic.com](https://console.anthropic.com) when prompted. Confirm with `gh secret list -R redpotatoe07/sickrabbit-theme` (shows names only, never values).

## GitHub integration (Phase 5)

When ready to connect the live theme:

1. In Shopify admin → Online Store → Themes → Add theme → "Connect from GitHub"
2. Authorise Shopify's GitHub app on `redpotatoe07/sickrabbit-theme`
3. Pick the `main` branch
4. Merchant edits from admin will auto-commit back to `main`

## Troubleshooting

| Symptom | Fix |
|---|---|
| `shopify theme dev` says "store not found" | Check the `--store` flag spelling, re-auth with `shopify auth logout` |
| Hot reload stops working | Kill the process and restart. CSS changes occasionally need a hard refresh |
| `shopify theme check` complains about schema | Validate JSON in `templates/*.json` and any `{% schema %}` blocks |
| Merchant admin edits don't appear locally | `shopify theme pull --live` to sync them down |
