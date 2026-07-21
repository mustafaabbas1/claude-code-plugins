# implement — reference

Detailed mechanics behind the `sdlc:implement` skill. SKILL.md describes the
process; this file documents templates, `gh`/`git` calls, and the
test-runner detection cascade.

## Resolving the issue arg

```bash
# <project-folder>/<NN>
projects/<slug>/issues/<NN>-*.md

# bare <NN> across all projects
ls projects/*/issues/<NN>-*.md 2>/dev/null
```

If the bare-`<NN>` glob matches files in more than one project, list them
and ask which one. If it matches a path directly, use it as-is.

## Canonical issue body template

Used in step 4 (restructure) — same shape `sdlc:issues` writes:

```markdown
## Slice

<N> — <slice title> (from projects/<slug>/plan.md)

**Mode:** AFK | HITL — <one-line reason>

## User-visible behavior

<one paragraph>

## Acceptance criteria

- <testable bullet>

## Modules touched

- <module> — <what changes>

## Depends on

- <NN-slug>.md
- (or "none")

## Open questions

- (none) | - <item>

## Source plan

projects/<slug>/plan.md — slice <N>
```

When restructuring a hand-edited issue, sections the skill cannot
confidently infer get a best-guess that the user adjusts in plan mode before
approval.

## PR body template

```markdown
Implements `projects/<slug>/issues/<NN>-<slug>.md`.

## Summary

<one paragraph narrowed to what this PR delivers>

## Acceptance criteria

- [x] <behavioural AC, verified by tests>
- [x] <structural AC, verified by inspection>
- [ ] <observational AC, satisfied by the PR existing>

## Test plan

- <test name 1>
- <test name 2>

## Notes

<anything surfaced during implementation: ADR decisions, open questions
resolved, follow-up flagged, degraded context if any>

---
Implemented from [slice <K>](projects/<slug>/plan.md) via `sdlc:implement`.
```

## Test-runner detection cascade

Walked at step 4 (planning) for the workspace owning the touched files (walk
up from the first file in `Modules touched` to the nearest `package.json`).

1. **`package.json` `scripts.test`** present → use it. Package manager from
   `packageManager` field or lockfile (`pnpm-lock.yaml` → `pnpm test`,
   `yarn.lock` → `yarn test`, `package-lock.json` → `npm test`).
2. **No `test` script, but a test framework in `devDependencies`**
   (`vitest`, `jest`, `mocha`, `playwright`) → use the framework's CLI
   directly via the workspace's package manager (e.g. `pnpm exec vitest
   run`).
3. **Non-JS stack** (`pyproject.toml`, `Cargo.toml`, `go.mod`): use the
   equivalent (`pytest`, `cargo test`, `go test ./...`). Detected by
   presence of the marker file.
4. **No runner configured** → surface in plan mode as "no test runner
   detected; configure one before TDD, or override TDD-fit to no."

Targeted vs full runs:

- Inside the TDD loop: target the test files just touched (`vitest related
  <paths>`, `jest --findRelatedTests <paths>`, `pytest <path>`).
- Before opening the PR: full suite via the detected runner.

## `git`/`gh` calls used

### Dependency check (informational only)

```bash
git log --oneline --all --grep="<dependency-slug>"
```

Purely a heuristic to help the user confirm a dependency landed; never a
gate.

### Open PR collision check

```bash
gh pr list --repo <owner>/<repo> --state open --search "<module-path>" \
  --json number,title,headRefName
```

Surface results in the plan as "Open PRs touching <module>: #<N> <title>" —
informational, not blocking.

### PR create (step 9)

```bash
gh pr create \
  --repo <owner>/<repo> \
  --base <default-branch> \
  --head issue-<NN>-<slug> \
  --title "sdlc/<NN>: <issue title>" \
  --body-file <pr-body.md>
```

Add `--draft` **only** in the blocker case (step 7 fallthrough).

## Commit messages

Conventional Commits style, per the per-cycle commit decision:

- `test: <behavior>` — RED commit.
- `feat: <behavior>` / `fix: <behavior>` — GREEN commit. `fix:` for bugfix
  issues (the issue's user-visible behavior reads as "restore X"), `feat:`
  otherwise.
- `refactor: <area>` — REFACTOR commit, optional, only while green.

Body of each commit: empty by default. The PR body carries the narrative;
commits are scannable headers.

## Failure modes (quick reference)

| Situation | Action |
|---|---|
| Working tree dirty | Hard stop, print `git status`. |
| `origin/<default>` red | Hard stop, surface failing tests. |
| Dependency uncertain | Surface as reminder, ask user; never block automatically. |
| ADR conflict (pre-loop) | Hard stop with ADR reference. |
| ADR conflict (mid-loop) | Commit progress, push, draft PR with `## Blocker`. |
| Test won't go green (3 attempts) | Commit progress, push, draft PR with `## Blocker`. |
| Existing branch with commits | Resume — re-run tests, enter at RED for failing. |
| Existing open PR on branch | Don't create new; push updates existing. |
| No test runner configured | Surface in plan, ask user. |

## Runtime dependencies

- `gh` CLI authenticated, with repo write access (for the PR only — no
  `project` scope needed, since there's no GitHub Project involved).
- `git` for branch management.
- A working test runner in the target workspace, or willingness to add one
  (surfaced via plan mode).
