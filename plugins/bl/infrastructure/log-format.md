# BrainLift Change Log Format

## Overview

Each BrainLift file has a companion log file that tracks changes over time. The log lives as a sibling file (not inline) to avoid bloating the BrainLift's token count.

## File Naming

```
[brainlift-filename].log.md
```

Example: If the BrainLift is `ai-agents.md`, the log is `ai-agents.log.md`.

## Entry Format

```markdown
## [YYYY-MM-DD] action | Context

- **Action**: [action type]
- **Source**: [source title if applicable]
- **Changes**: [brief list of what changed]
- **Rationale**: [why — user's reasoning if provided]
```

### Action Types

| Action | When Used |
|--------|-----------|
| `ingest` | New source added to Knowledge Tree |
| `revise-dok3` | DOK3 insight added, edited, or removed |
| `revise-dok4` | DOK4 SPOV added, edited, or removed |
| `add-expert` | New expert added to Experts section |
| `refactor-kt` | Knowledge Tree reorganized (categories changed) |
| `lint` | Health check performed |
| `query` | Query exploration saved back to BrainLift |
| `replace-source` | Existing source replaced with better one |

## Rules

1. **Append-only**: Never edit or delete existing entries
2. **Parseable**: Each entry starts with `## [` — use `grep "^## \[" file.log.md` to list entries
3. **Create on first use**: If log file doesn't exist when a skill needs to append, create it with a header:
   ```markdown
   # Change Log: [BrainLift Topic]
   
   Chronological record of changes to this BrainLift.
   ```
4. **Keep entries concise**: 3-5 lines per entry maximum
5. **Include rationale when available**: The "why" behind changes is the most valuable part of the log over time

## Example Log

```markdown
# Change Log: AI Agents on Edge Infrastructure

Chronological record of changes to this BrainLift.

## [2026-03-15] ingest | "Durable Objects Deep Dive" - Rita Kozlov

- **Action**: ingest
- **Source**: Durable Objects Deep Dive - Rita Kozlov (2026-02)
- **Changes**: Added to "Infrastructure Primitives" category with 9 DOK1 facts, DOK2 summary
- **Rationale**: Fills gap in understanding DO concurrency model

## [2026-03-15] revise-dok3 | Updated edge latency insight

- **Action**: revise-dok3
- **Source**: Triggered by "Durable Objects Deep Dive" ingest
- **Changes**: Refined Insight #3 to add concurrency ceiling exception
- **Rationale**: New evidence shows DO throughput caps at ~1000 req/sec per instance

## [2026-03-22] lint | Health check

- **Action**: lint
- **Source**: n/a
- **Changes**: No edits — report generated
- **Rationale**: Weekly maintenance check. Score: 78/100. Flagged 2 stale sources.

## [2026-04-01] query | "What are the cost implications of DO scaling?"

- **Action**: query
- **Source**: n/a
- **Changes**: Saved brainstormed DOK3 insight on DO economics to Insight #5
- **Rationale**: Query revealed cross-source pattern on billing model alignment with agent economics
```

## Usage by Skills

### `/bl:update`
Append an `ingest` entry after successful source addition. If DOK3/4 were also revised, append additional entries for those.

### `/bl:query`
Append a `query` entry only if the user saves something back to the BrainLift (Initial Insight, DOK3 brainstorm, etc.).

### `/bl:lint`
Always append a `lint` entry with the overall score and key findings.
