# Leads Rules

The portable sales-pipeline spine that `rasa.module.leads` installs at
`.claude/leads-rules.md`. It covers what this module owns, the canonical
Lead record, the pipeline lifecycle, the shared **account handle** every
business-ops module joins on, the boundary against adjacent modules, and
the ledger conventions. **Read this file when capturing a lead, advancing
one through the pipeline, qualifying one, or asking "where does this
prospect stand?"**

This file is **Element-owned** — it refreshes on upgrade. It deliberately
does **not** decide the one thing that varies per project:

- **The exact pipeline stages, what makes a lead "qualified", and the
  lead-source taxonomy.**

That lives in the project-owned **`.claude/leads-gate.md`** (the leads
analogue of the task module's done-gate and the release module's
release-gate). This file references the seam; it hardcodes none of it. The
`/qualify` step and any stage advance read the seam for the project's real
stage list and entry/exit criteria — inventing a project's sales process
is the one thing this module refuses to guess.

## What this module owns

One record type: the **Lead** — a prospective sale attached to a
prospective customer. The module owns the lead's whole life from first
contact to won or lost, and nothing else:

- It **owns** the lead pipeline: capture, the stage a lead sits in, its
  estimated value, its owner, and the next action that moves it forward.
- It does **not** own the customer account itself (that's the shared
  account hub — see the account spine below), the signed contract that a
  won lead becomes (`rasa.module.contracts`), or the delivery work that
  follows (`rasa.module.field-log`). Those are soft, present-if-mounted
  hand-offs, not things this module writes.

The proven pattern here is deliberately small: **a simple sales pipeline
over the shared account handle.** The per-project stages live in the gate;
the record and the discipline are the same everywhere.

## The Lead record — canonical field set

Every lead is one entry in `leads/LEADS.md`, carrying at minimum:

| Field | What it is |
|---|---|
| **id** | Stable lead id, `LEAD-NNN` monotonic, never reused. |
| **account** | The prospective customer's stable handle: `@acct-<slug>` (see the account spine below). The join key to every other module. |
| **contact_name** | The person you're talking to at the account. |
| **company** | The prospect's company/org display name (human-facing; `account` is the join key). |
| **source** | Where the lead came from — from the project's source taxonomy in the gate (e.g. `referral`, `inbound`, `event`, `outbound`). |
| **stage** | Current pipeline stage (see lifecycle). The project's real stage list lives in the gate. |
| **value** | Estimated deal value (a number + currency). An estimate, not a booked figure. |
| **owner** | The person accountable for advancing this lead. |
| **next_action** | The single next thing that moves this lead forward. |
| **next_action_date** | When that next action is due. Absolute date. |
| **created** | When the lead was captured. Absolute date. |
| **notes** | Freeform running context — call notes, objections, history. |

Rules for the record:

- **`id` is monotonic and never reused.** A dead lead keeps its number; a
  new lead gets the next one.
- **Dates are absolute** (`2026-07-05`), never relative ("next week").
- **`value` is an estimate.** It exists to size and rank the pipeline, not
  to book revenue. The real figure lands in the contract (soft hand-off).
- **`account` is a handle, not a name.** Display the company via `company`
  or the account hub; join via `account`.

## The lifecycle — the pipeline

A lead moves through a small, ordered set of states. The **default**
lifecycle this module ships is:

```
new ──▶ contacted ──▶ qualified ──▶ proposal ──▶ won
                                              └──▶ lost
```

| Stage | Meaning |
|---|---|
| **new** | Captured, not yet worked. The raw top of funnel. |
| **contacted** | First outreach made; a conversation has started. |
| **qualified** | Confirmed a real fit + budget + need — the gate defines exactly what "qualified" requires for this project. |
| **proposal** | A proposal / quote is out; you're negotiating terms. |
| **won** | Closed-won. The hand-off point to `rasa.module.contracts` (a contract) and `rasa.module.field-log` (a kickoff visit). |
| **lost** | Closed-lost. Terminal; record why in `notes`. Never deleted — a lost lead is institutional memory. |

`won` and `lost` are the two terminal states. A lead is **never deleted** —
it reaches a terminal state and stays in the ledger.

**The project's real stages live in the gate.** Some projects run a
five-stage funnel; some collapse `contacted`/`qualified` or add a
`nurture` holding stage. `.claude/leads-gate.md` is the authoritative stage
list + the entry/exit criteria for each. This file's lifecycle is the
default the gate overrides — never advance a lead against a stage the
project's gate doesn't declare.

## Customer / Account reference (shared spine)

> **Customer / Account reference (shared spine).** Every record this module
> owns references its customer by a stable **account handle**:
> `account: @acct-<slug>` in the record's frontmatter (for example
> `@acct-acme`). The handle is an opaque, stable join key. This module **never
> invents or mutates the account's own record** — the account's name,
> contacts, address, and billing details live on the **account hub**, which
> RasaOS does not yet have as a first-class primitive (tracked as canon task
> **SA-032 — engagement-hub primitive**). Until the hub lands, treat the
> handle as the join key and, if a project-owned account ledger is mounted,
> resolve display names from it; **do not seed a competing accounts registry
> here.** Every sibling business-ops module (`leads`, `schedule`, `contracts`,
> `invoices`, `field-log`) uses this same `@acct-<slug>` handle, so records
> join across modules today by shared key and refactor to the canonical hub
> FK — with no change to the records themselves — when SA-032 resolves.

Concretely for leads: a lead's `account: @acct-acme` is the same handle a
contract, an invoice, a scheduled visit, and a field-log entry for Acme all
carry. That shared key is what lets "everything about Acme" be assembled
across modules **today**, before the hub exists — and is why a won lead can
hand off to a contract or a kickoff visit without either module having to
re-identify the customer.

## The boundary against adjacent modules (soft references)

This module stands alone. When siblings are mounted it hands off by the
shared account handle — but every one of these is **soft**: present-if-mounted,
graceful-if-absent. None is a `requires.elements[]` dependency.

- **`rasa.module.contracts`** — when a lead reaches **won**, it can hand
  off to a contract (the won deal becomes a signed agreement). Leads does
  not write contracts; it records the win and points at the account handle.
- **`rasa.module.field-log`** — a won lead can trigger a kickoff visit /
  first field-log entry against the same account. Leads does not schedule
  or log the visit; it hands off the handle.
- **the account hub (SA-032)** — the eventual home of the customer's own
  record. Until it lands, `account` is the join key and this module never
  seeds a competing accounts list.

If a sibling isn't mounted, the hand-off simply doesn't happen — the lead
still reaches its terminal state cleanly. **Never harden a hand-off into a
hard dependency.**

## The seam — `.claude/leads-gate.md`

The one genuinely per-project thing. The gate holds:

1. **The pipeline stages** — the exact ordered stage list for THIS project,
   with the **entry/exit criteria** for each (what has to be true to enter a
   stage and to leave it).
2. **What "qualified" means** — the concrete bar a lead must clear to be
   marked `qualified` (budget confirmed? decision-maker identified? a named
   need?). `/qualify` reads this; it will not invent a qualification bar.
3. **The lead-source taxonomy** — the project's allowed `source` values.

Ships with honest defaults + commented examples. The project fills it. The
skills read it rather than hardcoding any project's sales process.

## Ledger conventions

- One `leads/LEADS.md`, git-versioned, project-owned (seeded
  skip-if-exists). It is the live pipeline board.
- Markdown, human-first. The ledger must read cleanly top-to-bottom to a
  person — a pipeline you can scan, not a query you have to run.
- One entry per lead. Group or sort by stage so the board reads as a
  funnel; keep terminal (`won`/`lost`) leads in the ledger for history.
- Dates absolute. `id`s are `LEAD-NNN` monotonic, never reused.
- **Never delete a lead.** A lost lead is data — why it was lost is the
  most useful thing in the ledger. Move it to terminal, don't remove it.

## Skills (deferred — the build phase)

v0.1.0 is a shell: the spine (this file), the specification
(`content/BUILD_PLAN.md`), the seam template, and the ledger template. The
**skills are not authored yet** — they are held per the RasaOS
extract-after-proof precedent (`rasa.module.tasks`, `rasa.module.notes`),
built once a real consumer declares this module in `requires.elements[]`.
The intended skills, specified as milestones in `BUILD_PLAN.md`:

- **`/lead`** — capture a new lead + advance an existing one through stages.
- **`/leads`** — render the pipeline board (the funnel, by stage).
- **`/qualify`** — score a lead against the gate's qualification bar and
  advance it to `qualified` (or record why it didn't clear).

Until then, the ledger is maintained by hand against the discipline in this
file. Do not stub the skills — the shell is the deliverable.
