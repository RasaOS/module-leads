# `rasa.module.leads` — content

What this module ships and where it installs. This file is author-time
documentation (not installed into consumer projects).

## The one-liner

The **sales-pipeline** module: a portable `new → contacted → qualified →
proposal → won | lost` lead pipeline as git-versioned markdown. Every lead
attaches to the shared `@acct-<slug>` account handle, so it joins across
every sibling business-ops module.

## What installs where

| Source | Installs to | Policy | What it is |
|---|---|---|---|
| `content/leads-rules.md` | `.claude/leads-rules.md` | file-replace | The spine — the Lead record, the pipeline lifecycle, the account handle, the module boundary. Element-owned; refreshed on upgrade. |
| `seed/leads-gate.md.template` | `.claude/leads-gate.md` | skip-if-exists | **The seam** — the project's real pipeline stages, its qualification bar, its lead-source taxonomy. Project-owned. |
| `seed/leads/LEADS.md.template` | `leads/LEADS.md` | skip-if-exists | The live pipeline board. Project-owned; append + advance. |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | init-only-with-sha | Connection-Contract lockfile, SHA-stamped at init. |

## What is NOT here yet

The `/lead`, `/leads`, `/qualify` **skills** — they are the build phase.
See [`BUILD_PLAN.md`](BUILD_PLAN.md) for their contracts and the
extract-after-proof gate that holds them until a real consumer declares the
module in `requires.elements[]`.

## The shape

Toolkit module, `requires.parent_kind: [domain, orchestrator]`. Pure
Element-layer convention — no kernel engine. The account hub it joins on is
canon task **SA-032** (not yet a primitive); cross-refs to
`rasa.module.contracts` and `rasa.module.field-log` are soft
(present-if-mounted).

## See also

- [`BUILD_PLAN.md`](BUILD_PLAN.md) — the full spec + build plan.
- `content/leads-rules.md` — the installed spine.
- `elements/module-notes/` — the shell precedent this module follows
  (spine + spec + seam + ledger, skills deferred).
- Canon `ELEMENT_CONTRACT.md` §7 — install policies. Canon Spec §6 — the
  `module` kind.
