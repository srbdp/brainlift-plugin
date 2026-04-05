# BrainLift Linking Conventions

How the plugin creates machine-readable evidence chains and enables Obsidian graph view.

## Overview

BrainLifts use a two-layer linking system:
1. **In-file**: Lightweight anchor IDs and `Evidence:` lines inside BrainLift files
2. **In-index**: Full interlinked evidence graph in `index.md` (see `infrastructure/index-format.md`)

Both layers use Obsidian-compatible wikilink syntax so the workspace works as an Obsidian vault with graph view.

---

## Link Syntax

All links use Obsidian wikilinks:

| Syntax | Use Case | Example |
|--------|----------|---------|
| `[[filename]]` | Cross-BrainLift reference | `[[ai-agents]]` |
| `[[filename#heading]]` | Heading-level link | `[[ai-agents#I-1]]` |
| `[[filename#heading\|alias]]` | Aliased display text | `[[ai-agents#I-1\|Insight 1]]` |

**Filenames** omit the `.md` extension (Obsidian default).

---

## Anchor IDs

SPOVs and Insights get stable, parseable ID prefixes that serve as link targets.

### SPOV IDs

Format: `**SPOV-N**:` where N is a monotonically increasing integer.

```markdown
**DOK4 - SPOV**
- **SPOV-1**: Infrastructure-level actor models will replace...
  Evidence: [[ai-agents#I-1]], [[ai-agents#I-3]]
- **SPOV-2**: Edge-native AI agents are the next...
  Evidence: [[ai-agents#I-2]], [[ai-agents#I-4]]
```

### Insight IDs

Format: `**I-N**:` where N is a monotonically increasing integer.

```markdown
**DOK3 - Insights**
- **I-1**: The single-threaded concurrency model creates...
  Evidence: [[ai-agents#Durable Objects Deep Dive - Rita Kozlov (2026-02)]]
- **I-2**: Cost structures for edge compute align...
  Evidence: [[ai-agents#Edge Cost Analysis - Priya Sharma (2026-01)]]
```

### Source Headings

Sources already use bold titles as their heading: `**Source Title - Author (Date)**`. No new ID format needed — these are already valid link targets.

### ID Rules

1. **Stable**: IDs never renumber. `SPOV-3` means "the SPOV assigned ID 3 when created", not "the third SPOV in order."
2. **Monotonic**: New IDs use `max(existing IDs) + 1`. If SPOV-1 and SPOV-3 exist (SPOV-2 was deleted), the next SPOV is SPOV-4.
3. **Assigned by plugin**: The plugin assigns IDs during `/bl:update` Phase 6 and `/bl:query` Phase 4 when new DOK3/4 content is created.
4. **Retro-fitted by lint**: `/bl:lint` offers to add IDs to existing BrainLifts that lack them (migration helper).

---

## Evidence Lines

Each SPOV and Insight has an optional `Evidence:` line immediately after its text. This line makes the evidence chain machine-readable.

### Format

```
  Evidence: [[link1]], [[link2]], [[link3]]
```

- Indented 2 spaces (aligns under the list item text)
- Starts with `Evidence:` prefix (case-sensitive, for reliable parsing)
- Comma-separated Obsidian wikilinks
- One line only

### What Evidence Lines Point To

| Element | Evidence Links To | Example |
|---------|------------------|---------|
| SPOV | Supporting Insight IDs | `Evidence: [[ai-agents#I-1]], [[ai-agents#I-3]]` |
| Insight | Supporting Source headings | `Evidence: [[ai-agents#Durable Objects Deep Dive - Rita Kozlov (2026-02)]]` |

### Maintenance

- **Auto-generated**: The plugin creates Evidence lines when new DOK3/4 content is added via `/bl:update` or `/bl:query`
- **Auto-updated**: When DOK3/4 content is revised, Evidence lines are updated
- **Migration**: `/bl:lint` Option F generates Evidence lines for existing content by tracing prose-based evidence chains
- **Human editable**: Users can manually adjust Evidence lines if the plugin's inference is wrong

---

## Where Links Appear

| Location | Links Present? | Who Writes |
|----------|---------------|------------|
| BrainLift files — Evidence lines | Yes (wikilinks) | Plugin auto-generates |
| BrainLift files — DOK1/2 content | No | N/A |
| index.md — everywhere | Yes (wikilinks) | Plugin auto-generates |
| Log files | No | N/A — logs stay simple text |
| CLAUDE.md | No | N/A |

---

## Orphan and Broken Link Definitions

### Orphan Source
A source in the Knowledge Tree that is not referenced by any Insight's `Evidence:` line. Not an error — sources may be valuable without yet contributing to an Insight — but a signal for potential synthesis work.

### Orphan Insight
An Insight not referenced by any SPOV's `Evidence:` line. May indicate an Insight that hasn't yet contributed to a higher-level position.

### Orphan SPOV
A SPOV with no `Evidence:` line, or whose `Evidence:` line is empty. Violates the invariant that SPOVs must be supported by 2+ Insights.

### Broken Link
A wikilink whose target does not exist:
- `[[filename]]` where `filename.md` is not in the lifts directory
- `[[filename#heading]]` where the heading is not found in the target file

### Stale Link
A wikilink pointing to a heading that was renamed or removed. Detected by comparing link targets against actual headings in the target file.

---

## Parsing Patterns

For tools that need to extract links programmatically:

| Pattern | Regex | Captures |
|---------|-------|----------|
| SPOV ID | `\*\*SPOV-(\d+)\*\*:` | ID number |
| Insight ID | `\*\*I-(\d+)\*\*:` | ID number |
| Evidence line | `^\s+Evidence:\s+(.+)$` | Comma-separated link list |
| Wikilink | `\[\[([^\]\|]+)(?:\|([^\]]+))?\]\]` | Target, optional alias |
| Heading link | `\[\[([^#\]\|]+)#([^\]\|]+)(?:\|([^\]]+))?\]\]` | File, heading, optional alias |
