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

## Installation

Add the marketplace:

```bash
claude plugin marketplace add mustafaabbas1/claude-code-plugins
```

Install a plugin:

```bash
claude plugin install fun@mustafaabbas1-plugins
```

Use it:

```bash
/fun:impersonate <person or character name>
```

## Contributing

To add a new plugin:

1. Create a new directory under `plugins/` with a `plugin.json` and `skills/` subdirectory
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Open a PR

## License

MIT
