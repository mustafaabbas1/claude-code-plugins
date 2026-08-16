---
name: cleanup
description: Gated, multi-angle cleanup and review pass on a target — a PR, the current working diff (default), or a named module, package, subsystem, or directory. Spawns 8 parallel sub-agents (security, correctness/error-handling/concurrency, breaking-change/API compatibility, test quality, performance, code structure (DRY/SOLID/GoF), maintainability (readability/observability/docs), general code-review), synthesizes their findings into one prioritized list, then walks the user through it one item at a time with an honest cost/benefit and a recommendation — applying only what the user approves. Use when the user wants a deep review, says "clean up", "review this PR with sub-agents", "make it DRY / SOLID", "refactor holistically", or "tidy the tests".
---

# Cleanup — Gated Multi-Angle Review

Parallel multi-lens audit of a target, synthesized into one ranked list, then applied one approved item at a time.

## What this is (and isn't)

- **Is:** a full-spectrum pass — security, correctness, breaking changes, tests, performance, design structure, maintainability. Gated per item.
- **Isn't:** auto-apply cleanup (use `/simplify`). If the user wants changes applied without per-item approval, redirect them there.

## Core invariant — no auto-apply

**Every change is gated.** No item is applied without explicit user approval. Each is presented with a recommendation — including "skip" when that's the right call — and an honest cost/benefit. All of the following are violations:

- batching multiple decisions into one question,
- applying an "obvious" change without asking,
- bundling an approved change together with an unapproved one.

## Target resolution

First job is to figure out **what to review**. Order of precedence:

1. **Explicit arg** — a PR ref (`#1234`, a URL, a branch), or a path / module / package / subsystem name (`/code-monkey:cleanup web/backend/services/dbx_apcss`, `/code-monkey:cleanup the automated savings services`).
2. **Current working diff** — no arg and `git status` shows uncommitted or recent-branch changes: use `git diff <base>...HEAD` plus unstaged.
3. **Current working directory.**

For a *subsystem* argument, map it first: grep/Explore for the parallel modules, then read each fully before dispatching sub-agents. Reviewing across parallel modules surfaces consistency wins single-module review misses.

For a diff or module target, read the touched files **fully** — findings often live *near* the diff, not in it.

## Dispatch

Spawn 8 sub-agents in parallel (one message, parallel tool calls). Each is self-contained: the resolved target (PR ref, or explicit file list + base for a diff/module), a summary of what changed and why, and its job verbatim from the list below. Jobs are in priority order — also the tie-break order in synthesis.

1. Run `/security-review`, report findings.
2. Correctness/error-handling (swallowed exceptions, missing boundary checks, unhandled failure modes) + concurrency/race conditions (shared state, locking, retry idempotency).
3. Breaking changes/API compatibility (public signature changes, migration/rollback safety, versioning).
4. Tests: missing coverage of major paths, config-only tests, unjustified mocking/patching (favor factories/fixtures unless a real external service), tests that could be parameterized, per-file setup worth extracting into a helper or fixture.
5. Performance/efficiency (N+1s, hot-path complexity, missing batching/pagination).
6. Code structure: DRY opportunities, SOLID violations, Gang of Four pattern improvements.
7. Maintainability: readability (naming, overly verbose or stale comments — trim to the load-bearing "why"), missing observability (logs/metrics for new failure paths), stale docs or naming drift from codebase conventions.
8. Run `/code-review`, report findings.

Instruct every sub-agent to **look outside the target** before reporting:

- **Patterns to adopt.** A sibling module, page, or package may already solve the same problem cleanly — a `_setup_workspace` helper in one test file, a shared fixture in `conftest.py`, a utility in `utils/`. Propose adopting *that* pattern rather than inventing a new one.
- **Patterns to extract shared.** If the target duplicates code that already exists elsewhere, propose promoting it to a shared location.
- **Ripple effects.** If a change renames, extracts, or moves something, grep for callers, imports, and downstream usages. Cleanup that breaks unrelated call sites isn't cleanup.
- **Consistency across parallel modules.** If the target is one of N parallel services/pages/components, read the other N-1 to spot naming, log-wording, or structural drift worth aligning.

Sub-agents **must not** spawn agents of their own.

## Synthesize

One prioritized list, not a concatenation:

- merge overlapping findings;
- on conflict, lower-numbered job wins;
- order by real severity/impact, not by source;
- tag each item with which review(s) surfaced it;
- drop items that don't survive the trade-off priority below — a "finding" that would make the code worse isn't a finding.

## Trade-off priority

When two goals conflict, resolve in this order (highest first):

1. **Correctness & security** — does it work, and is it safe?
2. **Readability** — will a future reader understand this at a glance?
3. **Tests** — do they still tell a clear given/when/then story, and still catch regressions?
4. **DRY** — is duplicated *knowledge* eliminated? (Duplicated *code* expressing different domain intent is fine.)
5. **SOLID** — SRP / OCP / DIP violations worth fixing when they don't cost readability.
6. **GoF design patterns** — reach for a named pattern only if the skeleton has enough substance to earn its scaffolding.

If a change makes the code less readable to gain DRY, it's the wrong change. If a Template Method saves 15 lines of duplication but adds 40 lines of class scaffolding, skip it. Each recommendation states which rank it pulls toward and, if a lower-priority goal loses, names what's traded.

## Walk through

Present items **one at a time**: what, why, which review(s) surfaced it, a before/after snippet where useful, and an `AskUserQuestion` with the recommended option first. Wait for the user's call — fix / skip / discuss — before advancing. Record all responses, reconcile them, and then apply all together at the end, being sure to run linter and tests between each fix.

**On disagreement:** engage. Push back honestly if the user proposes something with poor cost/benefit (e.g. Template Method on three short parallel loops). Explain the trade-off, then defer to the user.

### Recommendation-writing rules

- **Be honest about payoff.** "Template Method adds ~40 lines of class scaffolding to eliminate ~15 lines of duplication."
- **Recommend "skip" when you mean it.** Not every item is worth applying; `AskUserQuestion` should sometimes lead with "Skip (Recommended)".
- **Match the codebase style.** If the repo uses `with pg_session_factory() as db:` inline throughout, don't propose fixtures that break that pattern — extract helpers that live inside it.
- **Distinguish real duplication from parallel domain logic.** Three services with slightly different domain rules (ownership check, extra fields) are *parallel*, not *duplicated*.

Examples of well-formed items:

- "Trim 4-line 'why' comment to 2 — drop what the log line already says; keep the two non-obvious points (missing = deleted-or-revoked; no auto-re-enable)."
- "Adopt the existing `_setup_workspace` pattern from the sibling test file across the other two. Cuts ~5 lines/test × 18 tests."
- "Skip: Template Method fits the shape, but the skeleton is 4 lines — the pattern earns its keep at 10+ line orchestration with multiple hook points."

## Commit

After all approved items land, offer a single follow-up commit (separate from any feature commit) with a concise message.
