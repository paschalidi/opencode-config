# High-Signal Contract

Shared by every specialist and by the orchestrator's verification step. A wrong or pedantic comment costs more than a missed nit: it trains the author to ignore the review.

## A finding qualifies only if

1. It is **introduced or touched by this PR** — the cited line is an added/modified line, or the PR changed what it depends on.
2. It is **concrete**: you can say *what goes wrong, when* — "When X, the code does Y, so Z." Name the input, state, or caller from the change map.
3. It is **grounded**: you read the cited code and, when the claim depends on callers, data, or siblings, you checked them (the change map lists them).
4. It has a **concrete fix** — replacement code or a precise instruction.

## Never flag

- **Pre-existing issues** in lines the PR didn't touch. (Exception: the PR's change makes a pre-existing issue newly reachable or worse — say so explicitly.)
- **Hypotheticals** that need inputs or state you can't point to ("if someone passes a negative id someday…").
- **Anything a linter/formatter/type checker catches**, whether or not the repo runs one.
- **Subjective preferences**: naming taste, "could be cleaner", alternative-but-equal structure.
- **Code that follows an explicit repo rule.** If the repo's `AGENTS.md` / `CLAUDE.md` / `CONTRIBUTING` explicitly prescribes the pattern (e.g. SourceWhale: inline table-name literals, no docstrings/type hints, `logging.*` directly, no unnecessary early returns, comprehensions over loops), it is correct here — even if a generic checklist item says otherwise.
- **Anything listed in [LEARNINGS.md](LEARNINGS.md)** as "don't flag".

## Convention precedence

1. **Personal standards files** — `~/.config/opencode/instructions/python-testing-standards.md`, `~/.config/opencode/instructions/logging-standards.md`. These win over the repo when they conflict.
2. **Explicit repo rules** — `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING`, ADRs. These win over generic checklist advice.
3. **Generic checklist items** (PYTHON-CHECKLIST idioms, magic-strings, guard clauses, …) — apply only where 1 and 2 are silent.

## Rule citations

A finding that rests on a rule (D-rule, R-rule, repo rule, personal standard) quotes the rule's exact text and its source file. If the rule can't be quoted, the finding stands on its concrete failure scenario alone — or it's dropped.

## Score every finding (0–10)

Give each finding a score and a one-line reason:

| Score | Meaning |
|---|---|
| 9–10 | Certain and consequential: verified failure path, data loss, security, broken caller |
| 7–8 | Very likely real; clear failure scenario; fix is obvious |
| 5–6 | Real but minor, or real but needs one assumption you couldn't check |
| 0–4 | Speculative, stylistic, or low impact — **don't report it** |

Report only findings scoring **≥ 5**. The orchestrator cuts the non-Critical budget by score, highest first.

## Declined to judge

End every report with one line listing what you deliberately did **not** assess and why (e.g. "Declined to judge: stream retry behaviour — consumer not in diff or change map"). Empty → `Declined to judge: none`. This separates "checked, found nothing" from "didn't check".
