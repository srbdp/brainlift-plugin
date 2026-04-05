# BrainLift Index Format

Schema and maintenance rules for the auto-generated `index.md` workspace catalog.

## Overview

`index.md` is a persistent, auto-generated file that catalogs all BrainLifts in a workspace. It serves as:

1. **Navigation layer**: Skills read it FIRST to find relevant BrainLifts without parsing every file
2. **Evidence graph**: Machine-readable map of SPOV -> Insight -> Source chains
3. **Source registry**: Global view of all sources across all BrainLifts
4. **Cross-BrainLift map**: Where topics and authors overlap between BrainLifts
5. **Obsidian graph hub**: All references use wikilinks, making it a connected hub in graph view

## File Location

```
[brainlift-root]/index.md
```

Sibling of `CLAUDE.md`, `lifts/`, `logs/`, `sources/`.

## Auto-Generated Marker

Line 1 must be:
```html
<!-- brainlift-index: auto-generated, do not edit manually -->
```

Line 2 records the last update:
```html
<!-- last-updated: 2026-04-06T14:30:00Z -->
```

Skills check for this marker to confirm the file is plugin-managed. If a user creates their own `index.md` without this marker, the plugin will not overwrite it.

---

## Schema

```markdown
<!-- brainlift-index: auto-generated, do not edit manually -->
<!-- last-updated: [ISO 8601 timestamp] -->
# BrainLift Index

## BrainLifts

### [[filename|Display Title]]
- **Purpose**: [In Scope statement]
- **Owner**: [name]
- **Stats**: [N] sources | [N] insights | [N] SPOVs | [N] categories | [N] blueprints
- **Categories**: [comma-separated category names]
- **Last Updated**: [YYYY-MM-DD]

[repeat for each BrainLift]

---

## Evidence Graph

### [[filename|Display Title]]

**SPOVs:**
| ID | SPOV | Supporting Insights | Status |
|----|------|-------------------|--------|
| [[filename#SPOV-N\|SPOV-N]] | [truncated text...] | [[filename#I-N\|I-N]], [[filename#I-N\|I-N]] | [status] |

**Insights:**
| ID | Insight | Supporting Sources | Status |
|----|---------|-------------------|--------|
| [[filename#I-N\|I-N]] | [truncated text...] | [Source A], [Source B] | [status] |

[repeat for each BrainLift]

---

## Source Registry

| Source | Author | Date | BrainLift | Category |
|--------|--------|------|-----------|----------|
| [[filename#Source Title - Author (Date)\|Source Title]] | [Author] | [Date] | [[filename]] | [Category] |

[one row per source across all BrainLifts]

---

## Cross-BrainLift Connections

[only present when connections have been flagged]

- [[brainlift-a#I-N]] ([topic]) relates to [[brainlift-b#I-N]] ([topic])
- Author **[Name]** appears in: [[brainlift-a]] ([N] sources), [[brainlift-b]] ([N] sources)

---

## Workspace Stats
- **Total BrainLifts**: [N]
- **Total Sources**: [N]
- **Total Insights**: [N]
- **Total SPOVs**: [N]
- **Last Full Rebuild**: [YYYY-MM-DD]
```

---

## Evidence Status Labels

| Status | Meaning |
|--------|---------|
| `Supported` | SPOV has 2+ insight links, each with 1+ source links |
| `Weak` | SPOV has <2 insight links, or insight has single-source evidence |
| `Unsupported` | No `Evidence:` line, or evidence links are empty/broken |
| `Unknown` | BrainLift has no anchor IDs yet (pre-migration) |

When a BrainLift has no anchor IDs or Evidence lines, the plugin uses prose-based inference from the evidence-auditor to populate the Evidence Graph with `Unknown` status and best-guess connections.

---

## Maintenance Triggers

| Event | Index Action | Scope |
|-------|-------------|-------|
| `/bl:init` (fresh) | Full creation | Empty index with workspace header |
| `/bl:init` (re-run on existing) | Full rebuild | Parse all BrainLifts, generate complete index |
| `/bl:update` (source added) | Incremental update | Catalog stats, Evidence Graph, Source Registry for affected BrainLift; Cross-BrainLift Connections if flagged; Workspace Stats |
| `/bl:query` (compound save) | Incremental update | Catalog stats, Evidence Graph for affected BrainLift if DOK3/4 changed |
| `/bl:lint` (fixes applied) | Incremental update | Evidence Graph for affected BrainLift; recalculate statuses |
| `/bl:lint` (migration: anchor IDs added) | Full rebuild | Reparse affected BrainLift with new link data |

---

## Rebuild vs. Incremental Update

### Full Rebuild
- Re-parse **all** BrainLift files via brainlift-reader
- Regenerate entire index.md from scratch
- Used by: `/bl:init`, and when lint detects staleness

### Incremental Update
- Re-parse only the **changed** BrainLift file
- Update its sections in index.md (catalog entry, evidence graph, source registry rows)
- Recalculate Workspace Stats
- Used by: `/bl:update`, `/bl:query`, `/bl:lint`

### How to Perform an Incremental Update

1. Parse the changed BrainLift file via brainlift-reader
2. Read the existing `index.md`
3. Replace the BrainLift's catalog entry (under `## BrainLifts`)
4. Replace the BrainLift's evidence graph section (under `## Evidence Graph`)
5. Remove old Source Registry rows for this BrainLift, insert new rows
6. Update Cross-BrainLift Connections if new flags were raised
7. Recalculate Workspace Stats totals
8. Update the `<!-- last-updated: -->` timestamp
9. Write the updated `index.md`

---

## Staleness Detection

The `<!-- last-updated: -->` timestamp records when the index was last written.

During `/bl:lint`, compare this timestamp against the modification times of all BrainLift files in `lifts/`. If any BrainLift file was modified more recently than the index timestamp:

```
Index is stale — [N] BrainLift(s) modified since last index update.
Want me to rebuild the index?
```

---

## Empty Index Template

Created by `/bl:init` when no BrainLifts exist yet:

```markdown
<!-- brainlift-index: auto-generated, do not edit manually -->
<!-- last-updated: [ISO 8601 timestamp] -->
# BrainLift Index

## BrainLifts

_No BrainLifts yet. Create a `.md` file in `lifts/` with a Purpose section, then run `/bl:update` to add sources._

---

## Evidence Graph

_Evidence graph will appear here as BrainLifts develop DOK3-4 content._

---

## Source Registry

| Source | Author | Date | BrainLift | Category |
|--------|--------|------|-----------|----------|

---

## Workspace Stats
- **Total BrainLifts**: 0
- **Total Sources**: 0
- **Total Insights**: 0
- **Total SPOVs**: 0
- **Last Full Rebuild**: [YYYY-MM-DD]
```
