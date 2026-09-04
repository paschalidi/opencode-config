---
name: full-cycle
description: Run the full ticket-to-PR pipeline as a multi-agent loop. Parent orchestrates; subagents implement, doc, review (parallel standards+spec axes), qa verify, and fix per PR-slice. Use when user says "run full cycle", "run pipeline", "full-cycle", or "ship it".
---

# Full Cycle Pipeline

Multi-agent. Parent thin. Subagents do work. Wait for user at gates.

## Pipeline

### 1. Read ticket
User gives ticket URL/id + base branch. Fetch + summarize: goal, AC, scope. Confirm. Derive **ticket-id**: tracker key if one exists, else kebab-case slug from the ticket title.

### 2. Plan + grill → `@ticket-planner`
Subagent grills via `@grill-me`, writes `plans/<ticket-id>.md`. User signs off.

### 2b. Create branch
`git checkout -b cp/<ticket-id>/<short-slug>` from base branch. Pattern: `cp/1234/add-list-endpoint`; when ticket-id is already a slug, `cp/<slug>` alone.
- Verify `.gitignore` excludes `plans/`. Never let plan files leak into commits.

### 3. Per PR-slice (loop)

| Step | Who | What |
|---|---|---|
| 3a | `@implementer` | Implement slice. Run tests/typecheck. Stage files. Return diff stat. Parent saves `task_id`. |
| 3b | `@docs-writer` | Inline docs on new public API. |
| 3c | parent | `git diff --cached` + glob standards files. |
| 3d | `@standards-reviewer` ‖ `@spec-reviewer` | Parallel fan-out. One message, two Task calls. Read-only. |
| 3e | parent | Print both reports. User picks: proceed / fix all / fix subset / reject. |
| 3f | `@implementer` (resume `task_id`) | Apply selected fixes. Re-test. Re-stage. → loop 3d–3e. |
| 3g | parent | Commit. Conventional Commits. `<type>(<scope>): <imperative>` — `<ticket-id> – ` prefix only when a ticket key exists. Tick the slice checkbox in the plan. |
| 3h | parent | Ask user → next slice. |

### 4. Architecture pass → `@feature-reviewer`
After last slice. Default on, optional. User picks deepening fixes. `@implementer` applies (fresh task_id). Commit each.

### 4b. QA pass → `@qa-verifier`
After architecture pass. Orchestrator asks user for target URL. If none, user explicitly skips. Otherwise invoke `@qa-verifier` (fresh task, read-only). Print report verbatim under `## QA`. One-line summary: totals per severity. User gate: proceed / fix all / fix subset / reject. Fixes via `@implementer` resume `task_id` from last slice, then re-run QA. Loop until user says proceed.

### 5. Open draft PR → `@open-draft-pr` skill
Push. Draft PR vs base from step 1. `review` label.

### 6. Apply human review → `@review-applier`
After PR gets review comments. Subagent applies every comment as one commit (title only), 👍 each. No comment left behind.

## Hard rules

- Never skip grilling. User signs off plan.
- Never skip per-slice review. No auto-pass.
- Never auto-apply fixes — user picks.
- Subagents never `git commit` / `git push`. Parent owns git state changes.
- Reviewers always fresh task, read-only. `@implementer` resumes `task_id` for fix mode.
- One slice = one commit (+ optional fix commits).
- PR always draft, `review` label, base = user-specified.
- `@review-applier` works post-PR, after human review. One comment = one commit. 👍 every applied comment.
- **Every commit in this pipeline uses Conventional Commits. No exceptions.**
- **Branch name always `cp/<ticket-id>/<kebab-case-slug>`** — or `cp/<slug>` when ticket-id is a slug. No exceptions.
- **Never commit plan files.** `plans/<ticket-id>.md` stays local, unstaged, untracked.
- **Progress lives in the plan**: parent ticks each slice's checkbox right after its commit. Unticked = not done = the resume point. Never pre-tick.
- Never skip the user gate after QA findings. QA is skippable only with explicit user consent (no test target).
- QA artifacts (screenshots under plans/qa/) are never committed.

**Caveman Ultra mode ACTIVE every response.**
