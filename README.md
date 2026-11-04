# nitro-boilerplate

Vercel's [Nitro](https://github.com/vercel/examples/tree/main/framework-boilerplates/nitro)
starter — a [Nitro](https://nitro.build/) server whose one file-based route,
`server/routes/index.ts`, answers with the template's welcome page — running
as one **Cloudflare Worker**, built by Nitro's own `cloudflare_module` preset
(in place of Vercel's zero-config detection) into the Worker and its
**Workers Static Assets**, and packaged as an **Orun baseline**: one command
takes a new app from nothing to live on stage and prod, in its own
repository, under its own name — in ONE landing.

The template's code is the code. What the port adds is the preset in
`nitro.config.ts`, the Worker's config, the monorepo wrapper, a test package,
the deploy contract and a `/health` route the deploy smoke-tests; what it
swaps is the platform's name and links in the page, and `nitropack: latest`
for a pinned `2.13.4`. What the baseline adds is everything needed to make a
copy of it somebody else's — built the way
[cirrus-baseline](https://github.com/sourceplane/cirrus-baseline) is. See
[`specs/epics/nitro-boilerplate`](specs/epics/nitro-boilerplate/README.md)
for the design, the milestones and the as-built status.

The example is MIT-licensed (vercel/examples); its licence ships with the app
as `apps/api/LICENSE.md`.

## Instantiating a product

```bash
git clone --depth 1 --branch <baseline-tag> https://github.com/sourceplane/nitro-boilerplate
cd nitro-boilerplate
orun new --blueprint repo-blueprint.yaml --out ~/apps/acme-api --run-hooks --resume \
  --set reponame=acme-api --set productname="Acme API" \
  --set githuborg=acme-inc --set subdomain=<workers-dev-subdomain>
```

Or, once the platform's registry lists it, from the Orun console (Baselines →
`nitro-boilerplate` → Build). Two phases — `01-product` (the repository, its CI and
the whole app in one pull request, deployed to stage and prod by one
convergence, smoke-tested) and `02-docs` (the live deployment recorded) —
each landing one pull request in the new repository.
**[BOOTSTRAP.md](BOOTSTRAP.md)** is the operator guide;
[docs/phases/](docs/phases/README.md) has a page per phase.

The new repository receives the **app only** — the Worker, its tests, its CI,
its tooling, a README of its own and its operating contract — renamed
throughout to the app's name, repository and Cloudflare Worker. None of this
repository's machinery ships, and nothing that ships names it.

| What makes it a baseline | Where |
|---|---|
| The console card (inputs, secrets, programme) | `blueprint.yaml` |
| The build document (modules, phases, hooks) | `repo-blueprint.yaml` |
| This tree's identity, and the bootstrap's task contracts | `.baseline/` |
| The rename engine | `tooling/rebrand/rebrand.mjs` |
| The bootstrap's helpers (deferrals, redeploy marker, work-tree strip) | `tooling/bootstrap/` |
| The deployment-record renderer | `hooks/render-deployment-docs.sh` |
| The app's README and context pack | `.baseline/product/README.md`, `ai/context/` |
| The baseline's tests: contract, leak, offline bootstrap, end to end | `testing/`, `.github/workflows/baseline.yml` |
| Cutting a `baseline-vN` tag | `.github/workflows/tag.yml` |

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
| `baseline.yml` | the baseline's own tiers: contract tests, the blueprint read by the pinned orun, the leak gate, the offline bootstrap, the app built by Nitro, dry-deployed, served and read route by route | no |
| `tag.yml` | cuts a `baseline-vN` tag (manual) | no |
