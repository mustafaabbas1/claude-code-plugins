# issues — example

One end-to-end walkthrough.

## Decomposing `sp-api-listings-ingestion`'s plan into issues

**Background.** `/sdlc:plan sp-api-listings-ingestion` published a plan with
Slice 0 (scaffolding, AFK), Slice 1 (ingest one seller, AFK), Slice 2 (batch
+ idempotent reruns — two AC bullets: "batches across all sellers" and
"reruns don't duplicate rows"), Slice 3/N+1 (cleanup), Slice B (empty).

**User invokes:** `/sdlc:issues sp-api-listings-ingestion`

### Skill behavior

1. **Resolve target, require plan** — both present.

2. **Set up issues folder** — `projects/sp-api-listings-ingestion/issues/`
   doesn't exist yet; created. No existing files, counter starts at `01`.

3. **Slice 0 grill:**
   - No existing issues for slice 0.
   - No open questions.
   - Decomposition: single AC ("all three modules scaffolded with types") →
     one issue.
   - Grill question: "One issue for all three modules, since they're
     trivially small stubs — agree, or split per-module?" → user agrees,
     one issue.
   - Writes `01-scaffold-modules.md`.

4. **Slice 1 grill:**
   - No existing issues.
   - No open questions.
   - Single AC cluster → one issue.
   - Writes `02-ingest-one-seller.md`, `## Depends on` → `01-scaffold-modules.md`.

5. **Slice 2 grill:**
   - No existing issues.
   - No open questions.
   - Two AC bullets ("batches across all sellers", "reruns don't duplicate
     rows") — proposes splitting into two issues since they're independently
     testable and one is strictly harder than the other.
   - Grill question: "Split batching and idempotency into separate issues —
     agree, or bundle since they'll likely touch the same loop?" → user
     says split.
   - Writes `03-batch-all-sellers.md` (depends on `02-...md`) and
     `04-idempotent-rerun.md` (depends on `03-...md`).

6. **Slice 3 (N+1) grill:**
   - Single AC → one issue: `05-cleanup-placeholder-types.md`, depends on
     `02`, `03`, `04`.

7. **Iterate** — user opens `04-idempotent-rerun.md` and tightens the AC
   wording directly; skill re-reads before the ADR nudge.

8. **ADR nudge** — flags "single-table schema keyed by
   seller+marketplace+asin+date" (from the plan's cross-cutting decisions)
   as worth an ADR, since it's now load-bearing across 3 issues.

9. **Done** — prints:
   ```
   Slice 0: 01-scaffold-modules.md
   Slice 1: 02-ingest-one-seller.md
   Slice 2: 03-batch-all-sellers.md, 04-idempotent-rerun.md
   Slice 3: 05-cleanup-placeholder-types.md
   ```

## What this example illustrates

- Numbers are assigned immediately as files are written — no placeholder
  substitution, no second publish pass.
- `## Depends on` references are real filenames from the first write.
- The per-slice grill can produce more than one issue per AC cluster when
  the user pushes back on a proposed bundling.
