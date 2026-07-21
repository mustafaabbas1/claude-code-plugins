---
name: implement
description: Implement a single issue file end-to-end — pull context from the upstream projects/<slug>/definition.md and plan.md, plus ADRs and the codebase, restructure the issue body to the canonical template, run a strict TDD loop where applicable, and open a ready-for-review PR. Use when the user wants to implement an issue, work on a ticket, ship a PR from an issue file, or pick up an AFK-labelled piece of work. Invoke as `/sdlc:implement <issue-arg>` where issue-arg is `<project-folder>/<NN>`, a path to the issue file, or a bare `<NN>` (only when it's unambiguous across projects).
---

This skill is the last mile of the pipeline: `sdlc:define` → `sdlc:plan` →
`sdlc:issues` → **`sdlc:implement`**. One issue per invocation. Output is a
branch + PR ready for human review. The skill assumes you'll run it in plan
mode so you can approve the plan + restructured issue body before any side
effect lands.

Unlike the upstream skills, this one still talks to GitHub — it still opens
a real PR for the code change. It just resolves the work item from a local
file instead of `gh issue view`, and there's no GitHub issue number to close
or reference.

Issue files carry **no status field** (open/closed/done) by design — a
`## Depends on` entry is informational prose for a human or this skill to
read, never a gate this skill enforces by querying state. Treat it as a
reminder to confirm with the user, not a blocking check.

## Process

### 1. Preflight

- Working tree clean (`git status --porcelain` empty). Dirty → hard stop. Do
  not stash, do not discard.
- `origin/<default-branch>` test suite green (run targeted baseline). Red →
  hard stop. The skill cannot tell its own failures from pre-existing ones.
- `gh auth status` succeeds (needed later for the PR).

### 2. Resolve the issue

Accept any of:
- `<project-folder>/<NN>` (e.g. `sp-api-listings-ingestion/03`)
- A path to the file (e.g.
  `projects/sp-api-listings-ingestion/issues/03-batch-all-sellers.md`)
- A bare `<NN>` — only when exactly one file across all
  `projects/*/issues/` starts with that number; otherwise list the matches
  and ask which project.

Read the resolved issue file. Parse its `## Depends on` section. For each
referenced filename: check whether it still exists (it should) and surface
it to the user as a reminder ("this depends on
`02-ingest-one-seller.md` — confirm that's already implemented and merged")
rather than blocking — there is no status field to query, so this is advisory
only. If the user isn't sure, offer to check `git log --oneline --all
--grep="02-ingest-one-seller"` for a matching past implementation commit/PR
as a heuristic, but proceed on the user's word either way.

### 3. Gather context

Read, in order:

1. The issue file itself.
2. `## Source plan` reference — read `projects/<slug>/plan.md` and locate
   the named slice section.
3. `projects/<slug>/definition.md` — glossary, principles, out-of-scope.
4. `docs/adrs/**/*.md` — every ADR is a constraint. Any direct conflict with
   the issue is a **hard stop** with the offending ADR surfaced.
5. `CLAUDE.md`, repo `README.md`, top-level codebase shape.
6. Bounded codebase pass on `Modules touched` + depth-1 dependents (callers
   and callees). Read enough to know how the change integrates, not the
   whole tree.
7. `gh pr list --search "<module-path>"` — flag potential merge collisions.

### 4. Restructure + plan

Draft a fully-structured replacement for the issue body using the canonical
template (see `REFERENCE.md`) if the file drifted from it (e.g. was
hand-written or edited loosely). Fold original prose into the appropriate
sections; never preserve verbatim. Best-guesses for fields the skill cannot
confidently infer (Mode, Modules touched) are surfaced for user adjustment.

Classify each AC bullet:
- **behavioral** → becomes a test in the test list.
- **observational** ("appears in the plan," "file exists") → not
  implementable code; checked off only when structurally true.
- **structural** ("uses package X," "lives at path Y") → constraint on the
  implementation, not a test.

Declare **TDD fit: yes / no / partial** up front, with reasoning.
(Cleanup-style issues commonly land at *no*; new-behavior issues at *yes*;
scaffolding at *partial*.)

Detect the test runner (see `REFERENCE.md` for the cascade). If no runner is
configured, surface as a plan-mode question — do not pick one unilaterally.

Output to plan mode: the restructured issue body, the test list, the
TDD-fit call, the branch name, and the detected runner. Wait for the user's
approval.

### 5. Commit the restructure

On approval, if the issue file's body changed, edit
`projects/<slug>/issues/<NN>-<slug>.md` in place (a normal file edit — there's
no API call to make, unlike the GitHub-backed version's `gh issue edit`).

### 6. Branch

`git fetch origin`, then `git checkout -b issue-<NN>-<slug> origin/<default-branch>`.

If branch exists locally: reuse. If it has commits beyond base, treat as
resume — re-run the test list and enter at RED for any failing test; honour
passing tests as done. If a PR is already open against the branch: do not
create a new one; push updates the existing.

### 7. TDD loop (only if TDD-fit is yes or partial)

For each behavioral AC bullet in order:

1. **RED**: write one failing test exercising the behavior through a public
   interface. Commit (`test: <behavior>`).
2. **GREEN**: minimal code to pass. Commit (`feat: <behavior>` or `fix:
   <behavior>` for bugfixes).
3. **REFACTOR** (optional): only while green. Commit (`refactor: <area>`) if
   anything changed.

Never refactor while red. After every cycle, re-run the targeted test set to
confirm no regression in this PR's behaviors.

If a test will not go green after 3 implementation attempts: stop. Commit
the failing test, push, open a **draft** PR with a `## Blocker` section
explaining what was tried.

If an ADR conflict is discovered mid-loop: same path — commit what exists,
push, open draft PR with `## Blocker: ADR-NNNN conflict`.

### 8. Pre-PR full suite

Run the full test suite. If red on tests outside the AC list: investigate;
either fix (if regression from this PR) or hard stop (if pre-existing —
should have been caught in step 1, surface as defect).

### 9. Open PR

Push the branch. `gh pr create` with title `sdlc/<NN>: <issue title>`, body
from the template in `REFERENCE.md` (opening line references the issue file
path instead of `Closes #N`, summary, AC checklist, test plan, notes,
plan-slice footer pointing at `projects/<slug>/plan.md`). Ready for review,
**not** draft (draft is reserved for the blocker case in step 7).

Do not set reviewers, labels, assignees, or projects. Human triage territory.

### 10. Hand-off

Print the PR URL. There's no GitHub issue to auto-close and, per this
pipeline's design, no status field on the issue file to update — the merged
PR and its commits are the record. Do not move, rename, or annotate the
issue file.

## Constraints

- Never force-push. Never `git reset --hard`. Never `git checkout --` to
  discard.
- Never edit the issue file without user approval of the restructured form.
- Never set PR reviewers, labels, assignees, or projects.
- Never add a status/done marker to the issue file — that's an explicit
  non-goal of this pipeline.
- Single issue per invocation. No batching. No auto-pick when the issue arg
  is ambiguous.
- Plan mode is the gate. If invoked outside plan mode, behave identically —
  the user is on their own for review.

See `REFERENCE.md` for the issue restructure template, PR body template,
`gh`/`git` calls, test-runner detection cascade. See `EXAMPLES.md` for an
end-to-end walkthrough.
