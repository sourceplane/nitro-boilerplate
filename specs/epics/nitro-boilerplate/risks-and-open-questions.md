# nitro-boilerplate — risks and open questions

## NI-A — No live bootstrap yet (OPEN)

Everything short of the platform is proven on every pull request: the card
against the build, the blueprint read by the pinned orun, the leak gate, the
offline bootstrap with the blueprint's own hooks, the app's own plan for the
01-product merge, and the produced app built by Nitro, dry-deployed, served,
smoke-tested and read route by route in the job tree orun stages. Not proven:
the repository created and linked, the token minted, a real `wrangler
deploy`, the smoke against workers.dev, and the routes read from the deployed
Worker.

## NI-B — `nitropack` pinned to 2.13.4 (ACCEPTED)

The template asks for `latest`. A floating major would let Nitro 3 (at
release candidate) land in an owner's build with no commit; 2.13.4 is what
npm called `latest` on the day of the port. Upgrading is a pull request that
bumps the pin, regenerates the lockfile and runs the tests and the e2e.

## NI-C — The committed config names Nitro's output layout (WATCH)

`wrangler.jsonc`'s `main` is `.output/server/index.mjs` and its assets
directory `.output/public`: Nitro 2's layout for the `cloudflare_module`
preset. A Nitro release that moves either breaks the dry run and the deploy.
The e2e replays the lane's build and dry run and serves the output on every
change, so a bump that moves them fails before it merges.

## NI-D — `useStorage()` is in memory on the Worker (ACCEPTED)

Nitro's storage layer defaults to an in-memory driver on the Workers preset:
data does not survive an isolate, and two isolates do not share it. The
template uses no storage. An owner who needs it mounts a driver (Workers KV
through unstorage's `cloudflare-kv-binding`, with a KV namespace added under
`infra/`) in `nitro.config.ts`.

## NI-E — `orun plan --changed` and the first landing (WATCH)

`orun plan --changed` plans nothing for a push with no `before`; the first
landing relies on merging into orun's seeded initial commit, as every
baseline of this family does. Confirmed by the first live bootstrap.

## NI-F — stack-tectonic 0.19.0 (ACCEPTED)

The family's one pin (`intent.yaml` explains why 0.20.0 is not taken). The
Worker builds with Nitro and bundles with wrangler's esbuild, so 0.20.0's
linked install would work for it; the pin is kept for one convention.

## NI-G — The welcome page's logo is hot-linked (ACCEPTED)

Upstream's page loads Nitro's logo from `https://nitro.build/icon.svg`. Kept
as it is: the page is the template's, and a logo of nitro.build's own is
theirs to serve. An owner who wants no third-party request adds the file
under `server/public/` and points the `<img>` at it.
