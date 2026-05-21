---
name: review-feature-architecture
description: Review and improve the architecture of a specific feature the user points to. Finds deepening opportunities scoped to that feature, informed by the domain language in CONTEXT.md and the decisions in docs/adr/. Use when the user asks to review, critique, or improve a specific feature's architecture — not the whole codebase.
---

# Review Feature Architecture

Surface architectural friction within a **single feature** and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. Scoped to the feature the user names; do not wander into unrelated areas.

## Glossary

Use these terms exactly in every suggestion. Consistent language is the point — don't drift into "component," "service," "API," or "boundary." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Use this, not "boundary.")
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

Key principles (see [LANGUAGE.md](LANGUAGE.md) for the full list):

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.**
- **One adapter = hypothetical seam. Two adapters = real seam.**

This skill is _informed_ by the project's domain model. The domain language gives names to good seams; ADRs record decisions the skill should not re-litigate.

## Process

### 0. Identify the feature

Ask the user which feature to review if not already clear. A feature is identified by one or more of:

- A directory or set of files ("the `patient_comms` app")
- A ticket or epic name ("OPH-336")
- A domain concept ("the action request resolution flow")
- A recent branch or PR ("the work on `cp/OPH-336/...`")

Once identified, **map the feature's footprint**: find every file, module, and cross-app touchpoint that belongs to it. Present a short summary back to the user: _"Here's what I see as the feature boundary — does this look right, or should I include/exclude anything?"_ Wait for confirmation before proceeding.

### 1. Explore (scoped)

Read the project's domain glossary and any ADRs in the area you're touching first.

Then walk the feature's modules. Stay within the feature boundary confirmed in Step 0 — only follow cross-feature dependencies far enough to understand the seam. Note where you experience friction:

- Where does understanding one concept require bouncing between many small modules within this feature?
- Where are modules **shallow** — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules within the feature leak across their seams?
- Which parts of the feature are untested, or hard to test through their current interface?
- Where does the feature's interface to the rest of the codebase create unnecessary coupling?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Present candidates

Present a numbered list of deepening opportunities **within the feature**. For each candidate:

- **Files** — which files/modules are involved
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and also in how tests would improve

**Use CONTEXT.md vocabulary for the domain, and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.** If `CONTEXT.md` defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service."

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly (e.g. _"contradicts ADR-0007 — but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.

Do NOT propose interfaces yet. Ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept not in `CONTEXT.md`?** Add the term to `CONTEXT.md`. Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `CONTEXT.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing — skip ephemeral reasons ("not worth it right now") and self-evident ones.
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
