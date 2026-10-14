---
name: domain-explorer
description: Research a domain or area of a codebase (e.g. "Activities") end to end and produce a cited dossier — architecture and data-flow diagrams, data model, entity lifecycle, every use case traced from trigger to data to UI, glossary, history, and open questions — then answer follow-ups from it. Use when asked to explore, understand, map, onboard onto, or explain a feature, domain, or part of a codebase.
---

# Domain Explorer

Orchestrator–worker codebase research, in the spirit of Anthropic's multi-agent research system, PocketFlow's codebase-to-tutorial pipeline, DeepWiki / Google Code Wiki, Windsurf Codemaps and Sourcegraph Deep Search. `@domain-explorer` leads; `@domain-scout` workers each research one **lens** or trace one **use case** in parallel and write their findings to files. The lead synthesizes, verifies, and teaches.

## Principles

1. **Cited or it didn't happen.** Every claim, table row, and diagram edge carries a `path:line` the author actually read. Anything uncited is marked ❓ and listed under Open questions. (Deep Search: list the searches and files behind every answer.)
2. **Map, then explain.** Find structure mechanically first (`rg`, stack templates, table definitions, routes), and only then name and group it. Diagrams are drawn from extracted relations, not from memory. (CodeBoarding, PocketFlow.)
3. **Start wide, then narrow.** Short broad searches for the domain's vocabulary first; specific searches once the shape is known. (Anthropic.)
4. **Files, not context.** Workers write to the dossier directory and return a path plus a 3-line summary. The plan lives in `plan.md` so it survives context limits. (Anthropic.)
5. **Use cases are the spine.** A domain is understood when every way it is triggered has been traced end to end: actor → trigger → entry point → logic → data written → side effects → where the result shows up. (Codemaps, codebase-onboarding.)
6. **Foundational first.** Readers get the model before the mechanics: glossary → data model → architecture → use cases → edge cases. (PocketFlow chapter ordering.)
7. **Speak the product's language.** Pair every code name with the user-facing label from the frontend, and the business term if one exists. Repo `AGENTS.md` / glossary rules apply.

## Depth (effort scaling)

| Depth | Lenses | Use-case traces | History | Typical scouts |
|---|---|---|---|---|
| `quick` | L1, L2, L4 | top 3 | no | 3–4 |
| `standard` (default) | all L1–L9 | top 8 | light | 9–14 |
| `deep` | all L1–L9 | all, max 20 | full | 15–30 |

Never spawn more scouts than the domain needs: merge lenses with nothing to find, and skip traces that duplicate one already done.

## Lenses (round 1 — one scout each)

| Id | Lens | The scout answers |
|---|---|---|
| L1 | **Data model** | Tables / collections / indexes holding the domain: keys, GSIs, important fields and their types, item shapes (one real example per item type), TTLs, search indexes, which code writes each field. Denormalized copies of domain data in other tables. |
| L2 | **Entry points** | Every way the domain is triggered: HTTP routes (method + path + handler), Lambda handlers, stream consumers, cron/scheduled jobs, webhooks, queues, MCP tools, CLIs, admin scripts. One row each, with the stack/template that wires it. |
| L3 | **Write paths** | Every code path that creates, updates, or deletes domain data: trigger → steps → validation → writes → side effects. Shared helpers vs duplicated logic. Idempotency, conditional writes, consistency. |
| L4 | **UI & clients** | Every page, component, store, and extension surface that shows or changes the domain: user-facing labels, which API each calls, filters/sorts offered, empty and error states. |
| L5 | **Async & propagation** | Streams, consumers, fan-out, denormalization, search indexing, notifications, scheduled recomputation. What happens *after* a write, in order, and what retries on failure. |
| L6 | **Integrations** | External systems pushing into or pulling from the domain (CRMs, ATSs, email/calendar, vendors): direction, mapping, sync triggers, conflict handling. |
| L7 | **Access & tenancy** | Who can see and do what: auth, roles, team/org scoping, ownership checks, impersonation, feature flags, plan gating. |
| L8 | **History & decisions** | Why it's shaped like this: `git log` on the core files, the PRs that introduced key pieces (`gh pr list --search`), ADRs, migrations/backfills, legacy or dead paths, flags mid-rollout. |
| L9 | **Tests & sharp edges** | Tests covering the domain (and gaps), TODO/FIXME/HACK, known bugs referenced in code, error handling, limits (sizes, pagination caps, timeouts), surprising behaviour. |

## Use-case harvest (between rounds)

The lead builds the use-case catalog from L2 (entry points) × L4 (UI actions) × L5 (async triggers) × L6 (integrations):

- One use case = **one actor with one goal** ("Recruiter logs a call against a candidate", "CRM sync imports a note", "Nightly job rolls up activity counts").
- Actors include humans (by role), the system (cron, streams), and external systems.
- Merge entry points that serve the same goal; split one entry point that serves different goals.
- Number them `UC-01…`, ranked by how often they run or how central they are (frontend-reachable, called from many places, or on a hot path rank higher).

## Use-case traces (round 2 — one scout per use case)

Each trace follows one use case end to end and returns:

- Actor, goal, trigger, and preconditions
- Ordered steps, each with `path:line`: UI action → API call → handler → domain logic → writes → async consequences → where the result appears
- Data written/changed (table.field)
- Failure modes: what happens on error at each risky step, and what state is left behind
- Variants and branches (flags, plan tiers, integration-specific behaviour)
- A Mermaid `sequenceDiagram`

## Dossier layout

Write everything under `<out>/<domain-slug>/` (default `<out>` = `./domain-dossiers`):

```
10-big-picture.md      ← plain-English one-pager; the human front door (Phase 5b)
README.md              ← reference index: summary, reading order, diagram index, open questions
plan.md                ← lead's plan, depth, lens assignments, status (crash-resume)
00-recon.md            ← vocabulary, footprint by directory, seed files
01-glossary.md         ← term | code name(s) | UI label | meaning | defined at
02-data-model.md       ← tables/fields + erDiagram + lifecycle stateDiagram
03-architecture.md     ← context + container flowcharts, data-flow flowchart
04-use-cases.md        ← catalog table + one section per traced use case (with sequenceDiagram)
05-async-and-integrations.md
06-access-and-tenancy.md
07-history-and-decisions.md
08-sharp-edges.md      ← limits, gotchas, test gaps, TODOs
09-open-questions.md   ← ❓ items, contradictions between scouts, things to ask a human
lens-L1-data-model.md … lens-L9-*.md, trace-UC-01.md …   ← raw scout output (kept for audit)
```

## Diagram rules

Every diagram is Mermaid in a fenced block, built from cited relations:

- **Context** (`flowchart LR`): the domain as one box; actors, external systems, and neighbouring domains around it.
- **Containers** (`flowchart TD`): entry points → handlers/Lambdas → shared modules → tables/indexes/streams → consumers → UI.
- **Data flow** (`flowchart LR`): one written field's journey through streams, copies, and indexes to its readers.
- **Data model** (`erDiagram`): tables and the keys that relate them.
- **Lifecycle** (`stateDiagram-v2`): the entity's status values and what code transitions between them.
- **Use case** (`sequenceDiagram`): one per traced use case.

Constraints:

- ≤ 15 nodes per diagram; split into several rather than cram.
- Pipe-style edge labels (`A -->|writes| B`); quote labels with special characters (`A["get_activities()"]`).
- Under each diagram, a **Sources** list mapping each node/edge to `path:line`. Uncited edges are dashed (`-.->`) and labelled ❓.
- If the `beautiful-mermaid` skill is available, render each diagram once to catch syntax errors; fix before shipping.

## Phase 5b — Big-picture page (mandatory)

Write `10-big-picture.md` — the front door for humans who will never read the rest of the dossier:

- Plain English, ELI5: short sentences, everyday words, **no metaphors or analogies**. Introduce a code name only with a plain gloss in the same sentence.
- Structure:
  1. What the domain is, in product language (one short paragraph).
  2. "The N things called `<term>`" — each distinct meaning the domain's name covers, named explicitly and never merged.
  3. One line each: where that meaning lives (which store) and who reads it.
  4. If more than one surface exposes the domain: a short comparison table (what each shows, naming differences, gaps).
  5. **"5 things to remember"** — exactly five, numbered, one line each.
  6. "Where to read more" — one line per numbered dossier file.
- One screen long. At most a handful of `path:line` anchors; the numbered files carry the receipts.
- Link it first from `README.md` ("New here? Read `10-big-picture.md`").

## Verification (before presenting)

The lead spawns verify-mode scouts over the synthesized dossier. Each one re-opens the cited `path:line` for every diagram edge, catalog row, and glossary entry in its assigned files, and returns ✅ / ❌ (with the correct citation) / ❓. The lead fixes ❌, keeps ❓ visible, and records contradictions between scouts in `09-open-questions.md`. Never present a dossier whose diagrams haven't been verified.

## After delivery — teach mode

The dossier becomes the knowledge base (DeepWiki / Code Wiki chat):

- Answer follow-ups from the dossier first, and go back to the code when it's silent; cite either way. Update the dossier when an answer adds something new.
- Offer next steps: a guided walkthrough in reading order, a quiz on the glossary and lifecycle, an animated explainer of the hero use case (`@explainer-builder`), or an architecture review of the domain (`@feature-reviewer`).
