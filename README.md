# BrainLift Plugin for Claude Code

A Claude Code plugin that helps you build and maintain [BrainLifts](https://github.com/srbdp/brainlift-plugin) — structured knowledge artifacts for developing real expertise in any domain.

The plugin handles the bookkeeping that kills knowledge bases: extracting facts, generating summaries, tracking evidence chains, flagging stale content, and keeping everything organized. You focus on the curation decisions that actually build expertise — what to include, what challenges your thinking, what patterns you see across sources.

Inspired by Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern: the LLM maintains the knowledge base, you direct the analysis.

## What's a BrainLift?

A BrainLift is a structured markdown file where you develop expertise on a focused topic. It organizes knowledge in layers:

```
Purpose (what's in/out of scope)
DOK4 - SPOVs (your expert positions — "spiky points of view")
DOK3 - Insights (surprising patterns you've found across sources)
────── your expertise is above / external information is below ──────
Experts (people worth following)
Knowledge Tree (organized sources with extracted facts and summaries)
```

Each layer is supported by the one below it. Your SPOVs are backed by insights, which are backed by facts extracted from real sources. No unsupported claims.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and working
- One or more BrainLift `.md` files in a directory (use the [BrainLift template](https://github.com/srbdp/brainlift-plugin) to get started, or run `/new-brainlift` if you have the full BrainLifting project)

## Install

### From the marketplace

```
/marketplace add srbdp-brainlift --source github --repo srbdp/brainlift-plugin
/plugin install bl@srbdp-brainlift
```

### Manual install

```bash
# Option 1: Point Claude Code at the plugin directory
claude --plugin-dir ./brainlift-plugin/plugins/bl

# Option 2: Copy to your global plugins
cp -r plugins/bl ~/.claude/plugins/bl
```

After installing, the three slash commands (`/bl:update`, `/bl:query`, `/bl:lint`) will be available in any Claude Code session.

---

## Commands

### `/bl:update` — Add a source to your BrainLift

The core workflow. Paste a URL and get guided through adding it to your BrainLift.

**Usage:**
```
/bl:update <url> [your thoughts about this content]
```

**Examples:**
```
/bl:update https://x.com/karpathy/status/123456 This changes how I think about knowledge management
/bl:update https://example.com/blog/edge-computing-agents
/bl:update https://arxiv.org/pdf/2401.12345.pdf
/bl:update
```

**What happens:**

1. **Ingest** — You paste a URL (supports tweets, articles, YouTube videos, PDFs). The plugin fetches the content automatically. Add your initial thoughts if you have them.

2. **Route** — The plugin finds your BrainLift files, reads each one's Purpose/scope, and suggests which BrainLift this source belongs in. You pick.

3. **Extract facts** — AI extracts 8-10 DOK1 facts from the source (specific, technical, verifiable). You review the list and choose which facts to keep, edit any that need tweaking, or add ones the AI missed.

4. **Place & summarize** — The plugin suggests a Knowledge Tree category for the source. AI generates a DOK2 summary of the source — you review and edit it to match your understanding. If the source is relevant to other categories too, that gets flagged.

5. **Assess impact** — This is where it gets interesting. The plugin shows your existing insights and SPOVs, then walks through structured questions:
   - Does this source **support** any of your existing insights?
   - Does it **contradict** anything?
   - Does it suggest a **new insight** across sources?
   - Does it affect your **SPOVs**?
   - Should it **replace** a weaker existing source?
   - Should the author be added to your **Experts** list?

   For any DOK3/4 changes, AI brainstorms draft candidates with evidence citations — framed as "here's what I'm seeing, does this resonate?" You accept, edit, or reject.

6. **Apply** — Preview all changes, approve, and they get applied to your BrainLift file. Category summaries get updated. A log entry is appended. If you're in a git repo, changes get committed.

---

### `/bl:query` — Ask questions against your BrainLift

Turn your BrainLift into something you can interrogate. Ask questions and get answers grounded in your accumulated knowledge — with citations.

**Usage:**
```
/bl:query [your question]
```

**Examples:**
```
/bl:query What are the tradeoffs of edge computing for AI agents?
/bl:query Where is my evidence weakest on the stateful agent problem?
/bl:query What do my sources disagree about?
/bl:query
```

**What happens:**

1. **Route** — Matches your question to the right BrainLift based on scope.

2. **Search** — Scans all DOK levels for relevant evidence. Shows you exactly which facts, summaries, insights, and SPOVs are relevant — and what's missing.

3. **Answer** — Synthesizes an answer citing specific sources and DOK levels. Flags contradictions between sources and gaps where your BrainLift has thin coverage.

4. **Compound** — This is the key idea from Karpathy's LLM Wiki: good explorations shouldn't disappear into chat history. After getting your answer, you can:
   - **Save an Initial Insight** on a specific source
   - **Brainstorm a DOK3 insight** — AI drafts a cross-source pattern with evidence for you to review
   - **Brainstorm a DOK4 SPOV** — AI drafts a candidate position with its supporting chain
   - **Flag a gap** — note that you need more sources on a topic
   - **Just explore** — not everything needs to be saved

Every query makes your BrainLift a little richer.

---

### `/bl:lint` — Health-check your BrainLift

The maintenance work that nobody does manually. Lint runs 27 automated checks across 6 dimensions and gives you a scored report.

**Usage:**
```
/bl:lint [path to brainlift file]
/bl:lint
```

**Examples:**
```
/bl:lint ./brainlifts/ai-agents.md
/bl:lint
```

**What you get:**

A health report like this:

```
BrainLift Health Report: AI Agents on Edge Infrastructure
Overall Score: 72/100

Evidence Chain     ██████████░░ 8/10
Token Health       ████████░░░░ 7/10
Semantic Quality   ██████░░░░░░ 6/10
Freshness          ████████████ 10/10
Structural         ██████░░░░░░ 6/10
Completeness       ████████░░░░ 7/10

Critical Issues:
- SPOV #2 has only 1 supporting DOK3 insight (needs 2+)
- DOK3-4 section is 3,200 tokens (above 2,500 functional limit)

Warnings:
- SPOVs #1 and #3 have low semantic distance (too similar)
- Source author "Jane Smith" not in Experts list

Suggestions:
- Knowledge Tree has 12 uncategorized sources — consider adding categories
- 2 sources older than 12 months
```

**The 6 dimensions checked:**

| Dimension | What it checks |
|-----------|---------------|
| **Evidence Chain** | Do your SPOVs have 2+ supporting insights? Do insights have DOK1-2 evidence? Are any claims unsupported? |
| **Token Health** | Are your DOK3-4 sections under 2,500 tokens? Are individual insights too long? Too many overlapping SPOVs? |
| **Semantic Quality** | Can your SPOVs be shuffled (good) or do they read like a narrative (bad)? Any hedging language in SPOVs? Insights that aren't actually surprising? |
| **Freshness** | Sources older than 12 months? Dated claims? Entire categories going stale? |
| **Structural** | Sources missing URLs, DOK1, or DOK2? Knowledge Tree needs reorganizing? |
| **Completeness** | Source authors not tracked as Experts? Echo chamber (all same perspective)? Absolute claims without exceptions? |

After the report, the plugin offers to help fix issues — brainstorming DOK3 insights to fill evidence gaps, suggesting Knowledge Tree restructuring, or identifying stale sources to replace. If you've run lint before, it compares scores to show progress.

---

## How AI and Human Work Together

The plugin draws a clear line between what AI handles and what you decide:

| AI handles (bookkeeping) | You handle (curation) |
|---|---|
| Fetch and parse content from URLs | Choose which sources to add |
| Extract DOK1 facts from sources | Select which facts matter |
| Generate DOK2 summaries | Review and edit summaries |
| Brainstorm DOK3 insights with evidence | Accept, edit, or reject suggestions |
| Brainstorm DOK4 SPOVs as candidates | Decide your actual positions |
| Update category summaries | Approve structural changes |
| Track evidence chains | Decide what to fix |
| Flag stale content and gaps | Prioritize maintenance |
| Format entries and commit changes | Final approval on all edits |

AI brainstorms DOK3/4 suggestions framed as "here's what I'm seeing across your sources — what resonates?" They come with evidence citations so you can evaluate them. You always have the final say.

## Change Logging

Every BrainLift gets a companion `.log.md` file (e.g., `ai-agents.log.md` alongside `ai-agents.md`). Skills automatically append entries after successful edits — what changed, when, from which source, and why. This gives you a timeline of how your thinking evolved.

```markdown
## [2026-03-15] ingest | "Durable Objects Deep Dive" - Rita Kozlov
- **Action**: ingest
- **Changes**: Added to "Infrastructure Primitives" with 9 DOK1 facts
- **Rationale**: Fills gap in understanding DO concurrency model

## [2026-03-22] lint | Health check
- **Action**: lint
- **Changes**: No edits — report generated
- **Rationale**: Score 78/100. Flagged 2 stale sources.
```

## Suggested Workflow

1. **Start with `/bl:update`** — Add 3-5 sources to build your Knowledge Tree foundation
2. **Use `/bl:query`** to explore patterns — "What themes are emerging?" "Where do my sources disagree?"
3. **Save brainstormed insights** when cross-source patterns resonate
4. **Run `/bl:lint` weekly** — catch evidence gaps, stale content, and token bloat before they accumulate
5. **Repeat** — every source you add and every question you ask makes the BrainLift richer

## Plugin Structure

```
brainlift-plugin/
├── plugins/bl/
│   ├── skills/
│   │   ├── update/SKILL.md        # Add sources workflow
│   │   ├── query/SKILL.md         # Query + compound explorations
│   │   └── lint/SKILL.md          # Health check + brainstorm fixes
│   ├── agents/
│   │   ├── content-fetcher.md     # Fetches tweets, articles, videos, PDFs
│   │   ├── dok-extractor.md       # Extracts DOK1-2, brainstorms DOK3
│   │   ├── brainlift-reader.md    # Parses BrainLift file structure
│   │   └── evidence-auditor.md    # 27-check health analysis
│   └── infrastructure/
│       └── log-format.md          # Change log specification
└── README.md
```

## License

MIT
