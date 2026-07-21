# Claude Code Plugins

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Available Plugins

### Impersonate

Make Claude speak as any person or character for the rest of your session. Claude will research the person's speaking style, analyze their patterns, and adopt their voice while still providing accurate technical assistance.

**Example:**
```
/fun:impersonate Morgan Freeman
```

**What it does:**
1. Searches for quotes, interviews, and speech patterns of the specified person or character
2. Identifies key characteristics like vocabulary, tone, catchphrases, and sentence structure
3. Adopts that speaking style for the remainder of the session
4. Maintains technical accuracy despite the voice change

### SDLC

Scope, plan, decompose, and implement a project using plain files checked
into the repo instead of an external tracker. Four skills form a pipeline:

```
/sdlc:define    # capture outcome + why → projects/<slug>/definition.md
/sdlc:plan      # break the outcome into vertical slices → projects/<slug>/plan.md
/sdlc:issues    # decompose slices into PR-sized issues → projects/<slug>/issues/<NN>-<slug>.md
/sdlc:implement # implement one issue end-to-end (TDD) and open a PR
```

Everything the pipeline produces lives under `projects/<project-folder>/` at
the repo root — `definition.md`, `plan.md`, and an `issues/` folder of
`<NN>-<issue-title>.md` files — and is meant to be committed like any other
file. `git log` gives you history on these for free; there's no external
project tracker involved. `sdlc:implement` is the exception: it still opens
a real GitHub PR for the code change itself.

**Example:**
```
/sdlc:define
/sdlc:plan sp-api-listings-ingestion
/sdlc:issues sp-api-listings-ingestion
/sdlc:implement sp-api-listings-ingestion/03
```

## Installation

Add the marketplace:

```bash
claude plugin marketplace add mustafaabbas1/claude-code-plugins
```

Install a plugin:

```bash
claude plugin install fun@mustafaabbas1-plugins
claude plugin install sdlc@mustafaabbas1-plugins
```

Use it:

```bash
/fun:impersonate <person or character name>
/sdlc:define
```

## Contributing

To add a new plugin:

1. Create a new directory under `plugins/` with a `plugin.json` and `skills/` subdirectory
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Open a PR

## License

MIT
