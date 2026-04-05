---
description: Ask questions against your BrainLift — get cited answers, discover gaps, and compound your explorations back into knowledge
argument-hint: [question about your domain]
---

You're helping the user query their BrainLift — asking questions against their accumulated knowledge and compounding the exploration back into the artifact.

## Philosophy

This skill implements Karpathy's key insight: **explorations should compound in the knowledge base.** A question you ask today shouldn't disappear into chat history — the answer, the connections discovered, the gaps identified should all feed back into the BrainLift. Every query makes the BrainLift richer.

## Core Rules

### What AI Does
- Search BrainLift content for relevant evidence
- Synthesize answers citing specific DOK levels and sources
- Flag gaps, contradictions, and patterns
- Draft Initial Insights, DOK3 brainstorms, and DOK4 candidates for user to curate
- Apply approved changes to the BrainLift file
- Append log entries

### What Human Does
- Ask the questions that matter
- Decide what to save back to the BrainLift
- Accept, edit, or reject AI-drafted DOK3/4 suggestions
- Direct further investigation

### BrainLift File Structure

```
Owner(s)
Purpose (In Scope / Out of Scope)
───────────────────────────────────
DOK4 - SPOV
DOK3 - Insights
  └─ DOK3 - Blueprints
─── BRIGHT LINE: above = owner's expertise, below = external information ───
Experts
DOK2 - Knowledge Tree
  ├─ Category (with Category Summary)
  │   └─ Sources
  │       ├─ DOK2 - Summary
  │       ├─ DOK1 - Facts
  │       ├─ Initial Insights
  │       └─ Link to Content
───────────────────────────────────
```

---

## 4-Phase Workflow

### Phase 1: Route to BrainLift

Parse `$ARGUMENTS` for the user's question.

**If no question provided**: Ask the user what they want to explore.

**Find BrainLift files** using the Discovery Protocol (see `infrastructure/discovery.md`):
1. Walk up from `$CWD` looking for `CLAUDE.md` with `<!-- brainlift-root -->` marker
2. If not found, check `~/.brainlift` pointer file for the workspace path
3. If neither exists, ask the user for a directory path (offer to save as `~/.brainlift` for next time)
4. Resolve lifts directory: `[root]/lifts/` if it exists, else `[root]/` as fallback

**Find the right BrainLift**:
1. Glob `[lifts_dir]/*.md` (exclude `*.log.md`)
2. Read each file's opening to extract **Purpose** (In Scope / Out of Scope)
3. Score relevance of the question against each BrainLift's scope
4. Present ranked matches:

```
Which BrainLift should I search?

1. **[BrainLift Topic]** - [In Scope summary]
   Relevance: [Why this question fits]

2. **[BrainLift Topic]** - [In Scope summary]
   Relevance: [Partial fit]

Which one? (or specify a file path)
```

**After selection**: Spawn a brainlift-reader agent to parse the full BrainLift:

```
Task(
  description: "Parse BrainLift structure",
  prompt: "You are a brainlift-reader agent.

  Read and parse this BrainLift file: [path]

  Return the full structured parse report with all sections,
  counts, and token estimates."
)
```

---

### Phase 2: Evidence Search

With the parsed BrainLift, search for content relevant to the question.

**Search across all DOK levels**:

1. **DOK1-2 (Knowledge Tree)**: Scan facts and summaries for relevant evidence. Note which sources and categories contain relevant material.

2. **DOK3 (Insights)**: Check if any existing insights address the question directly or tangentially.

3. **DOK4 (SPOVs)**: Check if any SPOVs take a position on the question's topic.

4. **Initial Insights**: Check for partially-developed thinking relevant to the question.

**Present the evidence map**:

```
Here's what your BrainLift has on this topic:

**Directly relevant:**
- DOK1 from [Source A]: "[fact snippet]"
- DOK1 from [Source B]: "[fact snippet]"
- DOK2 from [Source C]: "[summary snippet]"
- DOK3 Insight #2: "[insight snippet]"

**Tangentially related:**
- DOK1 from [Source D]: "[fact snippet]"
- DOK4 SPOV #1 touches on this: "[spov snippet]"

**Gaps — your BrainLift doesn't cover:**
- [Aspect X of the question has no evidence]
- [Aspect Y has only 1 source — thin coverage]
```

---

### Phase 3: Synthesize Answer

Using the evidence found, answer the user's question.

**Answer structure**:

```
## Answer

[2-4 paragraph synthesis answering the question, weaving together evidence
from multiple DOK levels. Every claim cites its source.]

**Evidence used:**
- [Source A] DOK1: [concept]
- [Source B] DOK2: [pattern]
- [Insight #N]: [synthesis]

**Contradictions found:**
- [Source A says X, but Source B says Y — this tension is unresolved]

**Gaps identified:**
- [Your BrainLift doesn't address [aspect] — this might be worth a new source]
- [Only 1 source covers [topic] — thin evidence base]

**Related questions your BrainLift could answer:**
- [Question 1]
- [Question 2]
```

**Important**: Clearly distinguish between what the BrainLift's evidence supports vs. general knowledge. If the BrainLift is thin on a topic, say so — don't fill in with external knowledge unless the user asks.

---

### Phase 4: Compound

This is the key Karpathy-inspired phase — making explorations compound.

```
Would you like to save anything from this exploration?

1. **Save Initial Insight** — I'll draft an insight for a specific source's
   "Initial Insights" section based on what we discovered.

2. **Brainstorm DOK3 Insight** — I see a cross-source pattern worth developing.
   I'll draft a candidate insight with evidence citations for you to review.

3. **Brainstorm DOK4 SPOV** — This exploration points toward a position.
   I'll draft a candidate SPOV with its supporting chain for you to consider.

4. **Flag a gap** — Note that this question revealed missing coverage.
   I can suggest sources to investigate.

5. **Just exploring** — Nothing to save, and that's fine.
```

**If user chooses 1 (Initial Insight)**:
- Ask which source the insight relates to
- Draft the Initial Insight text based on the exploration
- Present: "Here's a draft Initial Insight for [Source]. Edit or approve:"
- On approval, insert into the source's Initial Insights section using Edit tool

**If user chooses 2 (DOK3 Brainstorm)**:
- Draft a candidate insight that synthesizes evidence from 2+ sources
- Show the evidence chain:
  ```
  Here's a pattern I'm seeing — does this resonate?

  **Draft Insight**: [2-3 sentence insight]

  **Supporting evidence**:
  - [Source A] DOK1: "[fact]"
  - [Source B] DOK2: "[pattern]"
  - [Source C] DOK1: "[fact]"

  This is a brainstorm, not a finished insight. Edit it to match
  your thinking, or reject if it doesn't land.
  ```
- If user edits/approves, insert into DOK3 Insights section

**If user chooses 3 (DOK4 Brainstorm)**:
- Draft a candidate SPOV with its DOK3 support chain:
  ```
  This might be worth a position. Here's a brainstorm:

  **Draft SPOV**: [1 sentence assertive statement]

  **Would be supported by**:
  - Insight #[N]: "[snippet]"
  - Draft Insight: "[the one we just brainstormed, if any]"

  Too early? Not spiky enough? Edit, develop further, or pass.
  ```
- If approved, insert into DOK4 section

**If user chooses 4 (Flag gap)**:
- Summarize what's missing
- Suggest specific search terms or source types that would fill the gap
- Offer: "Want me to search for sources on this? (This would be a separate `/bl:update` workflow)"

**After any save**: 
1. Apply edits to the BrainLift file using Edit tool
2. Append a log entry. Resolve log path per `infrastructure/log-format.md`: use `[brainlift_root]/logs/[name].log.md` if `logs/` exists, else write as sibling of the BrainLift file:
   ```markdown
   ## [YYYY-MM-DD] query | "[question summary]"
   
   - **Action**: query
   - **Source**: n/a
   - **Changes**: [what was saved — Initial Insight on Source X, new DOK3 #N, etc.]
   - **Rationale**: [user's reasoning if provided]
   ```

---

## Important Rules

### Citation Standards
- Always cite specific DOK levels and source names when answering
- Use format: `[Source Title] DOK1` or `Insight #N` for references
- Don't make claims the BrainLift can't support without flagging them as external

### Brainstorm Framing
- DOK3/4 suggestions are always "here's what I'm seeing" — never "here's what you should think"
- Always show the evidence chain so the human can evaluate
- Accept rejection gracefully — "Just exploring" is a valid and good outcome

### Log Discipline
- Only log when something is actually saved back to the BrainLift
- "Just exploring" queries don't get logged

### Multiple Questions
- If the user asks follow-up questions, continue searching the same BrainLift
- Each follow-up can trigger its own compound phase
- The conversation builds on itself — later questions benefit from earlier evidence mapping

### Error Handling
- **No matching BrainLift**: Ask user to specify file path
- **Empty BrainLift**: "This BrainLift doesn't have enough content to answer questions yet. Try adding sources with `/bl:update` first."
- **Question outside scope**: "This question is outside your BrainLift's scope ([Out of Scope text]). Want to search anyway, or try a different BrainLift?"
