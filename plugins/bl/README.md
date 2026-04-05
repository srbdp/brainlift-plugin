# bl — BrainLift Plugin (v2.1.0)

Technical reference for the `bl` Claude Code plugin. For usage guide and examples, see the [root README](../../README.md).

## Skills

| Skill | Description | Phases |
|-------|-------------|--------|
| `/bl:init` | Set up workspace in current directory — creates `lifts/`, `logs/`, `sources/`, `CLAUDE.md`, `~/.brainlift` | 5: Confirm → Detect existing → Scaffold → Migrate → Summary |
| `/bl:update <url> [thoughts]` | Add a source to a BrainLift | 6: Ingest → Route → Extract DOK1 → Place & DOK2 → Impact Assessment → Apply |
| `/bl:query [question]` | Query BrainLift knowledge, compound explorations | 4: Route → Evidence Search → Synthesize → Compound |
| `/bl:lint [path]` | Health-check with scored report | 3: Parse & Analyze → Report → Action Plan |

## Agents

| Agent | Role | Tools | Used By |
|-------|------|-------|---------|
| `content-fetcher` | URL type detection + content extraction (tweets via FxTwitter API, articles, YouTube, PDFs) | WebFetch, WebSearch | update |
| `dok-extractor` | DOK1 fact extraction, DOK2 summary generation, cross-source DOK3 brainstorming | WebFetch, Read | update |
| `brainlift-reader` | Parse BrainLift `.md` files into structured sections with fuzzy header matching | Read, Glob | query, lint |
| `evidence-auditor` | 27-check health analysis across 6 weighted categories, scored 0-100 | Read | lint |

## Discovery Protocol

All skills (except init) use a three-step cascade to find BrainLift files automatically. See `infrastructure/discovery.md` for full spec.

| Step | How | When it works |
|------|-----|---------------|
| 1. CLAUDE.md marker | Walk up from `$CWD` looking for `<!-- brainlift-root -->` | User is in their BrainLift workspace |
| 2. ~/.brainlift pointer | Read one-line file with absolute path to workspace | User is in a different project |
| 3. Ask user | Prompt once, offer to save as `~/.brainlift` | First time, no init yet |

After discovery, resolves `lifts/` directory (falls back to root if `lifts/` doesn't exist).

## Workspace Structure

Created by `/bl:init`:

```
workspace/
├── CLAUDE.md       # Marker + project instructions (auto-loaded by Claude Code)
├── lifts/          # BrainLift .md files only
├── logs/           # [name].log.md change logs
└── sources/        # Optional clipped raw content
```

## Audit Categories (evidence-auditor)

| Category | Weight | What's Checked |
|----------|--------|---------------|
| Evidence Chain | 25% | DOK4→DOK3→DOK1-2 support hierarchy, single-source insights, unsupported claims |
| Token Health | 15% | DOK3-4 token count vs thresholds (peak <1000, functional <2500, danger >5000) |
| Semantic Quality | 20% | SPOV narrative flow, semantic distance, hedging language, obvious insights |
| Freshness | 15% | Source age, dated claims, all-stale categories |
| Structural | 10% | Missing URLs/DOK1/DOK2, empty Initial Insights, KT needing refactoring |
| Completeness | 15% | Untracked experts, echo chamber, missing qualifications, no blueprints |

## AI/Human Division of Labor

**AI generates**: DOK1 extraction, DOK2 summaries, DOK3/4 brainstorm candidates (with evidence citations), category summaries, formatting, log entries, health reports.

**Human curates**: Which facts to keep, DOK2 review/edit, DOK3/4 accept/edit/reject, what to include/exclude, structural decisions, all final approvals.

DOK3/4 brainstorms are framed as exploratory ("here's what I'm seeing — what resonates?"), not prescriptive.

## Change Log System

Each BrainLift gets a `[name].log.md` in the `logs/` directory (falls back to sibling file in flat setups). Append-only, parseable with `grep "^## \[" file.log.md`. Action types: `ingest`, `revise-dok3`, `revise-dok4`, `add-expert`, `refactor-kt`, `lint`, `query`, `replace-source`. See `infrastructure/log-format.md` for full spec.

## File Structure

```
plugins/bl/
├── .claude-plugin/plugin.json
├── skills/
│   ├── init/SKILL.md
│   ├── update/SKILL.md
│   ├── query/SKILL.md
│   └── lint/SKILL.md
├── agents/
│   ├── content-fetcher.md
│   ├── dok-extractor.md
│   ├── brainlift-reader.md
│   └── evidence-auditor.md
├── infrastructure/
│   ├── discovery.md
│   └── log-format.md
└── README.md
```
