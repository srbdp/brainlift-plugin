---
description: Health-check your BrainLift — audit evidence chains, token counts, semantic quality, freshness, and completeness
argument-hint: [path to brainlift file]
---

You're helping the user audit and maintain their BrainLift. This is the maintenance work that kills knowledge bases when left undone — and the work that LLMs are perfectly suited to handle.

## Philosophy

From Karpathy's LLM Wiki pattern: "Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass."

Lint is the BrainLift equivalent. It catches structural problems, evidence gaps, staleness, and quality issues — then brainstorms fixes the human can accept or reject.

## Core Rules

### What AI Does
- Parse and analyze the BrainLift structure
- Run systematic health checks across 6 categories
- Score and report findings
- Brainstorm candidate DOK3/4 content to fill evidence gaps
- Apply approved changes

### What Human Does
- Review the health report
- Prioritize which issues to fix
- Accept, edit, or reject brainstormed fixes
- Decide on structural changes (reorganization, source replacement)

---

## 3-Phase Workflow

### Phase 1: Parse & Analyze

**If path provided in `$ARGUMENTS`**: Use it directly.

**If no path provided**: Find BrainLift files and let user choose:
1. Use Glob to find `*.md` files (exclude `*.log.md`)
2. Read each file's Purpose section
3. Present list:
   ```
   Which BrainLift should I audit?

   1. **[Topic A]** — [N] sources, last modified [date]
   2. **[Topic B]** — [N] sources, last modified [date]
   ```

**After selection**: Spawn two agents in parallel:

1. **brainlift-reader** to parse the full structure:
   ```
   Task(
     description: "Parse BrainLift for lint",
     prompt: "You are a brainlift-reader agent.
   
     Read and parse this BrainLift file: [path]
   
     Return the full structured parse report with all sections,
     counts, and token estimates."
   )
   ```

2. **evidence-auditor** to analyze health (after reader completes):
   ```
   Task(
     description: "Audit BrainLift health",
     prompt: "You are an evidence-auditor agent.
   
     Analyze this BrainLift for health issues.
   
     BrainLift File Path: [path]
     BrainLift Parse Report:
     [paste the brainlift-reader output]
   
     Domain Context: [fast-moving / stable — infer from topic]
   
     Run all 6 audit categories and return the full health report
     with scores, issues, and brainstormed fixes."
   )
   ```

---

### Phase 2: Report

Present the health report in a scannable format:

```
# BrainLift Health Report: [Topic]

**Overall Score: [N]/100** [Excellent/Good/Needs Work/Weak/Critical]

Evidence Chain     [██████████░░] [N]/10
Token Health       [████████░░░░] [N]/10
Semantic Quality   [██████░░░░░░] [N]/10
Freshness          [████████████] [N]/10
Structural         [██████░░░░░░] [N]/10
Completeness       [████████░░░░] [N]/10

---

## Critical Issues (fix these)

1. **[Issue]**: [Specific description with quoted BrainLift content]
   → Suggested fix: [actionable suggestion]

2. **[Issue]**: [description]
   → Suggested fix: [suggestion]

---

## Warnings (consider these)

1. **[Warning]**: [description]
2. **[Warning]**: [description]

---

## Suggestions (nice to have)

1. [Suggestion]
2. [Suggestion]

---

## Evidence Chain Map

**DOK4 SPOVs:**
- SPOV #1: "[text]" → Supported by Insights #[N], #[N] ✓
- SPOV #2: "[text]" → Only 1 insight supports this ⚠️
- SPOV #3: "[text]" → No clear insight support ✗

**DOK3 Insights:**
- Insight #1: "[text]" → Evidence from [Source A], [Source B] ✓
- Insight #2: "[text]" → Single-source evidence ⚠️
- Insight #3: "[text]" → No traceable DOK1-2 support ✗
```

**Adapt the report to BrainLift maturity**:
- **Early stage** (< 5 sources, no DOK3-4): Focus on structural completeness and Knowledge Tree organization. Don't penalize missing DOK3-4.
- **Developing** (5-15 sources, some DOK3): Full audit but weight evidence chain and semantic quality higher.
- **Mature** (15+ sources, established DOK3-4): Full audit with strict standards. Focus on freshness, token optimization, and evidence chain integrity.

---

### Phase 3: Action Plan & Brainstorm Fixes

After presenting the report, offer to help fix issues:

```
Want to tackle any of these? I can help with:

A. **Fix evidence gaps** — I'll brainstorm DOK3 insights to strengthen
   weak evidence chains

B. **Optimize tokens** — I'll suggest which content to merge, condense,
   or move between DOK levels

C. **Restructure Knowledge Tree** — I'll suggest new categories based
   on your current sources

D. **Flag stale content** — I'll identify specific sources and claims
   that may need refreshing

E. **Run a deeper check** — Ask me specific questions about any
   finding in the report

Or just take the report and work on it yourself — that's fine too.
```

**If user chooses A (Evidence gaps)**:

For each unsupported or weakly-supported DOK3/4, brainstorm candidates:

```
SPOV #2 needs more DOK3 support. Based on your existing sources,
here are candidate insights that could support it:

**Candidate 1**: [2-3 sentence draft insight]
Supporting evidence:
  - [Source A] DOK1: "[fact]"
  - [Source B] DOK2: "[pattern]"

**Candidate 2**: [2-3 sentence draft insight]
Supporting evidence:
  - [Source C] DOK1: "[fact]"
  - [Source D] DOK1: "[fact]"

Do either of these resonate? Edit to match your thinking,
or reject if the evidence doesn't support it.
```

For unsupported DOK3 insights, suggest DOK1-2 evidence that might exist:
```
Insight #3 lacks DOK1-2 support. Either:
- Find sources that provide evidence (try `/bl:update` with relevant URLs)
- Revise the insight to match what your sources actually say
- Remove it if it's not supported

Based on your existing sources, the closest evidence I found:
- [Source X] DOK1: "[fact that partially supports]"
```

**If user chooses B (Token optimization)**:
- Identify DOK3/4 content that can be condensed
- Flag similar/overlapping insights that could be merged
- Suggest moving factual details from DOK3 down to DOK1-2
- Present before/after with token counts

**If user chooses C (Restructure KT)**:
- Analyze current sources by theme
- Suggest 2-4 category groupings with reasoning
- Present proposed structure for approval
- On approval, reorganize using Edit tool (mechanical, not content changes)

**If user chooses D (Stale content)**:
- List each stale source with age and topic
- For each, suggest: keep (still foundational), replace (find newer source), or remove
- If replacing, suggest search terms for finding newer sources

**After any changes**:
1. Apply edits to BrainLift using Edit tool
2. Show git diff for review
3. Append log entry:
   ```markdown
   ## [YYYY-MM-DD] lint | Health check
   
   - **Action**: lint
   - **Source**: n/a
   - **Changes**: [list changes if any were applied]
   - **Rationale**: Health score [N]/100. [Key findings summary.]
   ```

---

## Important Rules

### Report Accuracy
- Every finding must reference specific content from the BrainLift (quote it)
- Don't invent issues — if the BrainLift is genuinely healthy, say so
- Scoring must follow the evidence-auditor's formula consistently

### Brainstorm Framing
- All DOK3/4 suggestions are brainstorms, not prescriptions
- Always show the evidence chain so human can evaluate
- "None of these land" is a valid response — don't push

### Maturity Awareness
- Don't score an early BrainLift (< 5 sources) the same as a mature one
- Flag the maturity stage in the report header
- Adjust expectations and skip irrelevant checks

### Non-Destructive
- Never delete content without explicit user approval
- Reorganization is mechanical (moving content) not editorial (changing content)
- Show diff before committing any changes

### Log Always
- Always append a lint log entry, even if no changes were made
- The log entry for a no-changes lint still records the score and key findings

### Comparison with Previous Lints
- If a previous lint entry exists in the log, compare scores:
  ```
  Previous lint (2026-03-22): 68/100
  Current lint:               78/100  (+10)
  
  Improved: Evidence Chain (+2), Token Health (+1)
  Declined: Freshness (-1) — 2 sources aged past threshold
  ```
