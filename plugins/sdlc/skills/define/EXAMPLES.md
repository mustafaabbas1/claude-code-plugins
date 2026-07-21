# define — example

One end-to-end walkthrough.

## Scoping a new project: SP-API listings ingestion

**User invokes:** `/sdlc:define` and says: "I want to pull every seller's
active listings from Amazon's SP-API daily and land them somewhere queryable
for the ads team."

### Skill behavior

1. **Anchor** — `git rev-parse --show-toplevel` → repo root. `ls projects/`
   → empty (first project in this repo).

2. **Resolve folder** — derives slug `sp-api-listings-ingestion` from the
   pitch. No collision. Create mode confirmed.

3. **Gather context** — reads `CLAUDE.md` (polyglot monorepo, Go/Python/TS,
   product integrates with SP-API and Ads API), `docs/adrs/` (none yet),
   root `README.md` (sparse, scaffold-only).

4. **Draft** — creates `projects/sp-api-listings-ingestion/definition.md`
   from the template, filling in:
   - **Outcome:** "Every seller's active listings are available in a
     queryable store within 24 hours of any change, without manual export."
   - **Problem:** ads team currently pulls listings ad hoc via spreadsheet
     exports; stale data breaks bid recommendations.
   - **Users:** ads team, future reporting dashboards.
   - Leaves **Domain glossary**, **Principles**, **Out of scope** thin —
     flagged as gaps.

5. **Grill the gaps**, one at a time:
   - "Domain glossary is empty. I'd propose defining 'listing', 'active
     listing', 'marketplace', and 'ingest run' as the core terms — agree, or
     is there a term I'm missing?" → user confirms, adds "seller".
   - "Principles: I'd propose 'prefer batch pulls over per-listing polling'
     since SP-API rate-limits per-endpoint — agree?" → user confirms, adds
     "no client is allowed direct DB access; only through the API."
   - "Out of scope: should real-time listing change notifications
     (webhooks) be explicitly out of scope for this project, deferred to a
     later one?" → user confirms.

6. **Iterate** — user tweaks the Outcome paragraph directly in their editor;
   skill re-reads from disk before continuing.

7. **Done** — prints:
   ```
   projects/sp-api-listings-ingestion/definition.md

   Outcome: Every seller's active listings are available in a queryable
   store within 24 hours of any change, without manual export.
   ```

## What this example illustrates

- The file is created and edited in place from the first draft — no
  temporary/gitignored intermediate file.
- Gaps are resolved one question at a time, each with a recommended answer,
  never batched.
- The skill never invents glossary terms or principles on its own — every
  addition is confirmed by the user.
- Nothing is committed to git by the skill itself; that's left to the user.
