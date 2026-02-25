# BrainLift Plugin for Claude Code

Add new sources to existing BrainLifts with a guided 6-phase interview.

## Install

1. Add the marketplace:

```
/marketplace add srbdp-brainlift --source github --repo srbdp/brainlift-plugin
```

2. Install the plugin:

```
/plugin install bl@srbdp-brainlift
```

## Usage

```
/bl:update <url> [your thoughts about this content]
```

Guides you through:

1. **Ingest** - Paste a URL (tweet, article, video, PDF) + optional thoughts
2. **Route** - Matches the source to one of your BrainLifts by Purpose/scope
3. **Extract DOK1** - AI extracts facts; you curate which to keep
4. **Place & DOK2** - Suggests Knowledge Tree category; you write your own summary
5. **DOK3/DOK4 Impact** - Structured questions about how this affects your insights/SPOVs
6. **Apply & Commit** - Previews changes, applies edits, git commits

## Manual Install (alternative)

```bash
# Clone and use directly
claude --plugin-dir ./brainlift-plugin/plugins/bl

# Or copy to global plugins
cp -r plugins/bl ~/.claude/plugins/bl
```

## License

MIT
