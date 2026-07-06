# `rasa.module.leads` — Specification & Build Plan

**Status:** v0.1.0 = spine + spec + seam + ledger template. The skills are
the build phase (M-1..M-3 below), deliberately gated.

This is the author-time specification. The installed spine is
`content/leads-rules.md`; this file is the design record behind it.

---

## Why this module exists

Every business a RasaOS vertical serves has a top of funnel: prospects come
in, get worked, and either close or don't. That pipeline is the same shape
everywhere — the fields, the states, the discipline of "one next action per
lead" — while the *stages* and *what counts as qualified* differ per
business. `rasa.module.leads` distills the invariant sales pipeline into a
portable module and pushes the per-business part into a seam.

It is one of six business-ops modules built to a shared spine — `leads`,
`schedule`, `contracts`, `invoices`, `field-log` — that all join on the
same `@acct-<slug>` account handle. That shared key is what makes them a
coherent CRM-shaped set instead of six disconnected lists.

## Non-goals (hard boundaries)

- **Not the account/customer record.** Leads references the customer by
  handle; it never owns the account's name, contacts, or billing. That's
  the account hub (canon SA-032). Do not seed a competing accounts list.
- **Not the contract.** A won lead *hands off* to `rasa.module.contracts`;
  it does not author the signed agreement or book real revenue. `value` is
  an estimate for sizing the pipeline.
- **Not the delivery.** A won lead can trigger a kickoff visit via
  `rasa.module.field-log`; leads does not schedule or log the visit.
- **Not a task tracker.** The lead's `next_action` is a pipeline field, not
  an assignable work item — that's `rasa.module.tasks` if mounted.

---

## Entity model

One project-owned ledger + one project-owned seam.

### The Lead record (in `leads/LEADS.md`)

| Field | Type | Notes |
|---|---|---|
| **id** | `LEAD-NNN` | Monotonic, never reused. |
| **account** | `@acct-<slug>` | The shared account handle — the join key to every sibling module. |
| **contact_name** | text | The person at the account. |
| **company** | text | Display name of the prospect's org. |
| **source** | enum | From the gate's lead-source taxonomy. |
| **stage** | enum | From the gate's stage list (default: new/contacted/qualified/proposal/won/lost). |
| **value** | number + currency | Estimated deal value. An estimate, not booked revenue. |
| **owner** | text | Accountable for advancing the lead. |
| **next_action** | text | The single next step. |
| **next_action_date** | date | When it's due. Absolute. |
| **created** | date | When captured. Absolute. |
| **notes** | freeform | Call notes, objections, history. |

Rules: `id` monotonic + never reused; dates absolute; `account` is a handle
not a name; leads are never deleted (terminal `won`/`lost` stay in the
ledger).

### The lifecycle

```
new ──▶ contacted ──▶ qualified ──▶ proposal ──▶ won
                                              └──▶ lost
```

`won` and `lost` are terminal. The **project's real stage list + entry/exit
criteria live in the gate** — this is the default the gate overrides.

### The account spine

Every record carries `account: @acct-<slug>`. The verbatim shared-spine
convention is in `content/leads-rules.md` → *Customer / Account reference*.
Summary: opaque stable join key; this module never mutates the account's
own record; the canonical account hub is canon task **SA-032**; until it
lands, records join across the six modules by shared handle and refactor to
the hub FK unchanged when SA-032 resolves.

---

## The adapter seam — `.claude/leads-gate.md`

The crux of the design, mirroring `module.tasks`' done-gate and
`module.releases`' release-gate. It holds the three things that genuinely
vary per business:

1. **The pipeline stages** — the exact ordered stage list, with **entry/exit
   criteria** per stage. The default lifecycle is a starting point, not a
   mandate.
2. **What "qualified" means** — the concrete bar (budget confirmed?
   decision-maker identified? named need?). `/qualify` hard-reads this and
   refuses to invent a qualification bar.
3. **The lead-source taxonomy** — the allowed `source` values.

Why a seam and not fixed logic: a project's sales *process* is exactly the
part this module cannot know — the same reason the done-gate holds "what
counts as done." The horizontal core is *the record + the pipeline
discipline*; the vertical part is *the specific stages and qualification
rule*.

---

## Skills (the build phase — M-1..M-3)

Three skills, MVP-scoped. Each is a thin driver over the ledger + seam; the
discipline lives in `leads-rules.md`, not duplicated per skill. **Not
authored in v0.1.0** — see the gate below.

### M-1 — `/lead`
- **`/lead <name/company>`** — capture a new lead into `leads/LEADS.md`
  (assigns the next `LEAD-NNN`, sets `stage: new`, requires an `account`
  handle and an `owner`).
- **`/lead advance LEAD-NNN`** — move a lead to the next stage, checking
  the gate's exit criteria for the current stage. Sets the next action.

### M-2 — `/leads`
- **`/leads [stage <s>|owner <o>|account <@acct>]`** — render the pipeline
  board: the funnel by stage, with value totals. Read-only. Default view is
  the whole active pipeline grouped by stage.

### M-3 — `/qualify`
- **`/qualify LEAD-NNN`** — score a lead against the gate's qualification
  bar. **Hard-stops if the gate has no qualification criteria** (guessing a
  project's "qualified" bar is the one inference the skill refuses). On a
  pass, advances to `qualified`; on a fail, records why in `notes` and
  leaves the stage.

**Style/quality bar:** match the sibling modules' SKILL.md files — a crisp
operation list, honest hard-stop behavior, no invented plumbing. Fan-out
plan: author `/lead` as the reference skill, then `/leads` + `/qualify` in
parallel against it.

---

## The gate — when to build the skills

Per the RasaOS extract-after-proof precedent (`module.tasks`,
`module.notes`), hold the skill build until **one real consumer** declares
`rasa.module.leads` in its `requires.elements[]` — a domain or orchestrator
that actually runs a sales pipeline. Building the skills before that
consumer would repeat the premature-abstraction trap the shell exists to
avoid.

The natural first consumer is whichever vertical orchestrator manages a
book of business (a firm/company orchestrator, or the soccer-club platform's
registration/tryout funnel viewed as a pipeline).

---

## Soft cross-references

All soft — present-if-mounted, graceful-if-absent, never
`requires.elements[]`:

- **`rasa.module.contracts`** — a won lead hands off to a contract.
- **`rasa.module.field-log`** — a won lead can trigger a kickoff visit.
- **the account hub (SA-032)** — the eventual canonical customer record.

---

## Version plan

- **v0.1.0 (this)** — spine (`leads-rules.md`) + spec (this file) + seam
  template + ledger template. No skills.
- **v0.2.0** — M-1..M-3 skills authored, once a real consumer declares the
  module. `bin/init` smoke-tested into that consumer.
- **v1.0.0** — the seam format + install shape locked after the pipeline
  has run through at least one real vertical unchanged.
