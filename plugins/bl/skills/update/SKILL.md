---
description: Add new content to an existing BrainLift - paste a URL and your thoughts, get guided through the update
argument-hint: <url> [your thoughts about this content]
---

You're helping the user add new content to an existing BrainLift. This is the core BrainMaxxing workflow: ingest a source, extract knowledge, and update the BrainLift through a guided interview.

## Philosophy (v2)

Inspired by Karpathy's LLM Wiki pattern: a single source ingest should ripple across the BrainLift — updating summaries, flagging cross-references, suggesting insights. The LLM handles all bookkeeping so the human can focus on curation decisions.

**Relaxed bright lines**: AI generates DOK2 directly (not as a strawman). AI brainstorms DOK3/4 suggestions with evidence, framed as brainstorming. Human curates — accepts, edits, or rejects everything. The learning happens through curation decisions, not the mechanical act of writing.

## Core Rules

### DOK Framework
- **DOK1 (Facts)**: Objective, verifiable information directly from sources. 2-3 sentences each. AI extracts these.
- **DOK2 (Summary)**: Synthesis of multiple DOK1s from a single source. Objective, pattern-identifying. 2-3 sentences each. AI generates these; human reviews/edits.
- **DOK3 (Insights)**: Surprising, contrarian patterns synthesized across multiple sources. Subjective. AI brainstorms candidates with evidence; human accepts/edits/rejects.
- **DOK4 (SPOV)**: Spiky Points of View - expert positions on disputed topics. Must be supported by 2+ DOK3 insights. AI brainstorms candidates; human decides.

### What AI Does vs. What Human Does

**AI handles** (bookkeeping & drafting):
- Content fetching and type detection
- DOK1 fact extraction
- DOK2 summary generation (full, not strawman)
- DOK3/4 brainstorming with evidence citations
- BrainLift routing and category suggestion
- Category summary updates
- Expert detection and formatting
- Cross-category relevance flagging
- Source replacement analysis
- Formatting and file editing
- Git operations and log entries

**Human handles** (curation & direction):
- Selecting which DOK1 facts to include
- Reviewing/editing AI-generated DOK2
- Accepting/editing/rejecting brainstormed DOK3/4
- Deciding what to include/exclude
- Knowledge Tree curation decisions
- Reading original sources

### Evidence Chain
- DOK4 requires 2+ DOK3 insights (else empty claim)
- DOK3 requires DOK1-2 evidence (else baseless assertion)
- DOK2 requires DOK1 facts (else unsupported summary)

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

### Token Optimization
- Peak power: 500-1000 tokens for DOK3-4
- Functional range: 2000-2500 tokens
- Danger zone: 5000+ tokens
- SPOVs must be semantically distinct, not narrative
- Include "except when" clauses and document uncertainty

### BrainLift Source Entry Format
```markdown
**[Source Title] - [Author] ([Date])**

**DOK2 - Summary**
- [AI-generated summary — why valuable, main contribution]

**DOK1 - Facts**
- **[Concept]**: [2-3 sentence technical description]
- **[Concept]**: [2-3 sentence technical description]
[8-10 facts]

**Initial Insights**
- [Early synthesis — left for user to fill after reading]

[Source URL]
```

## Command Usage

```
/bl:update https://x.com/user/status/123 This challenges my thinking on X
/bl:update https://example.com/article
/bl:update
```

---

## 6-Phase Interview Flow

### Phase 1: Content Ingestion

Parse `$ARGUMENTS` for URL(s) and any additional text (user's thoughts).

**If no URL provided**: Ask the user to paste a link or content directly.

**If URL provided**: Detect URL type and fetch content:

1. **Twitter/X** (`twitter.com` or `x.com`):
   - Extract `screen_name` and tweet `id` from URL
   - Spawn a content-fetcher agent using the Task tool:
     ```
     Task(
       subagent_type: "general-purpose",
       description: "Fetch tweet content",
       prompt: "You are a content-fetcher agent.

       Fetch this tweet: [URL]

       Use the FxTwitter API (no API key needed):
       1. Status API: https://api.fxtwitter.com/:screen_name/status/:id
          - Returns JSON with tweet.text, tweet.author, engagement metrics,
            tweet.quote (nested quote tweet), tweet.replying_to_status (parent tweet ID)
       2. User API: https://api.fxtwitter.com/:screen_name
          - Returns JSON with user.description (bio), user.followers, user.location, etc.

       Fetch the tweet first. Then fetch the author's profile via User API.
       If tweet.replying_to_status is set, fetch the parent tweet for reply context.
       If tweet.quote exists, include the full quote tweet text.

       Format as structured report with Metadata, Content, Context, and Author Profile sections."
     )
     ```

2. **YouTube** (`youtube.com` or `youtu.be`):
   - Spawn content-fetcher agent with YouTube-specific instructions
   - Extract title, channel, description, transcript if available

3. **PDF** (URL ending in `.pdf`):
   - Use WebFetch or Firecrawl to extract text content

4. **Articles/Blogs** (everything else):
   - Spawn content-fetcher agent with WebFetch instructions
   - Extract title, author, date, main article text

**Present extracted content summary** to user:
```
I fetched this content:
- **Title**: [title]
- **Author**: [name] (@handle)
- **Date**: [date]
- **Type**: [tweet/article/video]

**Content preview**:
[First 200-300 words or full tweet text]

Does this look right? Any additional context you want to add?
```

Note any user thoughts provided in $ARGUMENTS - these inform later phases.

---

### Phase 2: BrainLift Routing

**If this is the first time**: Ask the user for the path to their BrainLift files directory.

**Find candidate BrainLifts**:
1. Use Glob to find `*.md` files in the BrainLift directory (exclude `*.log.md`)
2. Read each file's opening section to extract **Purpose** (In Scope / Out of Scope)
3. Score content relevance against each BrainLift's scope
4. Present ranked matches:

```
I found these BrainLifts that might be relevant:

1. **[BrainLift Topic]** - [In Scope summary]
   Relevance: [Why this content fits]

2. **[BrainLift Topic]** - [In Scope summary]
   Relevance: [Partial fit because...]

3. **[BrainLift Topic]** - [In Scope summary]
   Relevance: [Tangential - might not fit]

Which BrainLift should this go into?
```

**After user selects**: Read the full selected BrainLift file to understand:
- Current Knowledge Tree structure and categories
- Existing DOK3 Insights
- Existing DOK4 SPOVs
- Current experts list
- Existing DOK1-2 from other sources (for DOK3 brainstorming)

---

### Phase 3: DOK1 Extraction

Spawn a dok-extractor agent with the existing BrainLift context:

```
Task(
  subagent_type: "dok-extractor",
  description: "Extract DOK1-2 and brainstorm DOK3",
  prompt: "You are a dok-extractor agent.

  Source URL: [URL]
  Source Title: [title]
  Author: [author]
  Date: [date]
  Content: [fetched content text if available]

  BrainLift Purpose:
  - In Scope: [from selected BrainLift]
  - Out of Scope: [from selected BrainLift]

  Existing DOK4 SPOVs:
  [list current SPOVs — write DOK2 through this lens]

  Existing Knowledge Tree Summary:
  [summarize existing sources' DOK1-2 for cross-source pattern detection]

  Extract:
  - 8-10 DOK1 facts (2-3 sentences each, technical, verifiable)
  - 3-4 DOK2 patterns (2-3 sentences each, synthesizing DOK1s)
  - Article summary (2-3 paragraphs)
  - DOK3 brainstorm (if cross-source patterns emerge)

  Format as complete source entry with brainstormed insights section."
)
```

**Present extracted DOK1 facts to user**:
```
Here are the DOK1 facts I extracted. Review and select which to include:

1. **[Concept]**: [description]
2. **[Concept]**: [description]
...

Which facts do you want to include? You can:
- Select by number (e.g., "1, 3, 5, 7")
- Edit any fact before including
- Add facts I missed
- Remove facts that aren't relevant
```

This is AI-assisted extraction — the user curates what goes in.

---

### Phase 4: Placement & DOK2

**Parse the Knowledge Tree** from the BrainLift and present categories:

```
Your Knowledge Tree currently has these categories:

1. **[Category A]** - [X sources]
2. **[Category B]** - [Y sources]
3. **[Category C]** - [Z sources]

I suggest placing this source in **[Category X]** because [reasoning].

Or should we create a new category? Where does this fit best?
```

**Cross-category relevance flags** (Karpathy enhancement):
After the user confirms placement, scan the selected DOK1 facts against other Knowledge Tree categories:
```
Note: Some facts from this source are also relevant to **[Category Y]**:
- "[DOK1 fact about X]" connects to sources in Category Y about [topic]

This is just a flag — the source stays in [Category X]. But you might want
to reference this connection in Category Y's summary later.
```

**Present AI-generated DOK2 summary**:

```
Here's the DOK2 summary I've written for this source:

**DOK2 - Summary**
- [AI-generated summary highlighting source value, main contribution,
  and how it relates to existing BrainLift knowledge. If SPOVs exist,
  written through that lens.]

Edit anything that doesn't match your understanding, or approve to continue.
```

The user reviews and either approves, edits, or rewrites. No "strawman" framing — this is a direct draft the user curates.

---

### Phase 5: Higher DOK Impact Assessment

Display the BrainLift's existing DOK3 and DOK4 sections:

```
Here are your current Insights and SPOVs:

**DOK4 - SPOVs:**
1. [SPOV text]
2. [SPOV text]
3. [SPOV text]

**DOK3 - Insights:**
1. [Insight text]
2. [Insight text]
3. [Insight text]
```

Then ask structured questions:

```
Now let's think about higher-level impact:

1. **Does this source SUPPORT any existing DOK3 insights?**
   (Which ones? How does it strengthen them?)

2. **Does this source CONTRADICT any existing DOK3 insights?**
   (Which ones? Should you revise them?)

3. **Does this source suggest a NEW DOK3 insight?**
   (What cross-source pattern are you seeing?)

4. **Does it affect any DOK4 SPOVs?**
   (Strengthen, weaken, add nuance?)

5. **Or is this just a Knowledge Tree addition for now?**
   (Perfectly fine - not every source changes DOK3-4)

6. **Does this source suggest a new or revised Blueprint?**
   (Operational playbook based on your insights)

7. **Should any existing source be replaced by this one?**
   (Don't just append — curate)
```

**If the dok-extractor returned brainstormed insights**, present them now:
```
The dok-extractor also spotted some cross-source patterns:

**Brainstormed Insight**: [draft insight]
Supporting evidence:
- This source DOK1: "[concept]"
- [Existing Source] DOK1: "[concept]"

Does this resonate? Want to develop it further, edit it, or pass?
```

**If user identifies DOK3 updates** — AI drafts the revision:
- For new DOK3: "Based on what you're describing, here's a draft insight — edit to match your thinking:"
  ```
  [2-3 sentence draft insight with evidence citations]

  Supporting evidence:
  - [Source A] DOK1: "[fact]"
  - [Source B] DOK2: "[pattern]"
  ```
- For revised DOK3: "Here's your current insight revised to incorporate this new evidence:"
  ```
  Current: [old text]
  Revised: [new text incorporating new evidence]

  Keep, edit, or reject?
  ```
- Remind about evidence chain: "Make sure this insight is supported by DOK1-2"

**If user identifies DOK4 updates** — AI brainstorms:
- "This might support a new SPOV position. Here's a brainstorm:"
  ```
  Draft SPOV: [1 sentence assertive statement]

  Would be supported by:
  - Insight #[N]: "[snippet]"
  - [New insight if just created]

  Too early? Not spiky enough? Edit or pass.
  ```
- For revised DOK4: "Here's a refined version with the new nuance:"
  ```
  Current: [old SPOV]
  Revised: [updated SPOV]
  
  Still has 2+ supporting insights? Keep, edit, or reject?
  ```

**Source replacement analysis** (Karpathy enhancement):
When presenting question 7, add analysis:
```
Looking at existing sources in **[Category]**:

- **[Existing Source A]**: Covers [topics]. Overlap with new source: [high/medium/low]
  [If high]: This new source may supersede it — newer and covers similar ground.
- **[Existing Source B]**: Covers [topics]. Overlap: low — different angle.

Replace anything, or keep all?
```

**Expert detection** (Karpathy enhancement):
```
Should the author be added to your Experts section?

Author: [name]
Affiliation: [company/org]
Known for: [brief description]
```

Also check: "This author appears in [N] of your existing sources — strong signal for Expert inclusion" if applicable.

If adding expert, format:
```markdown
**[Author Name]**
- Who: [Title and affiliation]
- Focus: [Specific expertise area]
- Why Follow: [Relevance to BrainLift]
- Where:
  - @[handle]
  - [URL]
```

---

### Phase 6: Apply Changes & Commit

**Summarize all proposed changes** before touching the file:

```
Here's everything that will change in your BrainLift:

**Knowledge Tree ([Category Name]):**
- ADD new source entry: "[Source Title] - [Author] ([Date])"
  - DOK1: [X] facts
  - DOK2: [AI-generated, user-approved summary]
  - Initial Insights: [blank — fill after reading]
  - Link: [URL]

**Category Summary** (if updating):
- CURRENT: [existing summary]
- PROPOSED: [updated summary reflecting new source]

**DOK3 Insights** (if any changes):
- REVISE Insight #2: [show old → new]
- ADD new insight: [text]

**DOK4 SPOVs** (if any changes):
- REVISE SPOV #1: [show old → new]

**Experts** (if adding):
- ADD: [Expert Name] to [Category]

**Cross-references noted:**
- [Category Y] may want to reference [specific fact]

Approve these changes?
```

**On approval**, apply edits:

1. **Add new source entry** in the Knowledge Tree under the selected category
   - Use the Edit tool to insert at the correct position
   - Include all selected DOK1 facts
   - Include user-approved DOK2 summary
   - Leave Initial Insights blank (with guidance text for user)
   - Include source URL

2. **Update category summary** (Karpathy enhancement)
   - Generate an updated category summary reflecting the new source
   - Present for user approval before applying
   - This is mechanical DOK2 work — AI generates, user approves

3. **Update DOK3 insights** if any were revised or added
   - Edit existing insight text or insert new insights

4. **Update DOK4 SPOVs** if any were revised
   - Edit existing SPOV text

5. **Add expert entry** if applicable
   - Insert in the Experts section

**After applying edits**:
- If in a git repo, run `git diff` to show actual file changes
- Show the diff to the user for final review
- On final approval: stage the file and commit with a descriptive message like:
  `"Add source: [Source Title] by [Author] to [BrainLift Topic] Knowledge Tree"`
- **Never auto-push** - user pushes manually

**Append log entry** (Karpathy enhancement):
After successful commit, append to the companion `.log.md` file:
```markdown
## [YYYY-MM-DD] ingest | "[Source Title]" - [Author]

- **Action**: ingest
- **Source**: [Source Title] - [Author] ([Date])
- **Changes**: Added to [Category] with [N] DOK1 facts. [If DOK3/4 changed: "Revised Insight #N", "Added new SPOV", etc.]
- **Rationale**: [User's thoughts from $ARGUMENTS if provided, or brief note on why this source was added]
```

If DOK3/4 were also revised, append additional log entries for those changes.

**Suggest next steps**:
```
Source added! Next steps:
- Read the original source directly (don't just rely on extraction)
- Fill in your Initial Insights after reading
- Run `/bl:lint` to check your BrainLift's health
- Use `/bl:query` to explore how this source changes your understanding
```

---

## Error Handling

- **URL fetch fails**: Ask user to paste content directly
- **No matching BrainLift**: Ask user to specify file path manually
- **Content is behind paywall**: Note the limitation, ask if user can paste the content
- **BrainLift file has unexpected format**: Best-effort parsing, confirm with user before editing
- **Log file doesn't exist**: Create it with header before appending first entry

## Quality Checks

Before applying changes, verify:
- [ ] DOK1 facts are specific and technical (not vague)
- [ ] DOK2 summary is reviewed/approved by user
- [ ] Source URL is included
- [ ] Placement in Knowledge Tree makes sense
- [ ] If DOK3 updated: evidence chain still holds
- [ ] If DOK4 updated: still has 2+ supporting DOK3s
- [ ] Category summary updated to reflect new source
- [ ] Log entry appended
