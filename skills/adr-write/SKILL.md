---
name: adr-write
description: Write an architecture decision record (ADR/RFC) for the SourceWhale codebase after a thorough, cited lookup of the code. Produces one markdown file in the team's RFC format — header with Status/Date/Scope, plain-language Context with GitHub links, product-facing Real world scenarios in When/and/then form, Suggested Solution, Done looks like, What needs to change, Consequences and risks, Alternatives considered. Use when asked to write, draft, rewrite or reformat an ADR or RFC, or to turn an improvement idea into a decision record.
---

# ADR Write

`@adr-write` leads. It scopes the decision, maps the code, sends `@domain-scout` workers to read the code in parallel, verifies every claim against `origin/prod`, writes one ADR file in the format below, then re-checks the written file against the code. The gold example is `domain-dossiers/backend-architecture/adr-01-no-silent-background-failures.md` on the `cp/dossiers` branch: read it before writing.

## Principles

1. **Verified or it doesn't go in.** Every count, line number and behaviour comes from a command you ran or a file you read on `origin/prod`. Run `git fetch origin prod` first, then read with `git show origin/prod:<path>` and `git show origin/prod:<path> | grep -n`. Count with commands (`grep -c`, `awk`), never by eye. A claim you could not check is marked "(not re-checked)" or left out.
2. **Two audiences, kept apart.** Context and What needs to change are for developers: code names and links. Real world scenarios are for product people: technical names (DLQ, OpenSearch, DynamoDB, Sentry, Lambda) are fine, code never is.
3. **Plain words.** Short sentences (under 25 words). One idea per bullet. No "easy", "simple", "quick", "straightforward", "trivial" or "just". No invented numbers.
4. **Never lose context.** When editing an existing ADR, move or reword content; never drop a fact, link or decision unless the user asks. Change only the section the user asked about.
5. **Check for overlap.** Before writing, look for existing ADRs and RFCs on the same topic: `domain-dossiers/**/adr-*.md`, `docs/adr/`, and the Notion Improvements backlog (`https://app.notion.com/p/3f2eb382802680bba588de0bc706b1b3`) when the Notion tools are available. If one exists, build on it and say so in Scope instead of proposing a competing design.

## Code links

Every code reference in Context, Suggested Solution and What needs to change is a GitHub link to `prod` with line numbers:

```markdown
[`sqs-alarms.yaml`](https://github.com/sourcewhale/app/blob/prod/backend/stacks/nested/sqs-alarms.yaml#L10-L24)
[`streams.py` – `DdbStream`](https://github.com/sourcewhale/app/blob/prod/backend/sdk/aws/storage/ddb/streams.py#L40)
```

- Paths are repo-relative (they start with `backend/`, `frontend/`, …).
- Line numbers come from `origin/prod`, not the local branch.
- Link the file or symbol name, not the whole sentence.
- Never put a link or code name in Real world scenarios.

## The format

File: `domain-dossiers/<topic>/adr-NN-<kebab-slug>.md` (NN = next free number in that folder), unless the user names another path.

```markdown
# <Area> – <Short name> (RFC)

**Status:** 💡 Suggestion — draft for discussion, not yet agreed

**Date:** <YYYY-MM-DD, today>

**Scope:** <What this covers, in one or two sentences. Then what is out of scope, and where that lives (another ADR/RFC, linked).>

---

## Context

- **<One-line summary in plain words.>** <One or two short sentences explaining it, with code links.>
- **<…>** <…>

## Real world scenarios

1. **<What the user or customer experiences, as a headline.>**
    - **When** <trigger in the product or the system>,
    - **and** <the thing that fails>,
    - **and** <why nothing catches it> (optional),
    - **then** <what the system does>,
    - **and then** <what the user/customer sees, and how we find out>.
2. **<…>**
    - **When** …

## Suggested Solution

<A short paragraph: the target design, in plain words.>

<Optional: one small YAML or Python sketch.>

## Done looks like

- <Measurable outcome: a count, a time, a check that passes.>
- <…>

## What needs to change

1. **<Bold name of the change>**
    - File: [`<file>` – `<symbol>`](https://github.com/sourcewhale/app/blob/prod/<path>#L<a>-L<b>)
    - <What changes.>
    - <Why, or what it replaces.>
2. **<…>**

## Consequences and risks

- <Risk, then what we do about it.>

## Alternatives considered

- **<Option.>** Rejected: <reason>. (or Deferred: <reason>.)
```

### Section rules

| Section | Rule |
| --- | --- |
| Title | `<Area> – <Short name> (RFC)`. Area is one word: Reliability, Security, Observability, Deploys, Data, Performance, CI… |
| Status / Date / Scope | Exactly as above. Scope always names what is out of scope. Then a `---` line. |
| Context | Plain words (explain like to a new joiner). Each bullet starts with a bold one-line summary. Code links on every code reference. 3–6 bullets. |
| Real world scenarios | **At least 2.** For product people. When / and / then / and then. Technical names allowed, code never: no function names, file paths, config keys, links or snippets. Each scenario must be traced in the code first. |
| Suggested Solution | Called exactly this (not "Decision"). Target state, not steps. |
| Done looks like | Directly under Suggested Solution and above What needs to change. Measurable. |
| What needs to change | Called exactly this (not "First PR", not phases). Numbered changes, each with a bold name, a `File:` line with a link, and bullets. Cover every point the Suggested Solution promises. |
| Consequences and risks | Plain bullets, not a table. |
| Alternatives considered | Bold option, then Rejected/Deferred and the reason. |

Sections not listed above (phases, Risk/Mitigation tables, "First PR", "Decision", "Failure scenario") are not used.

## Lookup lenses

Give each `@domain-scout` one lens. Ask it to write cited findings to `domain-dossiers/<topic>/research/adr-NN-<lens>.md` and return the path plus a 3-line summary.

| Lens | What to find |
| --- | --- |
| L1 Current behaviour | The code path end to end: entry point, handler, SDK helpers, tables, queues. Who calls what. |
| L2 Failure paths | Every `try/except`, `catchExceptions=True`, retry, timeout, sleep, DLQ, `MaximumRetryAttempts`, `EventInvokeConfig`. What happens on error, step by step. |
| L3 Infrastructure | SAM/CloudFormation in `backend/stacks/*.yaml`: functions, event sources, batch sizes, concurrency, alarms, IAM. Counts by command. |
| L4 Blast radius | Callers and consumers (`get_function_usage` if available, else `rg`), how many functions/tables/teams are touched, user-facing features that depend on it (check `frontend/web-app/src`). |
| L5 History | `git log --since=6.months` on the files, related PRs, earlier attempts, existing ADRs/RFCs. |

Small decisions can merge lenses; never skip L2 when the ADR is about failure.

## Final verification

The written ADR is a claim, not a fact, even when the scouts and the earlier verify step agreed. Check the file as written, top to bottom, against `origin/prod`.

1. **List the claims** in `domain-dossiers/<topic>/research/adr-NN-verification.md`, one row each: claim, check you ran, verdict. A claim is any count, link, code name, or statement about how the code or product behaves today.
2. **Check each claim**:
   - **Link:** the path exists, and the cited lines ±15 show what the sentence says (`git show origin/prod:<path> | sed -n '<a>,<b>p'`).
   - **Count:** re-run the command. The number matches the text.
   - **Behaviour:** the code does this. If it depends on callers or config, run a targeted `rg` to confirm.
   - **Real world scenario:** every step happens in the code. A "nothing catches it" step needs a search for the catcher (`try/except`, retry, DLQ, alarm) that comes back empty.
   - **What needs to change:** the target exists, and the change is not already on `origin/prod`.
   - **Alternatives and Scope:** rejection reasons about the code are true, and linked ADRs/RFCs say what Scope says they do.
3. **Give a verdict**:
   - **CONFIRMED** → keep. If the claim is right but the line numbers are off, fix them.
   - **REJECTED** → correct or remove the claim, and fix anything that relied on it (scenario, Suggested Solution, Done looks like, What needs to change). Re-check whatever you rewrote.
   - **UNSURE** → mark "(not re-checked)", or drop it if it is low impact.
4. **Stop and tell the user** if a rejected claim removes the problem the ADR exists to solve. Do not write around it.

For a one-section edit, check only that section and what it depends on. Report wrong claims elsewhere in the file without changing them.

## Self-review before handing over

- [ ] Final verification done: every claim has a verdict in the research file, and no REJECTED claim is left in the ADR.
- [ ] Header matches: title form, Status, Date, Scope with out-of-scope, `---`.
- [ ] Sections in order: Context, Real world scenarios, Suggested Solution, Done looks like, What needs to change, Consequences and risks, Alternatives considered.
- [ ] Every code reference outside Real world scenarios is a GitHub `prod` link with line numbers, re-checked on `origin/prod`.
- [ ] Real world scenarios: at least 2, When/and/then/and then, zero code.
- [ ] Every number was produced by a command; anything unchecked says so.
- [ ] Every Suggested Solution promise has a matching item in What needs to change.
- [ ] No banned words; sentences short.
- [ ] Overlap with existing ADRs/RFCs checked and stated in Scope.
