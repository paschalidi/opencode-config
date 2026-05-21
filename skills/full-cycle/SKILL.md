---
name: full-cycle
description: Run the full ticket-to-PR pipeline: read ticket → grill+plan → code → docs → review → fix → open draft PR. Use when user says "run full cycle", "run pipeline", "full-cycle", or "ship it".
---

# Full Cycle Pipeline

Run steps in order. Wait for user input at interactive gates.

## Pipeline

### 1. Read ticket
User provides ticket URL or key. Fetch and summarize: goal, acceptance criteria, scope. Confirm with user.

### 2. Plan + grill
Interview user about approach using `@grill-me` mindset — walk decision branches, recommend answers, explore codebase to resolve questions. Write plan to `plans/<ticket-key>.md` with PR breakdown.

### 3. Write code
Implement per plan. One PR-slice at a time. After each slice, commit with Conventional Commits message. Ask user before moving to next slice.

### 4. Write docs
Run `@doc-weaver` workflow on changed files. Only public new code. No implementation detail, no cross-refs. Google-style (Python) or JSDoc (JS/TS).

### 5. Review
Run `@feature-reviewer` workflow. Present deepening candidates to user. User picks what to fix.

### 6. Fix
Apply selected fixes from review. One at a time. Commit each fix.

### 7. Open draft PR
Use `@open-draft-pr` workflow: commit staged changes, push, open draft PR against base branch from step 1. PR is always draft with `review` label.

## Hard rules
- Never skip the grilling step. User must sign off on plan before writing code.
- Never skip the review step. User must pick fixes before PR.
- Each code slice is a separate commit.
- PR is always draft with `review` label.

**Caveman Ultra mode ACTIVE every response.**
