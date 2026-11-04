# Epics

Orun-style work programs. Each epic is a folder carrying `epic.yaml` (the
declaration the platform reads: slug, key prefix, milestones with exit
criteria and task keys) beside its doc set. The pull request that adds the
folder is the adoption; the merge creates the epic in the workspace.

| Epic | Cluster | Status | What it is |
|------|---------|--------|------------|
| [`nitro-boilerplate/`](./nitro-boilerplate/) | **NI** | ✅ Shipped (`baseline-v1`); live bootstrap pending | Vercel's Nitro starter, its route kept, built by Nitro's own cloudflare_module preset into one Cloudflare Worker — and an Orun baseline that instantiates it in one landing (NI2) |

As-built ≠ intent: what shipped lives in each epic's `IMPLEMENTATION-STATUS.md`.
