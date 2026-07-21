# Plan: <project title>

## Approach

One paragraph. The overall strategy for delivering the Outcome from the
project definition. Why this shape of plan rather than another. Anchors the
rest of the document.

## Modules

For each module to build or modify:

### <module name>

- **Responsibility:** what this module is responsible for, in one sentence.
- **Interface (prose):** describe inputs, outputs, side effects. No code —
  code rots.
- **Status:** new | modified
- **Deep-module rationale:** why this is worth carving out as its own
  module. What invariants does it hide? What would change if it didn't
  exist?

(Repeat per module.)

## Cross-cutting decisions

Decisions that affect multiple slices. Each is one paragraph: the decision,
followed by the rationale. Examples: schema choices, framework picks, API
contracts, test-strategy stance, error-handling conventions. New ADRs may be
drafted from these after the plan is finalized.

- **<Decision title>:** <decision and rationale.>

(Repeat.)

## Slice 0 — Scaffolding / baseline

The infra, types, plumbing, and prep work that must land before any
functional slice can. Often AFK.

- **User-visible behavior:** None — internal prep only. Describe the
  visible-from-the-codebase delta (e.g., new package, new types, new
  migration).
- **Modules touched:** <list>
- **Acceptance criteria:**
  - <testable bullet>
- **HITL / AFK:** AFK (default). Reason: <one line>.
- **Depends on:** —
- **Open questions:** <list, or "none">

## Slice 1 — <title>

- **User-visible behavior:** <one paragraph: what's true after this slice
  ships, from a user's or operator's perspective.>
- **Modules touched:** <list, referencing Modules section. Single-module
  entries are a smell — slices should be vertical.>
- **Acceptance criteria:**
  - <testable bullet>
  - <testable bullet>
- **HITL / AFK:** <pick> Reason: <one line>.
- **Depends on:** 0
- **Open questions:** <list, or "none">

## Slice 2 — <title>

(Same shape as Slice 1.)

…continue for slices 3..N…

## Slice N+1 — Cleanup / backfill

Remove shims, delete dead code, backfill data, migrate stragglers, drop
feature flags. Usually AFK.

- **User-visible behavior:** <e.g., legacy endpoints removed; flag removed;
  data backfilled.>
- **Modules touched:** <list>
- **Acceptance criteria:**
  - <testable bullet>
- **HITL / AFK:** AFK (default).
- **Depends on:** 1, 2, …, N (every functional slice)
- **Open questions:** <list, or "none">

## Slice B — Bugfixes (parking lot)

Open-ended. Bugs surfaced during implementation get appended here as they're
found.

- **HITL / AFK:** AFK.
- **Depends on:** —
- **Items:**
  - (none yet)

## Open questions (cross-cutting)

Unknowns not tied to a single slice. Each entry names the slice(s) it blocks.

- <question> — blocks: <slice(s)>

## Out of scope for this plan

Things considered during planning and explicitly deferred to a later plan.
(Different from the project definition's Out of scope, which is project-wide
forever.)

- <item>
