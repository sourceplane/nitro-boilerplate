# nitro-boilerplate — open risks

What is not yet proven or not yet decided. The full write-up of every risk,
including the accepted and resolved ones, is
`specs/epics/nitro-boilerplate/risks-and-open-questions.md`; this page lists only
what is still live, so it should be short.

| ID | Risk | State | What closes it |
|---|---|---|---|
| NI-A | No live bootstrap yet: the repository created and linked, the deploy token minted, a real `wrangler deploy` of Nitro's build, the smoke and the probes against workers.dev, and the routes read from the deployed Worker are all unproven | OPEN | One bootstrap from a tag against a workspace |
| NI-C | The committed `wrangler.jsonc` names Nitro's output layout (`.output/server/index.mjs`, `.output/public`); a Nitro release that moves it breaks the deploy | WATCH | The e2e (the lane's build, dry run and `wrangler dev` over the output) on every change, and a deliberate `nitropack` bump |
| NI-E | `orun plan --changed` plans nothing for a push with no `before`; the first landing relies on merging into orun's seeded initial commit | WATCH | Confirmed by the first live bootstrap |

Accepted, and worth knowing before you change them: NI-B (`nitropack` pinned
to 2.13.4, not the template's `latest`), NI-D (Nitro's `useStorage()` is in
memory on the Worker; the template uses none), NI-F (stack-tectonic 0.19.0),
NI-G (the welcome page's logo is hot-linked from nitro.build, as upstream).
