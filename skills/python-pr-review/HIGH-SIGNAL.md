# High-Signal Contract

Shared by every specialist and by the orchestrator's verification step. A wrong or pedantic comment costs more than a missed nit: it trains the author to ignore the review.

## A finding qualifies only if

1. It is **introduced or touched by this PR** — the cited line is an added/modified line, or the PR changed what it depends on.
2. It is **concrete**: you can say *what goes wrong, when* — "When X, the code does Y, so Z." Name the input, state, or caller.
3. It is **grounded**: you read the cited code and, when the claim depends on callers, data, or siblings, you checked them.
4. It has a **concrete fix** — replacement code or a precise instruction.

## Never flag

- **Pre-existing issues** in lines the PR didn't touch. (Exception: the PR's change makes a pre-existing issue newly reachable or worse — say so explicitly.)
- **Hypotheticals** that need inputs or state you can't point to ("if someone passes a negative id someday…").
- **Anything a linter/formatter/type checker catches**, whether or not the repo runs one.
- **Subjective preferences**: naming taste, "could be cleaner", alternative-but-equal structure.
- **Code that follows an explicit repo rule.** If the repo's `AGENTS.md` / `CLAUDE.md` / `CONTRIBUTING` explicitly prescribes the pattern (e.g. SourceWhale: inline table-name literals, no docstrings/type hints, `logging.*` directly, no unnecessary early returns, comprehensions over loops), it is correct here — even if a generic checklist item says otherwise. (Personal standards files still win: testing and logging f-strings.)
- **Missing tests.** The codebase has no test suite. Only when the PR adds or changes tests may a comment suggest more cases — and test comments are always non-blocking.
- **Anything listed in [LEARNINGS.md](LEARNINGS.md)** as "don't flag".

## Rule citations

A finding that rests on a rule (D-rule, R-rule, repo rule, personal standard) quotes the rule's exact text and its source file. If the rule can't be quoted, the finding stands on its concrete failure scenario alone — or it's dropped.
