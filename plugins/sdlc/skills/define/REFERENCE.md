# define — reference

Detailed mechanics behind the `sdlc:define` skill. SKILL.md describes the
process; this file documents the file layout and resolution rules.

## File layout

```
projects/<slug>/
└── definition.md
```

`<slug>` is a kebab-case derivation of the project's working title (lowercase,
spaces/punctuation → hyphens, no trailing hyphen). This is the identity used
by every other `sdlc:*` skill — `plan` and `issues` both key off the same
`<slug>` folder.

## Resolving the project folder

**Create mode** (`/sdlc:define` with no arg):

```bash
git rev-parse --show-toplevel   # repo root
ls projects/ 2>/dev/null        # existing slugs, for collision check
```

If `projects/<slug>/` already exists, do not silently overwrite — tell the
user and ask whether they meant to update it (or want a different slug).

**Update mode** (`/sdlc:define <arg>`):

Accept any of:
- A bare slug: `sp-api-listings`
- A path: `projects/sp-api-listings`
- A fuzzy fragment: `listings` → match against `ls projects/` if exactly one
  folder contains the fragment; otherwise list candidates and ask.

If no folder matches at all, ask whether to create it fresh instead (falls
back to create-mode behavior).

## Why no gitignore, no draft/publish split

The waste-line-style GitHub-backed version of this skill drafted into a
gitignored local file and "published" to a GitHub Project README as a
separate step, because the durable copy lived on GitHub and the local file
was disposable scratch. Here, there is no external system — the local file
**is** the durable copy. So:

- No gitignore entry for `projects/**` — it must ship in the repo.
- No separate publish step — the file is edited directly to its final state.
- No draft directory — `projects/<slug>/definition.md` is opened and edited
  in place from the first draft onward.

## Failure handling

There's no publish call that can fail here (no network, no API). The only
failure mode is a filesystem one (e.g. no write permission on `projects/`) —
surface the error verbatim and stop; don't retry silently.
