# bl — BrainLift Plugin (v2.2.0)

Technical reference for the `bl` Claude Code plugin. For usage guide and examples, see the [root README](../../README.md).

## What's New in v2.2

- **`index.md`** — Auto-generated workspace catalog with evidence graph, source registry, and cross-BrainLift connections. Read by skills for fast navigation. Compatible with Obsidian graph view.
- **Evidence linking** — Machine-readable evidence chains via `Evidence:` wikilinks on SPOVs and Insights. Anchor IDs (`SPOV-N`, `I-N`) enable granular Obsidian graph view.
- **Index-first routing** — `/bl:query` and `/bl:update` read `index.md` first for faster BrainLift selection.
- **Link integrity checks** — `/bl:lint` validates evidence links, detects orphans and broken links.
- **Migration helpers** — `/bl:lint` Option F adds anchor IDs and evidence links to existing BrainLifts.
- **Idempotent init** — Running `/bl:init` in an already-initialized workspace adds missing v2.2 components (index.md).

## Skills

| Skill | Description | Phases |
|-------|-------------|--------|
| `/bl:init` | Set up workspace — creates `lifts/`, `logs/`, `sources/`, `CLAUDE.md`, `index.md`, `~/.brainlift`. Idempotent: re-running adds missing components. | 5: Confirm → Detect existing → Scaffold (+ Upgrade Check) → Migrate (+ Index Rebuild) → Summary |
| `/bl:update <url> [thoughts]` | Add a source to a BrainLift | 6: Ingest → Route (index-first) → Extract DOK1 → Place & DOK2 → Impact Assessment (+ cross-BrainLift) → Apply (+ anchor IDs, evidence links, index update) |
| `/bl:query [question]` | Query BrainLift knowledge, compound explorations | 4: Route (index-first) → Evidence Search → Synthesize (wikilink citations) → Compound (+ evidence links, index update) |
| `/bl:lint [path]` | Health-check with scored report | 3+: Parse & Analyze → Link Validation → Report (+ Link Integrity) → Action Plan (+ Options F, G) |

## Agents

| Agent | Role | Tools | Used By |
|-------|------|-------|---------|
| `content-fetcher` | URL type detection + content extraction (tweets via FxTwitter API, articles, YouTube, PDFs) | WebFetch, WebSearch | update |
| `dok-extractor` | DOK1 fact extraction, DOK2 summary generation, cross-source DOK3 brainstorming | WebFetch, Read | update |
| `brainlift-reader` | Parse BrainLift `.md` files into structured sections with fuzzy header matching. Extracts anchor IDs, Evidence links, wikilink inventory, orphan detection. | Read, Glob | query, lint, init (index rebuild) |
| `evidence-auditor` | Health analysis across 6 weighted categories (scored 0-100). Validates machine-readable evidence links alongside prose analysis. | Read | lint |

## Discovery Protocol

All skills (except init) use a three-step cascade to find BrainLift files automatically. See `infrastructure/discovery.md` for full spec.

| Step | How | When it works |
|------|-----|---------------|
| 1. CLAUDE.md marker | Walk up from `$CWD` looking for `<!-- brainlift-root -->` | User is in their BrainLift workspace |
| 2. ~/.brainlift pointer | Read one-line file with absolute path to workspace | User is in a different project |
| 3. Ask user | Prompt once, offer to save as `~/.brainlift` | First time, no init yet |

After discovery, resolves `lifts/` directory (falls back to root if `lifts/` doesn't exist) and checks for `index.md` for fast routing.

## Workspace Structure

Created by `/bl:init`:

```
workspace/
├── CLAUDE.md       # Marker + project instructions (auto-loaded by Claude Code)
├── index.md        # Auto-generated catalog, evidence graph, source registry
├── lifts/          # BrainLift .md files only
├── logs/           # [name].log.md change logs
└── sources/        # Optional clipped raw content
```

## Index System

`index.md` is auto-generated and auto-maintained by the plugin. See `infrastructure/index-format.md` for schema.

| Section | Contents |
|---------|----------|
| BrainLifts catalog | Purpose, stats, categories, last updated per BrainLift |
| Evidence Graph | SPOV → Insight → Source chains with support status |
| Source Registry | All sources across all BrainLifts |
| Cross-BrainLift Connections | Topic/author overlap between BrainLifts |
| Workspace Stats | Aggregate counts |

**Maintenance**: Updated incrementally after `/bl:update` and `/bl:query` (compound saves). Full rebuild on `/bl:init` (re-run) and `/bl:lint` Option G.

## Linking Conventions

See `infrastructure/linking.md` for full spec.

| Element | ID Format | Evidence Links To |
|---------|-----------|------------------|
| SPOV | `**SPOV-N**:` | Supporting Insight IDs |
| Insight | `**I-N**:` | Supporting Source headings |
| Source | Existing bold title | (link target, not link source) |

Evidence lines use Obsidian wikilinks: `Evidence: [[filename#I-1]], [[filename#I-3]]`

## Audit Categories (evidence-auditor)

| Category | Weight | What's Checked |
|----------|--------|---------------|
| Evidence Chain | 25% | DOK4→DOK3→DOK1-2 support hierarchy, single-source insights, unsupported claims, broken/missing evidence links, orphan sources |
| Token Health | 15% | DOK3-4 token count vs thresholds (peak <1000, functional <2500, danger >5000) |
| Semantic Quality | 20% | SPOV narrative flow, semantic distance, hedging language, obvious insights |
| Freshness | 15% | Source age, dated claims, all-stale categories |
| Structural | 10% | Missing URLs/DOK1/DOK2, empty Initial Insights, KT needing refactoring |
| Completeness | 15% | Untracked experts, echo chamber, missing qualifications, no blueprints |

## AI/Human Division of Labor

**AI generates**: DOK1 extraction, DOK2 summaries, DOK3/4 brainstorm candidates (with evidence citations and wikilinks), category summaries, formatting, log entries, health reports, index.md, evidence links.

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
│   ├── index-format.md
│   ├── linking.md
│   └── log-format.md
└── README.md
```
