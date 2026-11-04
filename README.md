# nitro-boilerplate

Vercel's [Nitro](https://github.com/vercel/examples/tree/main/framework-boilerplates/nitro)
starter — a [Nitro](https://nitro.build/) server whose one file-based route,
`server/routes/index.ts`, answers with the template's welcome page — running
as one **Cloudflare Worker**, built by Nitro's own `cloudflare_module` preset
(in place of Vercel's zero-config detection) into the Worker and its
**Workers Static Assets**, deployed by
[Orun](https://github.com/sourceplane/orun) to stage and prod on every merge.
Packaging it as an Orun baseline is the next milestone (NI2).

The template's code is the code. What the port adds is the preset in
`nitro.config.ts`, the Worker's config, the monorepo wrapper, a test package,
the deploy contract and a `/health` route the deploy smoke-tests; what it
swaps is the platform's name and links in the page, and `nitropack: latest`
for a pinned `2.13.4`. See
[`specs/epics/nitro-boilerplate`](specs/epics/nitro-boilerplate/README.md)
for the design, the milestones and the as-built status.

The example is MIT-licensed (vercel/examples); its licence ships with the app
as `apps/api/LICENSE.md`.

## What is here (the product)

| Surface | Where |
|---|---|
| The welcome page, `GET /` — the template's route, named by the site's name | `apps/api/server/routes/index.ts` |
| `GET /health` — the site's name and the environment it runs as | `apps/api/server/routes/health.ts` |
| The site's name (the rename's seam) | `apps/api/server/identity.ts` |
| Nitro's config — the template's `srcDir`, and the `cloudflare_module` preset | `apps/api/nitro.config.ts` |
| Worker config (name, compatibility, the build output as the Worker and its Static Assets, per-environment vars) | `apps/api/wrangler.jsonc` |
| Deploy contract | `apps/api/component.yaml` (`cloudflare-worker-turbo`) |
| Tests | `tests/api` (vitest: builds the app with Nitro's `node-listener` preset, serves it on Node and calls every route over HTTP) |
| The example's licence (MIT) | `apps/api/LICENSE.md` |

## Getting started

```bash
pnpm install          # also runs `nitro prepare` (the types under apps/api/.nitro)
pnpm typecheck
pnpm lint
pnpm test
pnpm build            # nitro build → apps/api/.output (the Worker and its assets)
```

Run it with Nitro's dev server, or the built Worker under workerd:

```bash
pnpm --filter @app/api dev            # nitro dev, http://localhost:3000
pnpm --filter @app/api preview        # wrangler dev over the build, http://localhost:8787
curl http://localhost:8787/           # the welcome page
curl http://localhost:8787/health     # {"status":"ok","name":"Nitro on Workers","environment":"local"}
```

Add a route by adding a file: `apps/api/server/routes/hello.ts` exporting
`defineEventHandler(...)` answers `/hello`; `server/api/` routes are mounted
under `/api`. Static files go in `apps/api/server/public/` (Workers Static
Assets serves them before the Worker runs).

## How it maps to Vercel

| Vercel | Here |
|---|---|
| Vercel detects Nitro and builds with its `vercel` preset | `nitro.config.ts` sets `preset: "cloudflare_module"`: `nitro build` writes the Worker (`.output/server/index.mjs`) and its static output (`.output/public`), which the committed `wrangler.jsonc` deploys |
| Nitro's public assets on Vercel's CDN | Workers Static Assets over `.output/public` (the `ASSETS` binding the preset reads); the template ships none |
| `nitropack: latest` | `nitropack: 2.13.4`, pinned (the lockfile records it): a floating major would let Nitro 3 land in a build unannounced |
| The page's "running on Vercel" and its Vercel docs links | "running on Cloudflare Workers", and links to the Workers docs and Nitro's Cloudflare provider page |
| `vercel dev` · `vercel build` · `vercel deploy` | `nitro dev` (or `wrangler dev` over the build) · `nitro build` · `wrangler deploy --env <stage\|prod>` |
| Deploy on push | Orun deploys `<repo>-api-{stage,prod}` on merge to `main` and smoke-tests `/health` (`apps/api/component.yaml`) |
| "Deploy with Vercel" (clone the template) | `orun new --blueprint repo-blueprint.yaml` — a new repository, branded, deployed |

## CI

| Workflow | What it runs | Ships to an app |
|---|---|---|
| `ci.yml` | the Orun pipeline, and the only workflow an app carries: `orun plan --changed`, then one job per lane. A pull request runs the verify lanes (the Worker built by Nitro, type-checked and dry-deployed per environment; the tests' `quick-check` lane installs and checks the package, it does not run them), except the bootstrap's own landings (`orun/BASE-*`), where `plan` alone is the check; a merge deploys stage → prod. It plans nothing while `intent.yaml` names the placeholder workspace `ws_CHANGE_ME`, as this repository does. A manual run (Orunbase's Deploy button, or Run workflow on `main`) plans the environment and components it names under the push-to-main trigger, so they deploy as a merge deploys them | yes |
