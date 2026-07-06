# Rasa · Module · Leads

**Canonical name:** `rasa.module.leads`
**Repo / folder:** `module-leads`
**Kind:** `module` (canon Spec §6)
**Contract:** Element Contract v1.3.0
**Version:** 0.1.0 (initial shell)
**Shape:** toolkit

## What this is

A portable **sales-lead pipeline** — capture a prospect, move it
`new → contacted → qualified → proposal → won | lost`, and never lose the
history. Mountable into any `domain` or `orchestrator` that manages a book
of business.

It is one of six business-ops modules built to a shared spine — `leads`,
`schedule`, `contracts`, `invoices`, `field-log` — that all join on the same
`@acct-<slug>` account handle.

## The record it owns

One record type: the **Lead** — `id`, `account` (the `@acct-<slug>` handle),
`contact_name`, `company`, `source`, `stage`, `value` (estimated), `owner`,
`next_action`, `next_action_date`, `created`, `notes`. Owned from first
contact to won or lost; never deleted.

## The account spine

Every lead references its customer by a stable **account handle**
(`account: @acct-<slug>`). The handle is an opaque join key — this module
**never invents or mutates the account's own record**. The canonical account
hub is canon task **SA-032** (engagement-hub primitive), not yet a
first-class primitive; until it lands, all six sibling modules join by the
shared handle and will refactor to the hub FK unchanged. Full convention in
`content/leads-rules.md` → *Customer / Account reference*.

## Element- vs project-owned files

| File | Owner | Policy |
|---|---|---|
| `.claude/leads-rules.md` | Element (refreshed on upgrade) | file-replace |
| `.claude/leads-gate.md` | **Project** — the stages + qualification bar + source taxonomy | skip-if-exists |
| `leads/LEADS.md` | **Project** — the live pipeline board | skip-if-exists |
| `.claude/rasa.lock.json` | Connection-Contract lockfile | init-only-with-sha |

## Skills deferred to build phase

The `/lead`, `/leads`, and `/qualify` skills are **not authored yet** — they
are held per the RasaOS extract-after-proof precedent (`rasa.module.tasks`,
`rasa.module.notes`), built once a real consumer declares this module in its
`requires.elements[]`. This v0.1.0 ships the spine + spec + seam + ledger
template. See `content/BUILD_PLAN.md` for the skill contracts and the gate.

## See also

- `~/rAI/rasa-os/elements/module-notes/` — the shell precedent this Element
  follows (spine + spec + seam + ledger, skills deferred).
- `content/BUILD_PLAN.md` — the full specification + build plan.
- Canon Spec §6 — the `module` kind; `ELEMENT_CONTRACT.md` §7 — install
  policies.
- `~/rAI/rasa-os/elements/REGISTRY.md` — the live workspace snapshot.
