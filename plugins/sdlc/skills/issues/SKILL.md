---
name: issues
description: Turn a project's plan.md into issue files — one issue per PR's worth of atomic work, written directly to projects/<slug>/issues/<NN>-<slug>.md. Use when the user wants to create implementation tickets from an existing plan, decompose slices into PR-sized issues, or populate a project's issues/ folder after planning. Invoke as `/sdlc:issues` (prompts to pick a project) or `/sdlc:issues <project-folder>`.
---

This skill consumes a project's `plan.md` (written by `sdlc:plan`) and
produces issue files under `projects/<slug>/issues/`. It does not plan —
`sdlc:plan` must already have run. Each issue is **one PR's worth of atomic
work**: a single acceptance criterion (or a tight cluster that must ship
together). Slices decompose into N issues — any N, including 1.

Unlike a GitHub-Issues-backed pipeline, there's no create-then-renumber step:
issue numbers are assigned the moment a file is written, so `## Depends on`
can reference real sibling filenames immediately — no placeholder
substitution pass needed.

## Process

### 1. Resolve the target project

Same rule as `sdlc:plan`: arg resolves against `projects/*/` (exact, then
fuzzy match, ask if ambiguous); no arg lists candidates and asks the user to
pick. Never auto-pick.

### 2. Require a plan

Read `projects/<slug>/plan.md`. If it doesn't exist, stop: "run `/sdlc:plan`
first." Do not fall back to inventing slices from the definition alone.

### 3. Set up the issues folder

Ensure `projects/<slug>/issues/` exists. Scan existing `<NN>-*.md` filenames
to find the current max `NN`; new issues continue that counter (topological
order across slices — the counter never resets per-slice).

### 4. Per-slice grill

Walk the plan's slices in dependency order (0, 1, …, N+1; skip B — B is a
parking lot filled during implementation, not decomposed here). For each
slice:

1. **Duplicate awareness.** Grep `projects/<slug>/issues/*.md` for this
   slice's number in the `## Slice` line. Print "Existing issues for slice
   K: 03-foo.md, 04-bar.md" or "none."
2. **Resolve open questions one at a time.** For each open question in the
   slice: surface it, give a recommended answer with rationale, wait for
   the user. If resolved, fold into AC / module list / behavior. If
   deferred, mark as defer-to-issue (carries into the body of whichever
   issue inherits it).
3. **Propose decomposition.** Walk the slice's AC bullets; group ones that
   must ship together; each group is one proposed issue. Print proposed
   titles + one-line summaries.
4. **One focused grill question** challenging the split (e.g., "AC 2 and 3
   share a migration — bundled as one issue; agree, or split?"). Wait for
   the user.
5. **Write the confirmed issues directly** to
   `projects/<slug>/issues/<NN>-<slug>.md` from `issue-template.md`,
   numbered in plan/topological order (continuing the running counter
   across slices). Because the filename is assigned at write time, fill
   `## Depends on` with the real sibling filenames immediately.

Skip slices that have no AC and only open questions — surface them, ask
whether to retire the slice or escalate back to `sdlc:plan`.

### 5. Iterate

The user may edit any file in `issues/` directly, delete a proposed issue,
or add their own. **Always re-read from disk** before continuing. Never
silently overwrite user edits.

### 6. End-of-grill ADR nudge

List decisions resolved during the grill that look architecturally
load-bearing (library picks, API contracts, schema choices). Suggest
"consider an ADR?" for each. Do not write ADRs — that's its own workflow.

### 7. Done

There is no publish/manifest confirmation step beyond the per-slice grills
already done. Print a final list of created issue files grouped by slice,
e.g.:

```
Slice 0: 01-scaffold-types.md
Slice 1: 02-ingest-one-seller.md
Slice 2: 03-batch-all-sellers.md, 04-idempotent-rerun.md
```

## Constraints

- Do not modify `definition.md` (`sdlc:define` territory) or `plan.md`
  (`sdlc:plan` territory).
- Do not commit anything yourself — writing issue files is a normal file
  edit; the user decides when to commit.
- `projects/<slug>/issues/**` must **not** be gitignored.
- One issue per file: `<NN>-<slug>.md`. No YAML sidecar, no separate draft
  location — the file written during the grill is the final file.
- No status/tracking fields (open/closed/done) on issues, by design —
  dependencies are documented prose for a human or `sdlc:implement` to read,
  not enforced state.

See `REFERENCE.md` for file layout and numbering details. See `EXAMPLES.md`
for an end-to-end walkthrough.
