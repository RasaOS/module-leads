# CLAUDE.md — `rasa.module.leads`

Per-repo working contract for Claude sessions opened inside this folder.
Extends `~/.claude/CLAUDE.md` and the workspace `~/rAI/rasa-os/CLAUDE.md`
(the `rasa.tenant.rasaos` tenant's contract); does not override them.

## What you are when you're in this folder

You are working on **`rasa.module.leads`** — a `module`-kind business-ops
Element. It ships a portable **sales-lead pipeline**: capture a prospect,
move it `new → contacted → qualified → proposal → won | lost`, and keep the
history. Toolkit shape; mountable into any `domain` or `orchestrator` that
manages a book of business. It is one of six business-ops modules (`leads`,
`schedule`, `contracts`, `invoices`, `field-log`) built to a shared spine.

The Element owns the pipeline spine (`content/leads-rules.md`) and the spec
(`content/BUILD_PLAN.md`). The project owns the seam
(`.claude/leads-gate.md` — its real stages + qualification bar + source
taxonomy) and the live ledger (`leads/LEADS.md`).

### The account spine

Every lead references its customer by a stable **account handle**
(`account: @acct-<slug>`) — an opaque join key shared by all six business-ops
modules. This module **never invents or mutates the account's own record**;
the canonical account hub is canon task **SA-032** (engagement-hub
primitive), not yet a first-class primitive. Until it lands, records join by
the shared handle and refactor to the hub FK unchanged. Do not seed a
competing accounts registry here.

### Skills are deferred (extract-after-proof)

The `/lead`, `/leads`, and `/qualify` skills are **not authored yet** — they
are held per the RasaOS extract-after-proof precedent (`rasa.module.tasks`,
`rasa.module.notes`), built once a real consumer declares this module in its
`requires.elements[]`. Do not stub them ahead of a real consumer; the shell
(spine + spec + seam + ledger) is the v0.1.0 deliverable.

## Status

**v0.1.0 — initial shell.** Spine + spec + adapter seam + ledger template.
Skills deferred. See CHANGELOG.md.

## Source of truth

- **`~/rAI/rasa-os/canon/`** — authoritative for every architectural
  decision (Spec §6 defines the `module` kind). Canon wins.
- **`elements/domain-core/`** — the template this Element forked
  from. Shape questions go there, not here.
- **`rasa.json`** — this Element's formal declaration.
- **`~/rAI/rasa-os/elements/REGISTRY.md`** — the live workspace snapshot.

## Don'ts

- **You are NOT the template.** If this contract ever describes
  `domain-core` (or any other Element), the template-CLAUDE.md
  drift class is back — flag it.
- **Don't author content ahead of the authoring phase** without the user
  driving.
- **Don't `bin/init` this Element into itself.** `content/` is the
  source (workspace rule).
- **Don't push to GitHub from the Cowork sandbox.** Local commit + tag;
  the user pushes (workspace rule).

## How a version bump works

Each bump: edit `VERSION` + `rasa.json#version`, write a CHANGELOG entry,
run `bin/check-manifest`, commit + tag `v<version>`. Update
`~/rAI/rasa-os/elements/REGISTRY.md` +
`~/rAI/rasa-os/elements/CHANGELOG.md` (track #2).
