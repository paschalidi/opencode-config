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

## Reporting

For each finding: **files** involved, **problem** (the friction), **solution** (plain English), **benefits** in terms of locality + leverage + how tests would improve.

Architecture findings default to **Important**; only **Critical** when they create data-loss, security, or correctness risk. Architecture is the axis most prone to taste — every finding must cite the deletion test, shallowness, a layering rule, or duplication. No vague "this could be cleaner".
