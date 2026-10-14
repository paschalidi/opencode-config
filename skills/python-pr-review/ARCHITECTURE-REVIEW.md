# Architecture Review (PR-scoped)

The contract for `@review-python-architecture`. A deep-module review of the diff, in the spirit of the `review-feature-architecture` skill but scoped to what the PR changes.

## Vocabulary — use these terms exactly

**Module** — anything with an interface and an implementation (function, class, package, slice).
**Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the signature.
**Implementation** — the code inside.
**Depth** — leverage at the interface: a lot of behaviour behind a small interface. Shallow = interface nearly as complex as the implementation.
**Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Not "boundary".)
**Adapter** — a concrete thing satisfying an interface at a seam.
**Leverage** — what callers get from depth. **Locality** — what maintainers get: change, bugs, knowledge concentrated in one place.

Full reference: `~/.config/opencode/skills/review-feature-architecture/LANGUAGE.md`. Never say "component", "service" (except the services layer below), "API", or "boundary" in findings.

## Core tests — apply to every new/changed module in the diff

- [ ] **Deletion test**: if deleting this module makes complexity vanish, it was a pass-through. If complexity reappears across N callers, it earns its keep. Pass-throughs → merge into the caller or deepen.
- [ ] **Shallowness**: interface nearly as complex as the implementation (thin wrapper forwarding 1:1 to a collaborator) → merge or deepen.
- [ ] **The interface is the test surface**: do new tests cross the same seam callers use, or poke past it (private methods, internals)?
- [ ] **One adapter = hypothetical seam**: a new interface/abstract seam with exactly one implementation and no second caller in sight → flag unless variation is imminent. Don't pay for seams nothing varies across.
- [ ] **Leaky seam**: callers must know implementation facts (ordering constraints, error modes, required config) that the interface doesn't express → move the fact into the interface or hide it.

## Layering — personal standard: models / services / handlers / http_api

- [ ] Business logic in the HTTP layer (views) → belongs in services.
- [ ] HTTP concerns (status codes, request parsing, response shaping) inside services/handlers → belongs in http_api.
- [ ] DB queries scattered in http/handlers instead of behind the model/service seam.
- [ ] A layer reaching past its neighbor (http importing handler internals, models importing services).

## Change-shape smells

- [ ] Logic duplicated across the diff that one deep module would own (locality: fix once, fixed everywhere).
- [ ] Feature leaking across app/module boundaries — new imports crossing modules that never touched before, without a seam.
- [ ] Circular dependencies introduced or hidden via function-level imports.
- [ ] New names clashing with domain vocabulary — if the repo has `CONTEXT.md`, names should come from it.
- [ ] Config/constants sprawled across files instead of one home.
- [ ] New module contradicting an ADR (`docs/adr/`) → surface as "contradicts ADR-XXXX" — flag it, don't re-litigate.

## System lenses — only when the change map rates impact M or H

Use the change map (callers, entry points, data touched, stream consumers, client callers, siblings) to look past the module at the system. Write each finding as a scenario: **"When X, the system does Y, because Z — so we risk W."** No scenario → no finding.

- [ ] **Placement & dependency direction** — code used by one entry point lives beside it; shared code lives in the shared layer (SourceWhale: one Lambda → beside the Lambda, shared → `backend/sdk`). New imports between sibling Lambdas/apps, or shared code importing from an entry point → flag.
- [ ] **Data flow end to end** — follow every written field through write → stream consumer → denormalized copy / index → readers. Who owns each copy? How does it get repaired if it drifts? A new field with no reader, or a reader with no writer → flag.
- [ ] **Failure & consistency** — partial failure midway through a multi-write, stream retries re-delivering the same event (idempotent?), out-of-order events, timeouts (SourceWhale API Gateway: 29s). Name the failure and what state it leaves behind.
- [ ] **Reversibility** — the change map's door call. One-way doors (new data shape, removed field, public contract change, backend change that frontend/extension must ship with) need a rollout/backfill/rollback story. Missing → flag.
- [ ] **Sibling consistency** — compare against the change map's siblings: pagination, auth/ownership checks, projection, error shape, retry wiring. Diverging without a reason → flag, citing the sibling.
- [ ] **End-to-end wiring** — a new route with no client caller, or a client call with no route, means the feature isn't actually connected. Flag as a question, not an issue.
- [ ] **Cost & scale** — per-item calls in loops over unbounded sets, fan-out that grows with tenants/records, full scans behind user-facing paths.

## Reporting

For each finding: **files** involved, **problem** (the friction), **solution** (plain English), **benefits** in terms of locality + leverage + how tests would improve. System-lens findings use the scenario sentence as the problem.

Architecture findings default to **Important**; only **Critical** when they create data-loss, security, or correctness risk. Architecture is the axis most prone to taste — every finding must cite the deletion test, shallowness, a layering rule, duplication, or a system-lens scenario. No vague "this could be cleaner".
