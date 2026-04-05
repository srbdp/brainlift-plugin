# BrainLift Discovery Protocol

How the plugin finds BrainLift files from any Claude Code session.

## Overview

All skills (`/bl:update`, `/bl:query`, `/bl:lint`) need to locate BrainLift files before they can operate. This protocol replaces the ad-hoc "ask the user for a directory" approach with a three-step cascade that works automatically in most cases.

## The Three-Step Cascade

### Step 1: Walk up from `$CWD` for CLAUDE.md marker

Starting from the current working directory, check each parent directory up to `~` for a `CLAUDE.md` file that contains the BrainLift workspace marker:

```html
<!-- brainlift-root -->
```

**How to check**: Read the first 10 lines of each `CLAUDE.md` found. The marker comment will be on line 1.

**If found**: That directory is the **BrainLift root**.

**Resolve lifts directory**:
- If `[root]/lifts/` exists → use it (recommended structure)
- Else → use `[root]/` directly (flat/legacy structure)

**Resolve logs directory**:
- If `[root]/logs/` exists → use it
- Else → logs are siblings of BrainLift files (legacy behavior)

This step succeeds when the user has run `/bl:init` and is working inside their BrainLift folder or any subfolder.

### Step 2: Check `~/.brainlift` pointer file

Read the file `~/.brainlift`. It contains one line: the absolute path to a BrainLift root directory.

```
/Users/bpizzacalla/brainlifts
```

**If the file exists and the path is valid**: Use that directory as the BrainLift root. Resolve `lifts/` and `logs/` the same way as Step 1.

This step succeeds when the user has run `/bl:init` but is currently working in a different project directory.

### Step 3: Ask the user

If neither Step 1 nor Step 2 succeeds:

1. Ask: "Where are your BrainLift files? (paste a directory path)"
2. Use the provided path as the BrainLift root
3. After successful routing, offer: "Want me to remember this location? I'll save it to `~/.brainlift` so you don't have to specify it next time."
4. If the user agrees, write the absolute path to `~/.brainlift`

## After Discovery

Once the lifts directory is resolved, the existing routing logic takes over:

1. **Glob** `[lifts_dir]/*.md` (exclude `*.log.md` for backward compatibility with flat setups)
2. **Read** each file's opening to extract the **Purpose** section (In Scope / Out of Scope)
3. **Score** content relevance against each BrainLift's scope
4. **Present** ranked matches for the user to pick

## Path Resolution Summary

| Component | Recommended (after init) | Legacy (flat folder) |
|-----------|-------------------------|---------------------|
| BrainLift root | Directory containing `CLAUDE.md` with marker | User-specified directory |
| Lifts directory | `[root]/lifts/` | `[root]/` |
| Logs directory | `[root]/logs/` | Same as lifts (sibling files) |
| Sources directory | `[root]/sources/` | N/A |
| Pointer file | `~/.brainlift` | N/A |

## Skill-Specific Notes

### `/bl:update`
Runs discovery in Phase 2 (BrainLift Routing). After discovery, proceeds with relevance scoring as normal.

### `/bl:query`
Runs discovery in Phase 1 (Route to BrainLift). Same flow.

### `/bl:lint`
If a path argument is provided in `$ARGUMENTS`, use it directly — skip discovery. Otherwise, run discovery as the no-argument fallback.

### `/bl:init`
Does NOT use this protocol — it scaffolds the workspace that this protocol discovers.
