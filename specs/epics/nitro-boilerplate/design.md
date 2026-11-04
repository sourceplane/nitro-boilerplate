# nitro-boilerplate — design

## 1. The starter

vercel/examples `framework-boilerplates/nitro` at `de86723` is a Nitro 2
project of seven files: `nitro.config.ts` (`srcDir: "server"`),
`server/routes/index.ts` (one `defineEventHandler` returning an inline HTML
welcome page: a title, a heading, a line about the platform, two links and
Nitro's logo hot-linked from nitro.build), a `package.json` whose one
dependency is `nitropack: latest` (its lockfile resolved 2.12.4), a
`tsconfig.json` extending Nitro's generated one, a `.gitignore`, an `.npmrc`
and a README. Vercel builds it with zero configuration: it detects Nitro and
builds with Nitro's `vercel` preset.

## 2. How each Vercel piece maps

| Vercel | Here |
|---|---|
| Zero-config detection, Nitro's `vercel` preset | `preset: "cloudflare_module"` in `nitro.config.ts`; `nitro build` writes `.output/server/index.mjs` (the Worker, an ES module) and `.output/public` (static output) |
| Vercel's CDN for Nitro's public assets | Workers Static Assets over `.output/public`, with the `ASSETS` binding the preset's runtime reads for its own asset lookup |
| A function per deployment, configured by Vercel | `apps/api/wrangler.jsonc`, committed: the Worker's name, compatibility date, `nodejs_compat`, the build output as `main` and `assets`, per-environment `ENVIRONMENT` vars |
| `nitropack: latest` | `nitropack: 2.13.4`, pinned |
| The page's Vercel copy and links | Cloudflare's: "running on Cloudflare Workers", the Workers docs and Nitro's Cloudflare provider page |
| Deploy on push | `apps/api/component.yaml` (`cloudflare-worker-turbo`): build, dry run on pull requests; build, deploy and a smoke over `/health` on merge |

## 3. What the spike proved

In a scratch copy, before any of the repository: nitropack 2.13.4 (npm's
`latest` on 2026-11-05) with `preset: "cloudflare_module"`,
`compatibilityDate: "2026-10-01"` and `cloudflare: { deployConfig: false,
nodeCompat: true }` builds in under a second into a 35 KiB (gzipped) Worker.
Under `wrangler dev` over a wrangler config naming that output, `GET /`
answers the template's page (200, `text/html`), `HEAD /` answers 200 with no
body, a `/health` route reading `event.context.cloudflare.env.ENVIRONMENT`
answers that environment's var, and an unknown path answers Nitro's own 404
(`Cannot find any route matching /nope.`). The dry run binds `ASSETS` and
the environment's vars and attaches no module beside the bundle.

Three choices fell out of it:

- **One wrangler config.** With `deployConfig` on, Nitro writes its own
  `wrangler.json` into `.output/server`; Orun's Worker composition checks
  for a committed config before any build and deploys from it. So Nitro's is
  off and the committed one names Nitro's output paths. The lane orders the
  build before the dry run and the deploy, which is what those paths need.
- **`ASSETS` stays.** The template ships no public file, so `.output/public`
  holds only the preset's empty `_headers`; but the preset's runtime serves
  Nitro's public assets through `env.ASSETS`, and a file an owner adds under
  `server/public/` (Nitro scans `<srcDir>/public`) should just work.
- **The pin.** `latest` floats across majors, and Nitro 3 is at release
  candidate. A pinned 2.13.4 makes an upgrade a pull request.

## 4. The repository

express-api's, which is hono-api's: a pnpm workspace (`apps/api`,
`tests/api`, `tooling/*`), turbo, the shared TypeScript and ESLint
configuration, `intent.yaml` (dev verify-only, stage, prod after stage, the
placeholder workspace `ws_CHANGE_ME`), `ci.yml` byte for byte
turso-starter's as hono-api carries it, and the baseline's machinery
(`blueprint.yaml`, `repo-blueprint.yaml`, `.baseline/`, `tooling/rebrand`,
`tooling/bootstrap`, `hooks/`, `testing/`).

Tests build the app with Nitro itself — `createNitro` with the
`node-listener` preset over the app's own config and routes, in a temporary
directory — and serve it on Node, so they check Nitro's routing and its 404
as well as the template's page. The end-to-end tier serves the
`cloudflare_module` build under workerd. Nitro's generated directories
(`.output`, `.nitro`, `.data`) are ignored by git, by the build document's
`ignore:` and by the leak gate.

## 5. Alternatives rejected

- **Nitro's generated `wrangler.json` as the deploy config.** It only exists
  after a build, so the composition's structure check would fail in a fresh
  checkout, and it would be a second config beside the one the deploy reads.
- **Dropping Static Assets.** The Worker would deploy, but a public file an
  owner adds would 404 with no hint why.
- **Nitro 3.** At release candidate on the day of the port; the template is
  Nitro 2, and so is this.
- **Rewriting the route as a plain fetch handler (hono-api's shape).** The
  starter is about Nitro: its file-based routing and its build are the point.
