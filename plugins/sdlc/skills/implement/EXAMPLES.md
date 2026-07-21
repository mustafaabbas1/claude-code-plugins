# implement — example

One end-to-end walkthrough.

## Implementing `03-batch-all-sellers.md`

**Background.** Earlier in the pipeline:
- `/sdlc:define` created `projects/sp-api-listings-ingestion/definition.md`
  with the Outcome "pull every seller's active listings daily and put them
  somewhere queryable."
- `/sdlc:plan` produced `plan.md` with slices 0–3 plus an empty bugfixes
  slice.
- `/sdlc:issues` produced `01-scaffold-modules.md`,
  `02-ingest-one-seller.md`, `03-batch-all-sellers.md` (slice 2, AFK,
  depends on `02-ingest-one-seller.md`), `04-idempotent-rerun.md`,
  `05-cleanup-placeholder-types.md`.

**User invokes:** `/sdlc:implement sp-api-listings-ingestion/03` (in plan
mode).

### Skill behavior

1. **Preflight** — working tree clean. Baseline `pnpm test` on `origin/main`
   green. `gh auth status` ok.

2. **Resolve the issue** — matches
   `projects/sp-api-listings-ingestion/issues/03-batch-all-sellers.md`.
   `## Depends on` lists `02-ingest-one-seller.md`. The skill surfaces: "This
   depends on `02-ingest-one-seller.md` — has that landed?" User confirms
   it merged last week. Proceeds (no automated gate — just the confirmation).

3. **Gather context:**
   - Issue body parsed.
   - `## Source plan` → reads `plan.md`, locates Slice 2 ("batch across all
     sellers with idempotent reruns") and its cross-cutting decisions
     (single-table schema keyed by seller+marketplace+asin+date).
   - `definition.md`: glossary, principle "prefer batch pulls over
     per-listing polling."
   - `docs/adrs/` now has ADR-0007 (the single-table schema, written after
     the `sdlc:issues` ADR nudge). Consistent with the issue.
   - `CLAUDE.md` + workspace shape.
   - Bounded codebase pass: `etl/sp-api/listings/ingest.py` (from issue 02),
     `packages/db/schemas/listings.py`. Depth-1 dependents: none yet.
   - `gh pr list --search "etl/sp-api/listings"` → no open PRs touching
     this area.

4. **Restructure + plan.** Issue body already matches the canonical
   template (written by `sdlc:issues`); no restructure needed. AC
   classification:
   - "Ingest runs across every seller in one invocation" → behavioral.
   - "Row count across all sellers matches sum of per-seller reports" →
     behavioral.
   - "Uses the single-table schema from ADR-0007" → structural.

   **TDD fit: yes.** Two behavioral ACs → two tracer-bullet tests.

   Test runner: `etl/` has `pyproject.toml` and `pytest` — cascade step 3.
   Branch name: `issue-03-batch-all-sellers`.

   Plan output: restructured body (unchanged), test list, TDD-fit, branch,
   runner. **User approves in plan mode.**

5. **Commit the restructure** — skipped, body already canonical, no edit
   needed.

6. **Branch.** `git fetch origin`, `git checkout -b issue-03-batch-all-sellers
   origin/main`. No existing branch or PR.

7. **TDD loop:**
   - **Test 1: runs across every seller.**
     - RED: `etl/sp-api/listings/test_batch.py` asserts
       `batch_ingest(seller_ids)` calls `ingest_for_seller` once per ID.
       Fails — `batch_ingest` not defined. Commit `test: batch ingest runs
       across every seller`.
     - GREEN: thin loop over `ingest_for_seller`. Commit `feat: batch ingest
       across all sellers`.
   - **Test 2: row count matches per-seller sum.**
     - RED: asserts `sum(len(r) for r in per_seller_results) ==
       total_rows_landed`. Fails — batch path double-counts retried
       sellers. Commit `test: batch row count matches per-seller sum`.
     - GREEN: fix retry accounting. Commit `fix: avoid double-counting
       retried sellers in batch total`.
   - No refactor needed this round.

8. **Pre-PR full suite.** `pytest etl/` all green.

9. **Open PR.** `git push -u origin issue-03-batch-all-sellers`. `gh pr
   create`:
   - Title: `sdlc/03: Batch ingest across all sellers`
   - Body (abbreviated):
     ```
     Implements `projects/sp-api-listings-ingestion/issues/03-batch-all-sellers.md`.

     ## Summary
     Extends single-seller ingest to loop across every seller in one
     invocation, with row counts reconciled against per-seller reports.

     ## Acceptance criteria
     - [x] Ingest runs across every seller in one invocation
     - [x] Row count across all sellers matches sum of per-seller reports
     - [x] Uses the single-table schema from ADR-0007 (verified by inspection)

     ## Test plan
     - batch ingest runs across every seller
     - batch row count matches per-seller sum

     ## Notes
     - Fixed a double-counting bug in retry accounting, surfaced by test 2.

     ---
     Implemented from [slice 2](projects/sp-api-listings-ingestion/plan.md) via `sdlc:implement`.
     ```
   - Ready for review, not draft.

10. **Hand-off.** Skill prints the PR URL. No issue file edited or
    annotated — the merged PR is the record.

## What this example illustrates

- The dependency check is a confirmation, not an automated gate — the user
  vouches for `02-ingest-one-seller.md` having landed; the skill doesn't try
  to verify it via any status field.
- The full upstream context chain (definition → plan → ADRs → codebase) is
  read once at the top.
- AC classification still governs verification per bullet even without
  GitHub issue numbers.
- The PR references the issue file by path and the plan by path + slice,
  with no GitHub issue to close.
