---
name: define
description: Capture the outcome and why of a project as projects/<slug>/definition.md — the single source of truth, checked into the repo. Use when the user wants to scope a new project, define a new workstream, or refine an existing project's outcome/glossary/principles. Invoke as `/sdlc:define` (create) or `/sdlc:define <project-folder>` (update).
---

This skill produces a **project definition** and writes it directly to
`projects/<slug>/definition.md` at the repo root. Project definitions are the
umbrella under which plans and issues later hang — they capture *outcome and
why*, not user stories, modules, or tickets. Those belong to `sdlc:plan` and
`sdlc:issues`.

`projects/<slug>/definition.md` is the **single source of truth** — a real,
committed file, not a draft. There is no external system to publish to and
no separate publish step: editing the file to completion *is* the deliverable.
Git gives you history on it for free (`git log -p projects/<slug>/definition.md`).

## Modes

- `/sdlc:define` — create a new project. Derive a kebab-case slug from the
  working title; if `projects/<slug>/` already exists, ask for a different
  name (or confirm the user means to update it).
- `/sdlc:define <project-folder>` — update an existing project. Accept a bare
  slug, a `projects/<slug>` path, or a fuzzy match against existing folder
  names under `projects/`. If nothing matches, ask whether to create it fresh.

## Process

### 1. Anchor the repo root

Run `git rev-parse --show-toplevel` to find where `projects/` lives. If this
isn't a git repo, ask the user where to root `projects/` instead of guessing.

### 2. Resolve the project folder

Create mode: derive a slug from the conversation's working title. List
`projects/` (if it exists) to check for a collision; if the slug is taken,
tell the user and ask for a different name or confirm they meant update mode.

Update mode: resolve the arg against existing `projects/*/` folder names. Be
permissive — exact match, then prefix/fuzzy match. If ambiguous, list
candidates and ask which one.

### 3. Gather context

Read these sources, in priority order:

1. The current conversation transcript (the user's pitch).
2. Update mode: the existing `projects/<slug>/definition.md`.
3. `docs/adrs/**/*.md` — every ADR is a constraint the definition must
   respect. Flag any conflict during grilling.
4. `CLAUDE.md` at repo root.
5. `README.md` at repo root.
6. Codebase shape: manifest files (`package.json`, `go.mod`, `pyproject.toml`,
   etc.), workspace config, top-level dirs.
7. Sibling folders under `projects/` (`ls projects/`, skim each
   `definition.md`'s Outcome line) — to avoid re-defining something already
   scoped elsewhere.

### 4. Draft

Create `projects/<slug>/` if needed and start (or open) `definition.md` from
`definition-template.md`, filling in as much as can be synthesized from
context.

In **create mode**, conversation is primary. In **update mode**, the existing
file is primary and conversation is a modifier.

### 5. Grill the gaps

Identify sections that are hand-wavy after synthesis (most often: glossary,
principles, out-of-scope). Resolve **one at a time**:

- Pick the most foundational unresolved section.
- Ask one focused question.
- Provide a recommended answer with rationale.
- Wait for the user. Update the file. Move to the next gap.

Never batch questions. Never silently invent glossary terms or principles.

### 6. Iterate

The user may edit `projects/<slug>/definition.md` directly in their editor,
or ask for changes in chat. Both work. **Always re-read the file from disk**
before treating it as current — never act on an in-memory copy that might be
stale.

### 7. Done

There is no publish step. When the user signals readiness ("looks good",
"done"), print the file path and a one-line recap of the Outcome. That's it —
the file on disk is the artifact.

## Constraints

- Do not create `plan.md` or `issues/`. Those are `sdlc:plan` and
  `sdlc:issues` territory.
- Do not commit anything to git yourself — writing/editing
  `projects/<slug>/definition.md` is a normal file edit; let the user decide
  when to commit, same as any other file Claude edits in a session.
- `projects/<slug>/definition.md` must **not** be gitignored — unlike a
  scratch draft, this file is the durable record and belongs in the repo.

See `REFERENCE.md` for the file-resolution details. See `EXAMPLES.md` for an
end-to-end walkthrough.
