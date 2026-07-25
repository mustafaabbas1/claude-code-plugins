---
name: cleanup
description: Spawn 8 parallel sub-agents against a PR (security, correctness/error-handling/concurrency, breaking-change/API compatibility, test quality, performance, code structure (DRY/SOLID/GoF), maintainability (readability/observability/docs), general code-review), synthesize their findings into one prioritized list, then walk the user through it one item at a time. Use when the user wants a deep multi-angle review of a pull request, says "review this PR with sub-agents", or invokes /code-monkey:cleanup <PR>.
---

Spawn 8 sub-agents in parallel (one message, parallel tool calls). Each is
self-contained (PR ref + summary of what changed/why + its job verbatim,
below), in priority order — also the tie-break order in synthesis:

1. Run `/security-review`, report findings.
2. Correctness/error-handling (swallowed exceptions, missing boundary
   checks, unhandled failure modes) + concurrency/race conditions (shared
   state, locking, retry idempotency).
3. Breaking changes/API compatibility (public signature changes,
   migration/rollback safety, versioning).
4. Tests: missing coverage of major paths, config-only tests, unjustified
   mocking/patching (favor factories/fixtures unless a real external
   service), tests that could be parameterized.
5. Performance/efficiency (N+1s, hot-path complexity, missing batching/pagination).
6. Code structure: DRY opportunities, SOLID violations, Gang of Four pattern improvements.
7. Maintainability: readability (naming, overly verbose/stale comments),
   missing observability (logs/metrics for new failure paths), stale docs
   or naming drift from codebase conventions.
8. Run `/code-review`, report findings.

## Synthesize

One prioritized list, not a concatenation. Merge overlapping findings;
on conflict, lower-numbered job wins; order by real severity/impact, not
source; tag each item with which review(s) surfaced it.

## Walk through

Present recommendations one at a time (what, why, source), wait for the
user's call — fix / skip / discuss — before the next. If fixing, make the
change first, then advance.
