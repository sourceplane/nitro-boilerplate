# nitro-boilerplate — decisions

The choices that shape this repository, one line each, with where the full
reasoning lives. Superseded decisions stay listed, marked, so the history
reads in order.

| # | Decision | Why | Source |
|---|---|---|---|
| 1 | The template's Nitro app, its route unchanged but for the platform's name and links, built by Nitro's own `cloudflare_module` preset into a plain Worker (`cloudflare-worker-turbo`) | Nitro ships a Workers preset; no adapter and no rewrite into a fetch handler | `specs/epics/nitro-boilerplate/design.md` |
| 2 | `nitropack` pinned to `2.13.4` (the template asks for `latest`) | A floating dependency would let Nitro 3 (at RC) land in a build unannounced; an upgrade is a deliberate pull request | design.md §3, risk NI-B |
| 3 | `wrangler.jsonc` is committed, and Nitro's own generated config is off (`cloudflare.deployConfig: false`) | One config a deploy reads; it names what `nitro build` writes (`.output/server/index.mjs`, `.output/public`), and the lane builds before it dry-runs or deploys | `apps/api/wrangler.jsonc`, risk NI-C |
| 4 | Workers Static Assets over `.output/public`, with the `ASSETS` binding | How the preset serves Nitro's public assets; the template ships none, so a file added under `server/public/` just works | design.md |
| 5 | `GET /health` as its own route file, and one name seam (`apps/api/server/identity.ts`) the welcome page reads | The deploy smoke and the bootstrap's probes read `/health`; the rename writes the name in one place | IMPLEMENTATION-STATUS |
| 6 | The tests build the app with Nitro itself (`node-listener` preset) and call it over HTTP on Node | The template's behaviour (file-based routing, Nitro's 404) independent of the runtime; the e2e serves the Worker build under workerd | `tests/api/src/app.test.ts` |
| 7 | One landing (`01-product`) then the record (`02-docs`) | A landing has a fixed cost; the plan, not the phases, carries deploy order (cirrus-baseline since baseline-v22) | `repo-blueprint.yaml` |
| 8 | `ci.yml` plans nothing while `intent.yaml` names `ws_CHANGE_ME` | A bootstrap cannot set a repository variable | hono-api's design |
| 9 | stack-tectonic 0.19.0 | The family's one pin (risk NI-F) | risks-and-open-questions.md |
| 10 | `subdomain` is read from the Cloudflare connection, never asked; the console asks for no domain | It belongs to the account; a domain is an owner's step after the bootstrap | `blueprint.yaml` |
