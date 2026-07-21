# plan — reference

Detailed mechanics behind the `sdlc:plan` skill. SKILL.md describes the
process; this file documents the file layout and history mechanics.

## File layout

```
projects/<slug>/
├── definition.md   # input, written by sdlc:define
└── plan.md         # output of this skill
```

## Resolving the target project

Same rule as `sdlc:define`'s update mode:

```bash
ls projects/                          # list candidates
```

- Arg given: exact match on folder name, then fuzzy fragment match. Ask if
  ambiguous.
- No arg: list all `projects/*/` folders with the first line under `##
  Outcome` in each `definition.md` (or the file's tagline line) so the user
  can pick with context, not just a bare name.

## Reading plan history

There's no append-only status-update log to fetch — `plan.md` is a single
file that gets overwritten on each revision. To recover history:

```bash
git log --oneline -- projects/<slug>/plan.md     # revision timestamps
git log -p -- projects/<slug>/plan.md            # full diff history
git show <commit>:projects/<slug>/plan.md        # a specific past version
```

Use `git log -1 --format=%ad -- projects/<slug>/plan.md` to tell the user
when the plan was last touched, mirroring what the GitHub-backed version got
from status-update timestamps — for free, without any extra bookkeeping
file.

## Why no publish step

The GitHub-backed version published the plan as an append-only
`createProjectV2StatusUpdate` mutation, specifically so old plans stayed
visible as history. Here, git already is that history mechanism for a
tracked file — there's nothing an extra "publish" indirection would buy, so
the skill just edits `plan.md` in place.

## Failure handling

No network calls, no API. The only failure mode is filesystem-level (e.g.
`projects/<slug>/` missing because `sdlc:define` was never run) — in that
case, stop and tell the user to run `/sdlc:define` first, don't silently
create a bare folder.
