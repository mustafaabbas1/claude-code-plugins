# plan — example

One end-to-end walkthrough.

## Planning `sp-api-listings-ingestion`

**Background.** `/sdlc:define` already produced
`projects/sp-api-listings-ingestion/definition.md` with the Outcome "every
seller's active listings are available in a queryable store within 24 hours,"
glossary terms (listing, active listing, marketplace, seller, ingest run),
and the principle "prefer batch pulls over per-listing polling."

**User invokes:** `/sdlc:plan sp-api-listings-ingestion`

### Skill behavior

1. **Resolve target** — exact folder match, no ambiguity.

2. **Gather context** — reads `definition.md`; no existing `plan.md` (fresh
   plan). No ADRs yet. `CLAUDE.md` confirms the monorepo layout (`etl/`
   Python, `packages/db` Go, `packages/types` TS). Bounded codebase pass on
   glossary terms: `etl/sp-api/` is an empty stub package, `packages/types/
   sp-api/` has placeholder types, `packages/db/schemas/` has no listings
   schema yet.

3. **Draft** `plan.md`:
   - **Approach:** batch-pull all sellers' listings once daily via a
     scheduled ETL job, land normalized rows in a new DB table, expose
     nothing else yet (no API layer this round).
   - **Modules:** `etl/sp-api/listings/` (new, SP-API client + parser),
     `packages/types/sp-api/listings.ts` (new, shared types),
     `packages/db/schemas/listings.ts` (new, table schema + client methods).
   - **Cross-cutting decisions:** "single-table schema keyed by
     seller+marketplace+asin+date" with rationale.
   - **Slice 0:** scaffold the three new modules with types and empty
     function stubs; AFK.
   - **Slice 1:** "Ingest one seller's active listings" — pulls + parses +
     lands rows; HITL because it's the first real SP-API call and needs a
     human to confirm real rate-limit behavior.
   - **Slice 2:** "Batch across all sellers with idempotent reruns" —
     depends on 1.
   - **Slice N+1 (3):** cleanup — remove the placeholder types, backfill
     historical listings once.
   - **Slice B:** empty, present.

4. **Per-slice grill:**
   - Slice 0: "AFK, agree?" → yes.
   - Slice 1: "HITL because first real SP-API call — agree, or is sandbox
     access good enough to go AFK?" → user says AFK is fine, sandbox
     credentials exist. Updated.
   - Slice 2: "Depends on 1 — agree the idempotency AC should be 'rerunning
     ingest twice produces zero duplicate rows'?" → yes.
   - Slice 3 (N+1): confirmed as-is.
   - Slice B: confirmed empty.

5. **Iterate** — none needed this round.

6. **Done** — prints "3 functional slices (1 scaffolding, 2 functional,
   1 cleanup), plus bugfixes parking lot." and the path
   `projects/sp-api-listings-ingestion/plan.md`.

## What this example illustrates

- The plan is drafted directly into `plan.md`, no draft/publish split.
- The per-slice grill can change a recommended HITL/AFK call when the user
  has better information (sandbox credentials).
- Slice numbering and dependency shape mirror the GitHub-backed original
  exactly — only the storage location changed.
