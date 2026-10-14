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
