---
description: Add new content to an existing BrainLift - paste a URL and your thoughts, get guided through the update
argument-hint: <url> [your thoughts about this content]
---

You're helping the user add new content to an existing BrainLift. This is the core BrainMaxxing workflow: ingest a source, extract knowledge, and update the BrainLift through a guided interview.

## Core BrainLifting Rules

### DOK Framework
- **DOK1 (Facts)**: Objective, verifiable information directly from sources. 2-3 sentences each. AI can extract these.
- **DOK2 (Summary)**: Synthesis of multiple DOK1s from a single source. Objective, pattern-identifying. 2-3 sentences each.
- **DOK3 (Insights)**: Surprising, contrarian patterns synthesized across multiple sources. Subjective. Must be supported by DOK1-2.
- **DOK4 (SPOV)**: Spiky Points of View - expert positions on disputed topics. Must be supported by 2+ DOK3 insights. Short, assertive statements.

### Bright Lines
1. **DOK1-2 vs DOK3-4**: DOK1-2 are objective (external world), DOK3-4 are subjective (owner's expertise)
2. **DOK1 vs DOK2**: No learning in DOK1 extraction (AI can do it). Learning begins at DOK2+ (human must write)
3. **BrainLift vs BrainMaxxing**: Humans curate their own BrainLift. No automation or copy-paste.

### Evidence Chain
- DOK4 requires 2+ DOK3 insights (else empty claim)
- DOK3 requires DOK1-2 evidence (else baseless assertion)
- DOK2 requires DOK1 facts (else unsupported summary)

### BrainLift File Structure

Canonical top-to-bottom layout of a BrainLift file. The skill must respect this order when inserting content.

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

**Section details relevant to updates:**

- **Purpose**: 1 sentence In Scope, 1 sentence Out of Scope. Used to route sources to the right BrainLift.
- **DOK4 - SPOV**: Short assertive statements. Assertions, not explanations. These are the lens through which the owner reads and summarizes new sources.
- **DOK3 - Insights**: Contrarian, surprising, cross-source synthesis. 2-3 sentences each.
- **DOK3 - Blueprints**: Structured directives that operationalize insights. Include: domain context, actions (with judgment calls), tools/systems, boundaries, QC/definition of done. Think: recipes, playbooks, SOPs. A new source might trigger a new or revised Blueprint.
- **Experts**: People/orgs that produce valuable sources. Organized into categories when list grows. Each entry: Who, Focus, Why Follow, Where.
- **Knowledge Tree**: Subdivides external knowledge by sub-topic. Can be flat or hierarchical. Categorizing and summarizing sources is itself DOK2 work. Should be refactored as DOK3-4 evolve.
  - Each source must have: DOK1 facts, owner's DOK2 summary, link to original
  - DOK2 summaries are written through the lens of existing DOK4 SPOVs (when they exist). Before SPOVs, summaries are generic.

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
- [Why valuable? Main contribution?]

**DOK1 - Facts**
- **[Concept]**: [2-3 sentence technical description]
- **[Concept]**: [2-3 sentence technical description]
[8-10 facts]

**Initial Insights**
- [User's early synthesis - left blank for user to fill]

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
1. Use Glob to find `*.md` files in the BrainLift directory
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

---

### Phase 3: DOK1 Extraction

Spawn a dok-extractor agent to extract facts from the source:

```
Task(
  subagent_type: "dok-extractor",
  description: "Extract DOK1-2 from source",
  prompt: "You are a dok-extractor agent.

  Source URL: [URL]
  Source Title: [title]
  Author: [author]
  Date: [date]
  Content: [fetched content text if available]

  BrainLift Purpose:
  - In Scope: [from selected BrainLift]
  - Out of Scope: [from selected BrainLift]

  Extract:
  - 8-10 DOK1 facts (2-3 sentences each, technical, verifiable)
  - 3-4 DOK2 patterns (2-3 sentences each, synthesizing DOK1s)

  Format DOK1 as:
  - **[Concept Name]**: [Detailed technical description - 2-3 sentences]

  Format DOK2 as:
  - **[Pattern Name]**: [Synthesis of multiple facts - 2-3 sentences]

  Return formatted DOK1 facts and DOK2 patterns."
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

This is AI-assisted extraction - allowed by the bright lines. The user curates what goes in.

---

### Phase 4: Placement & DOK2 Interview

**Parse the Knowledge Tree** from the BrainLift and present categories:

```
Your Knowledge Tree currently has these categories:

1. **[Category A]** - [X sources]
2. **[Category B]** - [Y sources]
3. **[Category C]** - [Z sources]

I suggest placing this source in **[Category X]** because [reasoning].

Or should we create a new category? Where does this fit best?
```

**After user confirms placement**, generate a DOK2 strawman:

```
Here's a strawman DOK2 summary to react against. DO NOT copy this -
write your own version. This is just to help you think about what to include:

---
STRAWMAN (react against this, don't copy):

**DOK2 - Summary**
- [AI-generated summary draft highlighting source value and key contribution]
---

Now write YOUR DOK2 summary. What makes this source valuable to your BrainLift?
Why did you choose to include it? What's its main contribution?

If you have SPOVs, write your summary through that lens — how does this source
relate to your existing positions?
```

**CRITICAL**: The user MUST type their own DOK2 summary. This is where learning begins.
Do NOT let them just accept the strawman. If they try to copy it, remind them of the bright line.

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

**If user identifies DOK3 updates**:
- Show the current insight text
- User writes the revised version (human-only for DOK3)
- Remind about evidence chain: "Make sure this insight is still supported by DOK1-2"

**If user identifies DOK4 updates**:
- Show the current SPOV text
- User writes the revised version
- Remind: "Does this SPOV still have 2+ supporting DOK3 insights?"

**If user identifies a NEW DOK3 insight**:
- Guide them to articulate it (2-3 sentences)
- Check: "Is this synthesized from multiple sources? Which DOK1-2 evidence supports it?"
- Remind: "A good insight should surprise you - is this non-obvious?"

**Expert consideration**:
```
Should the author of this source be added to your Experts section?

Author: [name]
Affiliation: [company/org]
Known for: [brief description]

If yes, I'll format an expert entry for you to review.
```

If yes, format:
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
  - DOK2: Your summary
  - Initial Insights: [blank - for you to fill after reading]
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

Approve these changes?
```

**On approval**, apply edits:

1. **Add new source entry** in the Knowledge Tree under the selected category
   - Use the Edit tool to insert at the correct position
   - Include all selected DOK1 facts
   - Include user's DOK2 summary
   - Leave Initial Insights blank (with TODO note for user)
   - Include source URL

2. **Update category summary** if needed
   - Edit the existing category summary to reflect the new source

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

**Suggest next steps**:
```
Source added! Next steps:
- Read the original source directly (don't just rely on extraction)
- Fill in your Initial Insights after reading
- Review your evidence chains (do DOK3s still have DOK1-2 support?)
- Test updated insights against other sources in your Knowledge Tree
```

---

## Important Rules

### What AI Does vs. What Human Does

**AI-Assisted (allowed)**:
- Content fetching and type detection
- DOK1 fact extraction
- DOK2 strawman generation (for reaction, not copying)
- BrainLift routing and category suggestion
- Formatting and file editing
- Git operations

**Human-Only (enforce bright line)**:
- Writing DOK2 summary (must type their own)
- Writing/revising DOK3 insights
- Writing/revising DOK4 SPOVs
- Deciding what to include/exclude
- Knowledge Tree curation decisions
- Reading original sources

### Quality Checks

Before applying changes, verify:
- [ ] DOK1 facts are specific and technical (not vague)
- [ ] DOK2 summary is user-written (not AI strawman)
- [ ] Source URL is included
- [ ] Placement in Knowledge Tree makes sense
- [ ] If DOK3 updated: evidence chain still holds
- [ ] If DOK4 updated: still has 2+ supporting DOK3s
- [ ] No bright line violations

### Error Handling

- **URL fetch fails**: Ask user to paste content directly
- **No matching BrainLift**: Ask user to specify file path manually
- **User tries to copy AI strawman**: Remind them of the bright line - they must write their own
- **Content is behind paywall**: Note the limitation, ask if user can paste the content
- **BrainLift file has unexpected format**: Best-effort parsing, confirm with user before editing
