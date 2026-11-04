# nitro-boilerplate — Vercel's Nitro starter on Cloudflare Workers

**The starter is a Nitro server with one file-based route that answers with
a welcome page, and Nitro ships a preset for Workers, so the port keeps the
app as it is: Nitro's own `cloudflare_module` preset builds it into one
Cloudflare Worker with Workers Static Assets, a `/health` route beside it,
deployed by Orun to stage and prod on every merge — and the baseline that
makes a copy of it somebody else's app in one landing.**

| | |
|---|---|
| Status | In progress — NI0 on main |
| Upstream | [vercel/examples `framework-boilerplates/nitro`](https://github.com/vercel/examples/tree/main/framework-boilerplates/nitro) (MIT), at `de86723` (2026-10-16) |
| Reference | [express-api](https://github.com/sourceplane/express-api) — the same plain Worker, scaffolding, tests and bootstrap; [nuxt-boilerplate](https://github.com/sourceplane/nuxt-boilerplate) — the same Nitro preset |
| Key | `NI` |

## Read in this order

1. [design.md](design.md) — the starter, how each Vercel piece maps to
   Cloudflare, what the spike proved, the repository and the build, and the
   alternatives rejected.
2. [implementation-plan.md](implementation-plan.md) — the milestones and
   what each one lands.
3. [risks-and-open-questions.md](risks-and-open-questions.md) — what is not
   proven, and what was accepted.
4. [IMPLEMENTATION-STATUS.md](IMPLEMENTATION-STATUS.md) — what shipped, and
   every departure from the upstream starter.

## Milestones

| Milestone | What it lands | Task |
|---|---|---|
| NI0 — the spec | the repository, its intent, CI, tooling and spec index; this doc set | NI-1 |
| NI1 — Nitro on Workers | the starter in `apps/api`, built by the `cloudflare_module` preset, its tests, the README | NI-2 |
| NI2 — a baseline | the console card, the build document, the context pack and the baseline's tests | NI-3 |
