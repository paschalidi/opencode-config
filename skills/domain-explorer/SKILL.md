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

## Notion page (PHASE 6b) — the only thing most people will read

The dossier is evidence; the Notion page is the documentation. `@domain-editor` writes it to `<dossier>/notion-page.md` in Notion-flavored Markdown, a fresh reader tests it, and the lead publishes it. Progressive disclosure: everything a newcomer needs is visible; everything a senior needs is one click away in a toggle.

### Template

```
<callout icon="💡" color="blue_bg">
	**In short:** <the direct answer to the brief, 2–3 sentences, product language>
</callout>
<one line: what this page covers · ~5 min to read the top · open the toggles for depth>

## What "<term>" means here
<the N meanings / core concepts, one bullet each: plain name — one-line meaning — where it lives>

## How it fits together
<one mermaid flowchart LR, ≤ 12 nodes, product names on boxes>
<one plain sentence per box or group of boxes, as bullets>

## 5 things to remember
1. … (exactly five, one line each)

## <Topic section> {toggle="true"}
	<2–3 plain sentences a junior can follow>
	<detail: bullets, ≤ 1 small table>
	<details>
	<summary>Where in the code</summary>
	- [path/to/file.py](<repo link>#L10-L25) — what's there
	</details>
… 4–6 topic sections, chosen from the brief (typical: where the data lives · how it gets created · where it shows up · who can see what · how it got this way · gotchas)

## Reference {toggle="true"}
	<glossary table: term | UI label | meaning — ≤ 15 rows>
	<one more table if the brief needs it, e.g. surfaces/endpoints — ≤ 12 rows>

## Open questions ({N}) {toggle="true"}
	- one line each, most important first, ≤ 8

---
<small footer line: generated from the research dossier at <path> on <date>; the dossier holds the full evidence>
```

### Budgets (hard limits)

| Part | Limit |
|---|---|
| Visible layer (callout + everything outside toggles) | ≤ 600 words |
| Each toggle section | ≤ 300 words, ≤ 1 table |
| Whole page | ≤ 2,500 words |
| Diagrams | ≤ 2 on the page, ≤ 12 nodes each, `flowchart LR` or `sequenceDiagram`; the second only inside a toggle |
| Tables | ≤ 15 rows; columns ≤ 4 |
| Code references | only inside "Where in the code" toggles, as GitHub links, ≤ 6 per section |

### Cut rules (lint before writing)

- No dossier-internal labels: lens ids (`L3`), use-case ids (`UC-07`), file names like `trace-*.md`/`verify-*.md`, "scout", "dossier phase".
- No ❓, "unverified", or hedging in the main text — uncertain items go to the Open questions toggle only.
- No `path:line` in prose — code references live in the "Where in the code" toggles as links.
- Every code name is paired with its UI label or a plain gloss on first use.
- No metaphors or analogies. No marketing words. No "simply", "just", "easy".
- Numbers only when they change a decision (limits, counts a developer will hit).
- Notion syntax: tabs for indentation; toggle children indented one tab; escape `\ * ~ ` $ [ ] < > { } | ^` outside code blocks; Mermaid labels with special characters in double quotes, `<br>` for line breaks inside labels.

### Reader test

A fresh scout in `reader` mode reads **only** `notion-page.md` — no dossier, no code — and answers:

1. Every question from the user's original brief.
2. Three newcomer questions: "What is <term> in one sentence?", "If I needed to change how <term> is shown to users, where would I start?", "What's the one thing most likely to surprise me?"

For each: answer, then `answered / partly / couldn't`, plus every term it found unexplained. The editor revises once on anything below "answered". A second reader test runs only if the first had 3 or more misses.

## Notion pages — Diátaxis format (PHASE 6b, `format: diataxis`)

The alternative to the single page: a small set of pages, each of one Diátaxis type, never mixed. Use it when the domain will grow, or when readers come with different jobs (understand vs look up vs do). Same research, same cut rules, same reader test.

Files go in `<dossier>/notion-diataxis/`:

| File | Diátaxis type | Purpose | Budget |
|---|---|---|---|
| `00-index.md` | landing | The answer + a page picker | ≤ 250 words |
| `01-how-it-works.md` | Explanation | Understand: what it is, how it fits, why it's like this | ≤ 1,200 words, ≤ 2 diagrams |
| `02-reference.md` | Reference | Look up: facts in tables, no narrative | ≤ 1,500 words, tables ≤ 20 rows |
| `03-working-with.md` | How-to | Do: 3–5 task recipes | ≤ 1,000 words; **omit the page** if the dossier supports fewer than 3 tasks |

### `00-index.md`

```
<callout icon="💡" color="blue_bg">
	**In short:** <direct answer to the brief, 2–3 sentences>
</callout>
<table header-row="true">  page | read this when… | you'll learn | read time  </table>
## Open questions ({N}) {toggle="true"}
	- … ≤ 8
<footer line: generated from <dossier path> on <date>>
```

### `01-how-it-works.md` — Explanation

Understanding only. No step-by-step instructions, no lookup tables longer than 5 rows.

- What "<term>" means here (each meaning, one paragraph)
- How it fits together (one `flowchart LR` ≤ 12 nodes + one sentence per box)
- How the main paths work (one `sequenceDiagram` for the most central flow, in a toggle)
- Why it's like this (history, as a short dated list)
- 5 things to remember
- "Where in the code" toggles per section, as in the single page

### `02-reference.md` — Reference

Facts only. No explanation beyond one plain line under each heading. Tables, in this order where the dossier supports them:

- Glossary: term | UI label | meaning
- Types: type | stored or derived | where it shows | notes
- Storage: store | keys | what's in it | written by
- Surfaces / endpoints: surface | method + path | what it returns | auth
- Limits and gotchas: limit | value | where it bites
- Every table row that cites code links to it in the last column.

### `03-working-with.md` — How-to

Action only, for tasks a developer on this domain will actually do (examples: "add a new type", "find out why an item is missing from a screen", "read this data through the public API"). Each recipe:

```
## <Task as a goal> {toggle="true"}
	**When:** <one line>
	1. <step with a code link>
	2. …
	**Watch out:** <1–2 gotchas from the research>
```

Only recipes the dossier supports with code paths. Never invent a procedure.

### Diátaxis rules (lint, in addition to the Cut rules)

- Each page holds one type. Explanation that creeps into Reference or How-to moves to `01` and is linked.
- No fact is written out twice. The second place links to the first.
- Child pages open with one line saying what the page is for and linking back to the index.

## Question-led brief (PHASE 6b, `format: brief`) — recommended

Built from what developers actually do with domain docs: they arrive with a question, forage for a good-enough answer, and learn best from one concrete example plus the reasons behind the design (Fraunhofer developer survey; Sillito et al., questions programmers ask; expert-to-novice explanation study; Every Page is Page One; Domain Storytelling). Inventory goes in a database, not prose.

Files go in `<dossier>/notion-brief/`:

| File | What | Budget |
|---|---|---|
| `00-hub.md` | Front page | ≤ 350 words, 1 diagram |
| `01-worked-example.md` | One concrete story followed through every surface | ≤ 600 words, 1 diagram |
| `q-<slug>.md` | One page per question, 5–8 of them | ≤ 400 words, ≤ 1 diagram each |
| `decisions.md` | Why it's like this + open decisions | ≤ 500 words |
| `types.json` | Rows for the Notion database | — |
| `diagrams/briefs/<id>.md` + `diagrams/registry.json` | Diagram briefs for `@domain-diagrammer` | — |

### `00-hub.md`

```
<callout icon="💡" color="blue_bg">
	**<Term> is** <one-sentence definition in product language>.
</callout>
**Terms you'll meet:** <5–7 terms, each "**term** — 6–12 word meaning", one line each>
<!-- diagram: overview -->
## Questions this answers
1. <question as a developer would ask it> → <one-line answer> (links to its q- page)
… 5–8 questions
## Worked example
<one line> → links to 01-worked-example
## Reference
- **<Types> database** — every type with its UI label, code, API name, stored/derived, where it shows (filterable; saved views answer the comparison questions)
- **Decisions** → links to decisions
<freshness line: generated <date> from commit <sha> · confidence <high/medium> · open questions <N> · owner <team or person, or "unassigned">>
```

### Choosing the questions

Start from the user's brief, then cover these developer question types where the research supports them (Sillito et al.): *what is it / where is it defined*, *what happens when …* (one end-to-end flow), *where does this data go*, *why is A different from B*, *where do I change …*, *who can see / do what*, *why is it built like this*. Phrase each exactly as a developer would type it into search. Merge or drop questions the research can't answer.

### `01-worked-example.md` — one concrete story

Pick the single scenario that exposes the most important behaviour (for a parity question: one interaction seen through every surface). Named, realistic but generic people ("Anna, a recruiter, emails Bob").

```
<one-line setup>
<!-- diagram: worked-example -->
1. **<Actor> <does something>.** <what gets recorded where, 1–2 lines> — then: what each surface shows at this moment
2. …  (4–7 numbered steps, Domain Storytelling style: who does what with what)
**What this shows:** <2–3 bullets — the lessons the story proves>
<details><summary>Where in the code</summary> links </details>
```

### `q-<slug>.md` — one answer page

Every page stands alone (a reader may land here first).

```
<one line: "Part of <Domain>: <link to hub>">
**Short answer:** <1–3 sentences>
<!-- diagram: <id> -->   (only if a picture answers it better than text)
<explanation: ≤ 4 short paragraphs or bullets, concrete, no inventory lists>
**Watch out:** <1–3 gotchas>
<details><summary>Where in the code</summary> ≤ 6 links </details>
```

### `decisions.md`

ADR-lite. For each past decision that shaped the domain: **Decision · When · Why (as far as the evidence shows) · Consequence today**. Then **Open decisions**: the questions a team must answer next, each with the options and what blocks them. Uncertain facts go here as open questions, never as claims.

### `types.json` — the database

```json
{"database": "<Domain> types",
 "properties": {"Name": "title", "UI label": "text", "Code": "text", "API name": "text",
                "Stored or derived": "select", "Shows in": "multi_select", "Notes": "text", "Code link": "url"},
 "views": [{"name": "Only in the app", "filter": "..."}, {"name": "Only in the API", "filter": "..."}],
 "rows": [{"Name": "...", "...": "..."}]}
```

Adapt the properties to the domain; keep ≤ 9. One row per type. Every row cites code in `Code link`.

### Rules for this format

- The Cut rules from the single-page format apply to every file.
- No fact is written out twice: the second place links to the first.
- Every page has a one-line definition reachable without scrolling (the hub callout), and every product term on any page is either in the hub's terms strip or explained in the sentence where it appears.
- Diagrams are editable draw.io files made by `@domain-diagrammer` from briefs, published as their exported PNG (diagram embedded), placed with `<!-- diagram: <id> -->`. Never write Mermaid in this format.

### Diagram briefs (`diagrams/briefs/<id>.md`)

```
id: <id>                      type: overview | dataflow | scenario | states
question: <the question this diagram answers — becomes the title>
takeaway: <one sentence — becomes the subtitle>
surprise: <optional one line for a callout>
nodes: <id> | <product label> | <code name> | <kind: trigger|process|store|surface|external> | <lane> | <code link>
flows: <from> -> <to> | <data carried (noun)> | <step number if scenario> | confirmed|unconfirmed
lanes: <lane id> = <meaning>   (≤ 4)
```

`diagrams/registry.json` lists every node used in any brief with one fixed label, kind and lane, so the same thing looks the same everywhere.

## Diagram spec (for `@domain-diagrammer`)

Sources: Moody's Physics of Notations (cognitive effectiveness of visual notations), C4 model diagram checklist and dynamic diagrams, classic data-flow diagram rules.

### 1. Pick the diagram from the question (cognitive fit)

| Question | Type | Layout |
|---|---|---|
| What exists and how does it connect? | `overview` | columns left→right: where it starts · where it's stored · how it's read · where people see it; lanes top→bottom |
| Where does this data go? | `dataflow` | one data item, left→right through every process and store to every reader |
| What happens when …? | `scenario` | numbered steps left→right (or top→bottom if > 6 steps); one scenario only |
| What states can it be in? | `states` | states as rounded boxes, transitions labelled with the trigger |
| How do these differ? | — | not a diagram: a database view or table |

### 2. Symbol set — at most 5 kinds, each with one shape (graphic economy, ~6 is the human limit)

| Kind | Shape | Fill | Text |
|---|---|---|---|
| `trigger` (actor, job, webhook) | pill, fully rounded | `#0F172A` | white; product label bold |
| `process` (code that transforms) | rounded rect r=12 | `#FFFFFF`, stroke `#CBD5E1` | label as verb + noun; code name in mono 11.5px below |
| `store` (table, index, queue) | cylinder (rect with elliptical top) | lane tint | name in mono 13.5px bold, one plain line below |
| `surface` (screen, API, report) | rounded rect r=12 | lane colour, solid | white text |
| `external` (customer CRM, vendor) | rounded rect, dashed stroke | `#FFFFFF` | product name |

Shape encodes **kind**; colour encodes **lane** (which path/meaning). Never the reverse, never both on one variable (semiotic clarity).

### 3. Colour — lanes only, ≤ 4, always in a legend

Lane palette in order: blue `#2563EB` (tint `#EFF6FF`), purple `#7C3AED` (tint `#F5F3FF`), teal `#0D9488` (tint `#F0FDFA`), slate `#64748B` (tint `#F1F5F9`). Shared elements that feed several lanes: amber `#D97706` (tint `#FFFBEB`) with a small "SHARED" tag. Warning callout: red `#DC2626` on `#FEF2F2`. Background white. Text `#0F172A`, secondary `#475569`.

### 4. Arrows

- Every arrow has a label naming the **data it carries** (noun: "sent email row", "activity document"), reading in the arrow's direction. Verbs belong in process names.
- No store → store and no external → external arrows: a process always sits between them.
- Solid = confirmed in code; dashed = unconfirmed or not wired up end to end.
- Scenario diagrams: a numbered circle (r=12, lane colour, white number) on each step, in order.
- Labels sit on a white halo (`paint-order: stroke`, white stroke 4px) so lines never cross through text.

### 5. Layout and text

- Canvas 1280 px wide, height to fit; render at 2×. 40 px outer margin.
- Title (26px, weight 750) = the question. Subtitle (15px, secondary) = the takeaway. Optional callout bar at the bottom for the surprise.
- Product label first, code name second and smaller (dual coding).
- ≤ 15 nodes, ≤ 20 arrows. Bigger → split into one diagram per scenario (complexity management).
- Same node → same label, kind, lane colour and relative position on every diagram in the set (cognitive integration, via `registry.json`).
- Legend bottom-left: only the kinds and lanes used, plus solid/dashed.
- Font: `-apple-system, 'SF Pro Text', 'Helvetica Neue', Arial, sans-serif`; code: `'SF Mono', Menlo, monospace`. No emoji.

### 6. draw.io implementation

Diagrams are authored as `<id>.drawio` (draw.io XML), the source of truth. Tooling (installed in `~/.config/opencode/skills/`):

- **`drawio-skill`** (Agents365) — authoring rules, linter, draft-render inspection, edge fixes, PNG repair. Default style preset `~/.drawio-skill/styles/sourcewhale-docs.json` = this spec in preset form.
- **`drawio`** (official jgraph) — draw.io CLI facts: `--layout libavoid` edge routing, `--normalize`, export flags.

Pipeline (run by `@domain-diagrammer`; scripts are in `~/.config/opencode/skills/drawio-skill/scripts/`):

1. Author the XML with the style strings below.
2. Lint: `python3 validate.py <id>.drawio --score --strict` → score 0.
3. Arrows crossing boxes → set `exitX/exitY/entryX/entryY` on that edge or add waypoints; stacked edges → `edgeports.py`. (`--layout libavoid` re-routes without moving boxes, but only on draw.io Desktop builds that support it — 32.4.1 does not; try it, and fall back if it reports "Unknown layout".)
4. Draft render without embedding, inspect, fix (≤ 3 rounds): `drawio -x -f png --width 2560 -o <id>.draft.png <id>.drawio`.
5. Final export with the editable diagram embedded: `drawio -x -f png -e -s 2 -o <id>.png <id>.drawio`, then `python3 repair_png.py <id>.png`.

Anyone can reopen the `.drawio` or the PNG in draw.io (desktop app, app.diagrams.net, or the VS Code extension) and edit it. The draw.io MCP (`open_drawio_xml`, `get_page`, `set_page`) opens a diagram in the editor or edits pages of an existing file. Where this spec and the preset or the skills disagree, this spec wins.

Use these style strings (replace `<lane>` / `<tint>` with the lane colour and tint):

| Element | `style` |
|---|---|
| canvas | `<mxGraphModel grid="0" page="0" background="#FFFFFF">`; place cells on a 10 px grid, total width ≈ 1280 |
| title | `text;html=1;align=left;verticalAlign=top;fontSize=26;fontStyle=1;fontColor=#0F172A;fontFamily=Helvetica;` |
| subtitle | `text;html=1;align=left;verticalAlign=top;fontSize=15;fontColor=#475569;fontFamily=Helvetica;whiteSpace=wrap;` |
| lane band | a **container**: `rounded=1;arcSize=3;container=1;collapsible=0;fillColor=<tint>;strokeColor=<tint>;opacity=70;verticalAlign=top;align=left;spacingLeft=10;spacingTop=6;fontSize=11;fontStyle=1;fontColor=<lane>;` with the CAPS lane name as its own value. Nodes in that lane are its children (`parent="lane-<lane>"`, coordinates relative to the lane). Edges sit in the innermost container holding both ends (usually `parent="1"`). This keeps `validate.py` free of overlap warnings. |
| `trigger` | `rounded=1;arcSize=50;whiteSpace=wrap;html=1;fillColor=#0F172A;strokeColor=none;fontColor=#FFFFFF;fontStyle=1;fontSize=14;fontFamily=Helvetica;` |
| `process` | `rounded=1;arcSize=12;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#CBD5E1;fontColor=#0F172A;fontSize=14;fontFamily=Helvetica;align=left;spacingLeft=14;` value: `<b>Verb noun</b><br><font face="Menlo" style="font-size:11px" color="#475569">code_name</font>` |
| `store` | `shape=cylinder3;boundedLbl=1;size=8;whiteSpace=wrap;html=1;fillColor=<tint>;strokeColor=<lane>;fontColor=#0F172A;fontSize=12;fontFamily=Helvetica;` value: `<font face="Menlo"><b>table-name</b></font><br>plain line` |
| `surface` | `rounded=1;arcSize=12;whiteSpace=wrap;html=1;fillColor=<lane>;strokeColor=none;fontColor=#FFFFFF;fontStyle=1;fontSize=15;fontFamily=Helvetica;` |
| `external` | `rounded=1;arcSize=12;dashed=1;dashPattern=6 4;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#64748B;fontColor=#0F172A;fontSize=14;fontFamily=Helvetica;` |
| shared element | as its kind, but `<lane>` = `#D97706`, `<tint>` = `#FFFBEB`, plus the word `SHARED` inside its own label (e.g. a first line `<font style="font-size:10px" color="#B45309"><b>SHARED</b></font>`) — not a separate overlapping cell |
| arrow | `edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;strokeColor=<lane>;strokeWidth=2;endArrow=blockThin;endFill=1;fontSize=12;fontColor=#334155;labelBackgroundColor=#FFFFFF;fontFamily=Helvetica;` value = data carried; add `dashed=1;` when unconfirmed |
| step badge | `ellipse;whiteSpace=wrap;html=1;fillColor=<lane>;strokeColor=none;fontColor=#FFFFFF;fontStyle=1;fontSize=12;` 24×24, placed on the arrow or node it numbers |
| callout | `rounded=1;arcSize=10;whiteSpace=wrap;html=1;fillColor=#FEF2F2;strokeColor=none;fontColor=#7F1D1D;fontSize=13;align=left;spacingLeft=14;` |
| legend | small samples of each kind/lane/line style used, bottom-left, with `text` labels at fontSize=11 |

Every arrow sets `source` and `target` to node cell ids (so it stays attached when a human moves boxes). Use `exitX/exitY/entryX/entryY` or waypoints only when the default route crosses something.

### 7. Checks before an image ships (C4 checklist + this spec)

- [ ] Title is a question; subtitle is one sentence.
- [ ] ≤ 15 nodes; ≤ 5 kinds; ≤ 4 lanes; legend covers every kind, lane, and line style used.
- [ ] Every arrow labelled with data, direction matches.
- [ ] No text clipped, overlapping, or crossed by a line (look at the rendered PNG).
- [ ] Every node in the brief appears; nothing appears that isn't in the brief.
- [ ] Alt text written: one sentence + the node list.

## After delivery — teach mode

The dossier becomes the knowledge base (DeepWiki / Code Wiki chat):

- Answer follow-ups from the dossier first, and go back to the code when it's silent; cite either way. Update the dossier when an answer adds something new.
- Offer next steps: a guided walkthrough in reading order, a quiz on the glossary and lifecycle, an animated explainer of the hero use case (`@explainer-builder`), or an architecture review of the domain (`@feature-reviewer`).
