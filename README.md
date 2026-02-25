# BrainLift Update Plugin

A focused Claude Code plugin for adding new sources to existing BrainLifts — guided DOK extraction and impact assessment.

## Installation

```bash
# Use directly from the plugin directory
claude --plugin-dir ./brainlift-plugin

# Or install as a shared plugin
cp -r brainlift-plugin ~/.claude/plugins/brainlift-plugin
```

## Skill

### `/brainlift-plugin:bl-update <url> [your thoughts]`

Add a new source to an existing BrainLift. Guides you through a 6-phase interview:

1. **Ingest** — Paste a URL (tweet, article, video, PDF) + optional thoughts. Content is fetched automatically.
2. **Route** — Finds your BrainLift files, matches the source to a BrainLift's Purpose/scope, you pick which one.
3. **Extract DOK1** — AI extracts facts from the source. You curate which to keep.
4. **Place & DOK2** — Suggests a Knowledge Tree category. Provides a DOK2 strawman for you to react against (you must write your own).
5. **DOK3/DOK4 Impact** — Shows existing insights/SPOVs, asks structured questions about support/contradiction/nuance. Guides you through writing any updates.
6. **Apply & Commit** — Previews changes, applies edits, git commits.

### Examples

```bash
/brainlift-plugin:bl-update https://x.com/user/status/123 This challenges my thinking on X
/brainlift-plugin:bl-update https://example.com/article
/brainlift-plugin:bl-update
```

## Agents

| Agent | Purpose | Tools |
|-------|---------|-------|
| `content-fetcher` | Detect URL type and fetch structured content (tweets, articles, videos, PDFs) | WebFetch, WebSearch |
| `dok-extractor` | Extract DOK1 facts and DOK2 summaries from a source | WebFetch, Read |

## How It Works

The plugin enforces BrainLift bright lines throughout:

- **AI extracts DOK1** (mechanical work) — allowed
- **You write DOK2-4** (synthesis, insights, SPOVs) — enforced
- **Evidence chains checked** before applying changes (DOK4 needs 2+ DOK3s, DOK3 needs DOK1-2)

## Plugin Structure

```
brainlift-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   └── bl-update/
│       └── SKILL.md             # Source → BrainLift update workflow
├── agents/
│   ├── content-fetcher.md       # URL type detection + content fetching
│   └── dok-extractor.md         # DOK1 fact extraction from sources
└── README.md
```

## License

MIT
