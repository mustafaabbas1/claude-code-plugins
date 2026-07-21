# issues — reference

Detailed mechanics behind the `sdlc:issues` skill. SKILL.md describes the
process; this file documents file layout, numbering, and duplicate-detection
details.

## File layout

```
projects/<slug>/
├── definition.md
├── plan.md
└── issues/
    ├── 01-scaffold-types.md
    ├── 02-ingest-one-seller.md
    ├── 03-batch-all-sellers.md
    └── 04-idempotent-rerun.md
```

Numbering is two-digit, zero-padded, assigned at draft time in topological
(plan) order across all slices — the counter continues across slice
boundaries, it does not reset per slice. Filename order is therefore always
a valid dependency order. If the user reorders dependencies after the fact
by hand, they're responsible for renumbering the affected files; this skill
does not renumber on a later run.

## Determining the next issue number

```bash
ls projects/<slug>/issues/ 2>/dev/null | sort   # existing NN-*.md, sorted
```

Take the highest existing `NN`, increment. On a fresh project (`issues/`
doesn't exist yet), start at `01`.

## Duplicate awareness during the per-slice grill

Before proposing new issues for a slice, check whether any already exist for
it:

```bash
grep -l "^<N> — " projects/<slug>/issues/*.md 2>/dev/null
```

(Matches the `## Slice` section's first line, which starts with the slice
number.) This is purely informational — surfaced to the user as "Existing
issues for slice K: ..." so they can decide whether more are needed, not a
gate that blocks anything.

## Why no placeholder/substitution pass

The GitHub-backed version drafted issues with `#TBD-<slug>` placeholders in
`## Depends on`, because the real issue number wasn't known until `gh issue
create` returned it at publish time — issues were drafted in a batch, then
created in a second pass. Here, the file *is* the issue: its number is fixed
the moment its filename is chosen, before its body is even written. So
`## Depends on` can just name the real sibling filename directly, with no
placeholder or later substitution step.

## Failure handling

No network calls. The only failure mode is filesystem-level — e.g.
`projects/<slug>/plan.md` missing (stop, tell the user to run `/sdlc:plan`
first) or a write permission issue (surface verbatim, stop).
