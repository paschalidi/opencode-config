---
name: ticket-planner
description: Read a ticket, stress-test the plan via @grill-me, and write a plan document split into PR-sized chunks. Use when the user says "plan this ticket", "create a plan", "ticket planner", "@ticket-planner", or when starting work on a ticket that needs a written plan.
---

# Ticket Planner

## Pipeline

### 1. Read ticket
- Fetch ticket via Jira MCP or parse provided URL/key.
- Summarize: goal, acceptance criteria, scope, linked PRs, comments.
- Confirm summary with user before proceeding.

### 2. Explore codebase
- Search for files mentioned in ticket (schemas, services, tasks, handlers).
- Read key files to understand current implementation.
- Note any existing patterns that must be preserved.

### 3. Stress-test plan via @grill-me
- Load the `@grill-me` skill and enter grilling mode.
- Walk decision branches with user: approach, edge cases, idempotency, ordering, rollback, testing strategy.
- Recommend answers where patterns are clear from codebase.
- Do not proceed to code until user signs off.

### 4. Write plan document
- Write plan to `plans/<ticket-key>.md`.
- Split work into PR-sized slices (one slice = one commit).
- Each slice must have: goal, files touched, test strategy, rollback plan.
- Include acceptance criteria checklist at end.

## Hard rules
- Never skip the grilling step. User must sign off on plan before writing code.
- One PR-slice per commit.
- Plan document must exist before any code is written.
