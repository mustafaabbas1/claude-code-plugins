---
name: plan
description: Plan how to deliver a project's Outcome as a set of implementation-ready vertical slices, and write the plan directly to projects/<slug>/plan.md. Use when the user wants to plan a project, break work into vertical slices, decide modules and cross-cutting decisions before issues are created, or write a plan that `sdlc:issues` will later turn into issue files. Invoke as `/sdlc:plan` (prompts to pick a project) or `/sdlc:plan <project-folder>`.
---

This skill produces a **project plan** and writes it directly to
`projects/<slug>/plan.md`. The plan is **implementation-ready** — each slice
contains everything the downstream `sdlc:issues` skill (or a human) needs to
produce a working issue file. There is no separate PRD-style document between
this and implementation.

`plan.md` is a single, overwritten file — there's no append-only
status-update mechanism here (that was a GitHub Projects affordance). Git
already gives you the plan's full history for free: `git log -p
projects/<slug>/plan.md`.

## Process

### 1. Resolve the target project

- With an arg: resolve it against `projects/*/` the same way `sdlc:define`
  does (exact, then fuzzy match; ask if ambiguous).
- With no arg: `ls projects/` and present a numbered list (folder name + the
  first line of each `definition.md`'s Outcome). Ask the user to pick. Never
  auto-pick, even when only one project exists.

If `projects/` doesn't exist or the target has no `definition.md`, stop and
tell the user to run `/sdlc:define` first.

### 2. Gather context

Read these sources, in priority order:

1. **The project definition** — `projects/<slug>/definition.md`. This is the
   primary input.
2. **The existing plan**, if `projects/<slug>/plan.md` already exists. Summarize
   it ("plan currently has N slices, last touched <date from `git log -1
   --format=%ad -- projects/<slug>/plan.md>`") and ask whether to revise it in
   place or start fresh. Starting fresh is safe — git history preserves the
   old version.
3. The current conversation transcript.
4. `docs/adrs/**/*.md` — every ADR is a constraint. Flag conflicts during
   grilling; new decisions go into the plan's Cross-cutting decisions
   section.
5. `CLAUDE.md`, root `README.md`, top-level codebase shape.
6. **Bounded codebase pass** scoped by glossary terms in the definition —
   read the workspaces/dirs whose names match domain terms (e.g. definition
   mentions "SP-API listings" → read `etl/sp-api/`, `packages/types/sp-api/`).
   Needed to draft the Modules section concretely.

### 3. Draft

Open (or create) `projects/<slug>/plan.md` from `plan-template.md`. Fill in
this order:

1. **Approach** — one paragraph: the overall strategy.
2. **Modules** — from the codebase pass + definition. Each entry:
   responsibility, public interface (prose, not code), new vs modified,
   deep-module rationale.
3. **Cross-cutting decisions** — from ADRs and obvious unknowns. Each is a
   one-paragraph decision with rationale.
4. **Slice 0: Scaffolding** — derived from Modules. Whatever infra/types/
   plumbing must land before any functional slice can.
5. **Functional slices 1..N** — outcome-first decomposition, reality-checked
   against Modules. Each slice must cut vertically through multiple layers,
   not horizontally through one. Per slice: title, user-visible behavior,
   modules touched, acceptance criteria (testable bullets), HITL vs AFK with
   one-line reason, depends on, open questions.
6. **Cleanup / backfill slice (N+1)** — derived from the diff between
   current state and end state.
7. **Bugfixes slice (B)** — always present, starts empty. Outside the
   dependency graph, AFK, no depends-on.
8. **Cross-cutting open questions** and **Out of scope for this plan**.

Slice 0 and N+1 are present by default; the user may remove them if genuinely
unneeded. Bugfixes is always present.

### 4. Per-slice grill

Walk every slice in dependency order — 0, 1, 2, …, N+1, then B. For each:

- Print the slice (title, behavior, AC, modules, HITL/AFK, depends on, open
  questions).
- Ask one confirming question, with a recommended answer where applicable
  ("HITL because the auth flow requires interactive consent — agree?").
- Wait for the user. Update the file. Move to the next slice.

Do not skip slices. Do not batch.

### 5. Iterate

The user may edit `projects/<slug>/plan.md` directly or ask for changes in
chat. Both work. **Always re-read the file from disk** before treating it as
current.

### 6. Done

There is no publish step. When the user signals readiness ("looks good"),
print a one-line summary ("N functional slices, plus scaffolding/cleanup/
bugfixes") and the file path.

## Constraints

- Do not touch `projects/<slug>/definition.md` (that's `sdlc:define`
  territory) or write issue files (that's `sdlc:issues`).
- Do not commit anything yourself — writing/editing `plan.md` is a normal
  file edit; the user decides when to commit.
- `projects/<slug>/plan.md` must **not** be gitignored.
- Slices must be vertical. If a slice's "modules touched" list contains only
  one module, treat that as a smell and grill for whether it should be
  merged or expanded.

See `REFERENCE.md` for file-resolution and history details. See
`EXAMPLES.md` for an end-to-end walkthrough.
