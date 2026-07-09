# CHANGELOG — `rasa.module.leads`

Reverse-chronological. Each entry is a version bump.

---

## 0.1.3 — 2026-07-09

### Element identity layer (canon SA-025)

- Added `rasa.identity` ("the RasaOS module for leads"); `bin/init` generates `.claude/rasa-identity.md` from it every install + stamps project-owned `.claude/rasa-deployment.md`; ships `/whoami`; CLAUDE.md "Who you are" header.

## 0.1.2 — 2026-07-09

### Added generic `/sync` + `/promote` + `/kit`-aware `bin/init` (canon SA-024)

- `bin/init` now clones the Element source into `<project>/kit/<element>/`; `/sync` smart-pulls upstream, `/promote` smart-pushes local edits back upstream (both directory-mirror → installed into consumers).

## 0.1.1 — 2026-07-09

### `parent_kind` → `[domain, tenant]` (canon SA-023)

- The `orchestrator` kind was folded into `tenant`; this module now mounts into a tenant or a domain (`requires.parent_kind: ["domain", "tenant"]`, was `["domain", "orchestrator"]`).

## 0.1.0 — INITIAL SHELL

Initial shell — spine + spec + adapter seam + ledger template; skills
deferred per extract-after-proof; account-handle spine threaded.

### Ships

- **`content/leads-rules.md`** — the portable sales-pipeline spine: the Lead
  record's canonical field set (id, account, contact_name, company, source,
  stage, value, owner, next_action, next_action_date, created, notes), the
  `new → contacted → qualified → proposal → won | lost` lifecycle, the shared
  `@acct-<slug>` account convention (verbatim), the soft boundary against
  `rasa.module.contracts` / `rasa.module.field-log`, and the ledger
  conventions. Element-owned (file-replace).
- **`content/README.md`** + **`content/BUILD_PLAN.md`** — author-time docs:
  what installs where, and the v0.1.0 specification (entity model, lifecycle,
  the leads-gate seam, the deferred skill contracts + the extract-after-proof
  gate). opt-in; not installed into consumers.
- **`seed/leads-gate.md.template`** → `.claude/leads-gate.md` — the adapter
  seam: the project's real pipeline stages + entry/exit criteria, its
  qualification bar, and its lead-source taxonomy. Project-owned
  (skip-if-exists); placeholder-free; honest defaults + commented examples.
- **`seed/leads/LEADS.md.template`** → `leads/LEADS.md` — the live pipeline
  board. Project-owned (skip-if-exists); one entry per lead, grouped by
  stage, joined via the account handle; terminal leads never deleted.
- **`seed/rasa.lock.json.template`** — Connection-Contract lockfile,
  SHA-stamped at init (init-only-with-sha).

### Shape

Toolkit `module`, `requires.parent_kind: [domain, orchestrator]`.
`capabilities: leads.capture, leads.pipeline, leads.qualification,
leads.account-linkage`. `permissions: fs:read, fs:write`.

### Deferred (build phase, per extract-after-proof)

- The `/lead`, `/leads`, `/qualify` skills — held until a real consumer
  declares `rasa.module.leads` in its `requires.elements[]`. Specified as
  M-1..M-3 in `content/BUILD_PLAN.md`.

### Notes

- Stripped the domain-core fork scaffold (SHAPE.md, agents/, rules/, skills/,
  output-style-enforcement/, seed CLAUDE.md + output-style templates); kept
  the canon-required lock template + bin/ + LICENSE + VERSION + .gitignore.
- The account hub the record joins on is canon task **SA-032**
  (engagement-hub primitive), not yet a first-class primitive.
- `bin/check-manifest` GREEN — rasa.json is a complete inventory.
