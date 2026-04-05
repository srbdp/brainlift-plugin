# bl — BrainLift Plugin (v2.0.0)

Technical reference for the `bl` Claude Code plugin. For usage guide and examples, see the [root README](../../README.md).

## Skills

| Skill | Description | Phases |
|-------|-------------|--------|
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

Each BrainLift gets a sibling `[name].log.md` file. Append-only, parseable with `grep "^## \[" file.log.md`. Action types: `ingest`, `revise-dok3`, `revise-dok4`, `add-expert`, `refactor-kt`, `lint`, `query`, `replace-source`. See `infrastructure/log-format.md` for full spec.

## File Structure

```
plugins/bl/
├── .claude-plugin/plugin.json
├── skills/
│   ├── update/SKILL.md
│   ├── query/SKILL.md
│   └── lint/SKILL.md
├── agents/
│   ├── content-fetcher.md
│   ├── dok-extractor.md
│   ├── brainlift-reader.md
│   └── evidence-auditor.md
├── infrastructure/
│   └── log-format.md
└── README.md
```
