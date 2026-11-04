# nitro-boilerplate — implementation status

| Milestone | State | Landed |
|---|---|---|
| NI0 — the spec | ✅ | NI-1 |
| NI1 — Nitro on Workers | ✅ | NI-2 |
| NI2 — a baseline | planned | NI-3 |

## Departures from the upstream starter

Everything not listed here is the starter as it is upstream at `de86723`.

| # | What | Why |
|---|---|---|
| 1 | The app lives in `apps/api` of a pnpm workspace (`@app/api`); upstream's `package.json`, `.gitignore`, `.npmrc` and README are replaced by the workspace's; its scripts (`build`, `dev`, `prepare`) are kept, `preview` runs `wrangler dev` over the build, and `typecheck`, `lint` and `deploy` are added | The workspace's shape (express-api's) |
| 2 | `nitropack: 2.13.4` (upstream `latest`, locked 2.12.4), with `wrangler` beside it | A pinned Nitro (risk NI-B); the deploy tool |
| 3 | `nitro.config.ts`: `preset: "cloudflare_module"`, `compatibilityDate: "2026-10-01"`, `cloudflare: { deployConfig: false, nodeCompat: true }` beside upstream's `srcDir` | Nitro's Workers preset in place of Vercel's detection; one wrangler config (design §3) |
| 4 | `server/routes/index.ts`: `<title>` and the heading read `SITE_NAME` (`"Nitro on Workers"`; upstream's are "Nitro on Vercel" and "Welcome to Nitro"), HTML-escaped; "running on Vercel" is "running on Cloudflare Workers"; the "▲ Vercel docs" link is "☁ Workers docs" (developers.cloudflare.com/workers/) and Nitro's docs link points at its Cloudflare provider page; the handler takes no unused `event` | One rename seam; the page no longer names a platform it does not run on |
| 5 | `server/routes/health.ts` added: `{"status":"ok","name":SITE_NAME,"environment":…}` from `event.context.cloudflare.env.ENVIRONMENT` (`process.env` on Node) | The deploy's smoke and the bootstrap's probes read it (hono-api's contract) |
| 6 | `server/identity.ts` added: `export const SITE_NAME = "Nitro on Workers"` | The rename's one seam |
| 7 | `tsconfig.json` still extends Nitro's generated config, and adds `strict: true` and excludes `eslint.config.js` (Nitro's includes every file of the package with `allowJs`) | The workspace type-checks strictly; the lint config is eslint's to check |
| 8 | `wrangler.jsonc` (`<repo>-api`, `nodejs_compat`, `.output/server/index.mjs`, Static Assets over `.output/public` with `ASSETS`, `env.{dev,stage,prod}` vars) and `component.yaml` (`cloudflare-worker-turbo`, a smoke over `/health`) | The Worker's config and the deploy contract replace Vercel's (design §2–3) |
| 9 | `apps/api/LICENSE.md` is vercel/examples' MIT licence (upstream's `package.json` names none) | The example ships under the repository's licence; the product carries it |
| 10 | Tests: `tests/api/src/app.test.ts` (vitest): the app built by Nitro with the `node-listener` preset, served on Node, every route called over HTTP | The starter ships none |
