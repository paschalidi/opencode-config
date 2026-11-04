# Duplication Review

The contract for the sixth review axis. A mechanical hunt for code in the diff that already exists elsewhere in the repo. This axis does NOT judge design quality — only "this already exists".

## Inputs

Patch path, local repo path (PR head). Read HIGH-SIGNAL.md and LEARNINGS.md first.

## Method

1. **Extract candidates from the diff**: every added function/helper; predicate, loop, or formula bodies >= 8 lines; `.source([...])` field lists; YAML resource blocks; component structure (Svelte). Skip one-liners and idioms every file contains (guard clauses, try/except shapes, standard imports).
2. **For each candidate, search the WHOLE repo, not just neighbors:**
   - Exact fragments: `rg -F "<distinctive line or field list>"`
   - Signatures: `rg "def <name>"` across the repo (reimplemented under the same name elsewhere?)
   - Distinctive literals: field-name lists, key names, error strings.
3. **Read each hit +/- 15 lines.** Classify honestly:
   - **Exact copy** — same behavior, same shape.
   - **Near copy with drift** — same intent, different details (the worst: they silently diverge; cite the differing lines).
   - **Coincidental** — similar lines, different behavior → discard, don't report.
4. **Recommend one of:** reuse the existing module at its current seam / move the shared logic to the natural seam (name it) / leave alone (say why, e.g. deliberate per-Lambda placement per AGENTS.md).

## Never flag

Vendored/generated code, test fixtures, trivial idioms, anything HIGH-SIGNAL.md or LEARNINGS.md excludes, code the PR itself deleted-and-moved.

## Reporting

Per finding: severity (Important; Critical only if drifted copies already disagree about behavior), file:line in the diff, file:line of the existing twin, classification (copy / drifted copy), the fix, drift risk in one sentence. Cap at the 5 strongest; list withheld counts. Keep prose ELI5-short; suggestion blocks compact (changed lines only, never re-stating unchanged code).

## Speed mandate

The fastest axis. Searches and targeted reads only — never read whole files cover-to-cover, never sweep directories by hand. Runs via `@explore` with this checklist as the brief.
