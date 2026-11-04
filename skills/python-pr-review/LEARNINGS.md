# Review Learnings

Read by every specialist and by the orchestrator's verification step. Each entry is a correction from the human reviewer: a kind of comment they deleted, rewrote, or rejected. Treat entries as "don't flag" (or "flag differently") rules.

The orchestrator appends an entry when the user rejects or rewrites a posted comment and says why. One entry per lesson; keep them short; newest last.

Format:

```
- YYYY-MM-DD · <repo or "all"> · <path glob or "*"> — Don't flag <X>. Why: <user's reason>. (from PR #<n>)
```

## Entries

<!-- none yet -->
- 2026-10-13 · all · * — Don't fill the full size-based comment budget when appending to a review on a PR that was already reviewed (second pass). Why: the author already has a first round of comments to process — post only the ~5 highest-impact findings. (from PR #15350)
- 2026-10-28 · all · * — Don't explain math findings (double-counting, proration, population changes) in abstract prose. Why: the human rewrote a blocking overlap comment because it wasn't understandable — the same finding with a worked example and a small timeline diagram landed instantly. (from PR #16123)
- 2026-10-28 · all · * — Don't cite code locations in comments as bare `file.py:line` names. Why: the human asked for actual GitHub URLs instead — use `[file.py:N](…/blob/<head-sha>/path#LN)` permalinks so each citation lands on the code in one click. (from PR #16123)
- 2026-10-28 · all · * — Don't post suggestion blocks much larger than the code they replace (e.g. a 60-line refactor behind a 44-line block). Why: the human called the pending comment "way too big" — keep suggestions compact: fewest new helpers, preserve the guards, land near the original's line count. (from PR #16135)
- 2026-10-29 · all · * — Don't pack precedent references into jargon-dense one-liners ("We've hit this shape before — X fixes it by PUTting the mapping and then reindexing…"). Why: the human asked "what does that mean?" — name the problem and spell the steps out in plain words, one idea per sentence; observed numbers beat abstract claims. (from PR #16027)
- 2026-10-29 · all · * — Don't paste whole-function suggestion blocks that re-state unchanged lines (guards, init) to reach a 1:1 line ratio. Why: the human's objection to the UTC/timezone comment was the 40-line code selection, not the prose — anchor the block to the changed sub-range, or skip the block and point at an in-file sibling pattern with a short all-changed-lines snippet. (from PR #16250)
- 2026-11-02 · all · * — Don't name UI elements by code-location shorthand ("Header Clear", "footer Apply") in comments. Why: the human asked "what does Header Clear mean?" — describe the button as the user sees it ("the Clear button next to the modal title"). (from PR #16181)
- 2026-11-02 · all · * — Don't describe a UI repro through a consumer path without verifying the consumer actually renders (component has the branch ≠ any screen uses it). Why: the human went looking for the "Candidates table date cell" Clear button and found nothing — the branch exists but no table renders date cells; the finding was real but latent, and the comment read as wrong. (from PR #16181)
