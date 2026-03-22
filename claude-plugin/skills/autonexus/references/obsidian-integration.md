# Obsidian Integration Layer — AutoNexus

This file defines all Obsidian MCP interactions. Every AutoNexus command loads this file. All Obsidian operations follow the protocols defined here.

## Prerequisites

- Obsidian desktop app running
- "Local REST API" community plugin installed and enabled in Obsidian
- Obsidian MCP server configured in Claude Code (`.mcp.json`)
- Vault exists (AutoNexus creates project folders on first run)

## Vault Layout

```
{Vault Root}/
├── Projects/
│   └── {ProjectName}/
│       ├── Knowledge/
│       │   ├── Architecture.md       ← System overview, components, data flow, tech stack
│       │   ├── Components.md         ← Per-component details, dependencies, risk areas
│       │   └── Dependencies.md       ← Import graph, call graph, external deps
│       ├── Decisions/
│       │   ├── {YYYY-MM-DD}-{slug}.md  ← ADR-format decision records
│       │   └── ...
│       ├── Iterations/
│       │   ├── {YYYY-MM-DD}/
│       │   │   ├── {session-id}-iter-{N}.md  ← Per-iteration concise notes
│       │   │   └── {session-id}-summary.md   ← Session summary
│       │   └── Archive/
│       │       └── {YYYY-MM-DD}/             ← Auto-archived after 30 days
│       ├── Predictions/
│       │   └── {YYYY-MM-DD}-{slug}/
│       │       ├── persona-{name}.md         ← One note per persona
│       │       └── debate.md                 ← Debate transcript + consensus
│       ├── Pipelines/
│       │   ├── {name}.md                     ← Saved pipeline definitions
│       │   └── runs/
│       │       └── {run-id}.md               ← Pipeline run logs
│       ├── Hooks/
│       │   └── {name}.md                     ← Event-driven hook definitions
│       └── Canvases/
│           ├── architecture.canvas           ← Component relationship diagram
│           ├── sprint-{N}-board.canvas       ← Sprint board kanban
│           ├── dependencies.canvas           ← Import/dependency graph
│           ├── timeline.canvas               ← Session history timeline
│           ├── knowledge-map.canvas          ← All notes with connections
│           ├── data-flow.canvas              ← Data flow diagram
│           └── pipeline-{name}.canvas        ← Pipeline step visualization
├── Cross-Project/
│   └── {slug}.md                             ← Patterns applicable across projects
└── Daily Notes/                              ← Managed by Obsidian's daily note plugin
```

## Project Name Resolution

Determine `{ProjectName}` in this order:

1. `git remote get-url origin` → extract repo name (strip `.git` suffix, take last path segment)
2. Read `package.json` → use `name` field
3. Fallback: `basename` of current working directory

Use the resolved name consistently for all Obsidian paths in the session.

---

## MCP Tool Quick Reference

| Tool | Use For | Key Params |
|------|---------|-----------|
| `obsidian_read_note` | Read note content | `filePath`, `format`, `includeStat` |
| `obsidian_update_note` | Create/append/overwrite notes | `targetType`, `targetIdentifier`, `content`, `wholeFileMode`, `createIfNeeded` |
| `obsidian_global_search` | Search vault for content | `query`, `searchInPath`, `contextLength`, `pageSize` |
| `obsidian_list_notes` | List folder contents | `dirPath`, `recursionDepth` |
| `obsidian_manage_frontmatter` | Get/set frontmatter keys | `filePath`, `operation`, `key`, `value` |
| `obsidian_manage_tags` | Add/remove/list tags | `filePath`, `operation`, `tags` |
| `obsidian_search_replace` | Find and replace in notes | `targetType`, `replacements`, `replaceAll` |
| `obsidian_delete_note` | Delete a note | `filePath` |

---

## Retry-Then-Queue Protocol

**CRITICAL: Every Obsidian MCP call in AutoNexus MUST follow this protocol. The loop NEVER blocks on an Obsidian failure.**

### Flow

```
FOR every Obsidian MCP call:
  1. Attempt the MCP call
  2. IF success:
       - Check if obsidian-sync-queue.md has pending items
       - IF yes: attempt to drain 1-3 queued items (opportunistic)
       - RETURN result
  3. IF failure:
       - Log: "Obsidian MCP failed (attempt 1): {error}"
       - Wait 2 seconds
       - Retry the MCP call once
  4. IF retry succeeds:
       - Drain queue opportunistically
       - RETURN result
  5. IF retry fails:
       - Log: "Obsidian MCP failed (attempt 2): {error}. Queuing operation."
       - Append the operation to obsidian-sync-queue.md
       - RETURN null (the loop continues uninterrupted)
```

### obsidian-sync-queue.md Format

This file lives in the project root and is gitignored. It accumulates failed MCP operations:

```markdown
# Obsidian Sync Queue
<!-- AutoNexus queued operations. Drain: next successful MCP call, session end, or next session start. -->

## Pending

### Entry 1
- **Tool:** obsidian_update_note
- **Path:** Projects/MyApp/Iterations/2026-03-22/sess-001-iter-5.md
- **Mode:** overwrite
- **Content:**
```
{full note content here}
```
- **Fail Count:** 1
- **Queued At:** 2026-03-22T14:30:00

## Failed (Manual Review)
<!-- Items that failed 3+ drain attempts are moved here -->
```

### Queue Drain Protocol

Queue drain is attempted at these moments:
1. **Opportunistic:** After every successful MCP call, drain 1-3 pending items
2. **Session end:** Before writing the session summary, attempt full drain
3. **Session start:** At Phase 0, before archive, attempt full drain

```
FOR each pending item in queue:
  Attempt the MCP call
  IF success: remove from queue
  IF failure:
    Increment fail_count
    IF fail_count >= 3: move to "Failed (Manual Review)" section
  CONTINUE to next item (never block)

IF all items drained: delete obsidian-sync-queue.md
```

---

## Phase 0: Session Initialization

Run these steps at the start of every AutoNexus session, BEFORE the loop begins.

### Step 1: Tool Existence Check

Before calling any Obsidian MCP tool, check whether Obsidian MCP tools exist in the current tool set. Look for tools named `obsidian_read_note`, `obsidian_update_note`, `obsidian_global_search`, `obsidian_list_notes`.

```
IF obsidian MCP tools are NOT in available tool set:
  SET OBSIDIAN_AVAILABLE = false
  SET OBSIDIAN_NOT_CONFIGURED = true
  WARN once: "Obsidian MCP is not configured. AutoNexus is running in local-only mode.
              The autonomous loop works fully without Obsidian — you get git + TSV tracking.
              To enable persistent knowledge in Obsidian, run /autonexus:setup
              (it will walk you through everything automatically)."
  SKIP Steps 2-5 entirely
  DO NOT attempt any Obsidian MCP calls for the entire session
  PROCEED directly to loop setup
```

This handles the case where a user installs AutoNexus but has no Obsidian setup at all. The plugin works fully — they just don't get persistent knowledge.

### Step 1b: Connectivity Check (only if tools exist)

```
TRY:
  obsidian_list_notes({ dirPath: "Projects/", recursionDepth: 0 })
  SET OBSIDIAN_AVAILABLE = true
  LOG "Obsidian vault connected."
CATCH:
  SET OBSIDIAN_AVAILABLE = false
  WARN (once): "Obsidian MCP tools found but connection failed. Running in local-only mode.
                Check that Obsidian is open and Local REST API plugin is enabled.
                Local TSV + git continue normally. Obsidian writes queued."
```

If `OBSIDIAN_AVAILABLE = false` (from either step), ALL Obsidian operations in this session become no-ops. Do NOT prompt the user to fix it. Do NOT retry during the session. Log the warning once and proceed.

### Step 1c: Knowledge Decay Check (#22)

After confirming Obsidian is connected (`OBSIDIAN_AVAILABLE = true`), check whether the Knowledge Center is stale relative to the current codebase:

```
Read Knowledge Center Architecture.md frontmatter:
  obsidian_manage_frontmatter({ filePath: "Projects/{ProjectName}/Knowledge/Architecture.md", operation: "get", key: "commit_hash" })

Compare with current git rev-parse HEAD.
Count commits between: git rev-list --count {cached_hash}..HEAD

IF commits_behind > 20 OR days_since_update > 14:
  WARN: "Knowledge Center is {N} commits behind HEAD (last updated {date}).
         Suggest running /autonexus:learn --mode update after this session."
ELIF commits_behind > 5:
  INFO: "Knowledge Center is {N} commits behind. Still usable but may miss recent changes."
ELSE:
  (silent — knowledge is fresh)
```

If the Architecture.md note does not exist yet (first run), skip this check silently — the Knowledge Center will be created during the first pre-push rebuild.

### Step 2: Drain Sync Queue

If `obsidian-sync-queue.md` exists from a previous session:
- Attempt full drain per the Queue Drain Protocol
- Log results: "Drained {N}/{M} queued operations from previous session"

### Step 3: Smart Archival (#21)

Archive iteration notes using relevance scoring instead of blanket 30-day removal.

#### Relevance Scoring

Each iteration note receives a relevance score before archival decisions are made:

```
FOR each iteration note older than 14 days:
  score = 1  (base score)

  # Check if linked from a Decision note (backlink check)
  obsidian_global_search({
    query: "{note filename}",
    searchInPath: "Projects/{ProjectName}/Decisions/",
    contextLength: 50,
    pageSize: 5
  })
  IF results contain a link to this note:
    score += 10

  # Check frontmatter for "important" tag
  obsidian_manage_tags({
    filePath: "{note path}",
    operation: "list"
  })
  IF tags include "autonexus/important":
    SKIP — this note is NEVER archived regardless of age

  IF tags include "important":
    score += 5

  # Check frontmatter status
  obsidian_manage_frontmatter({
    filePath: "{note path}",
    operation: "get",
    key: "status"
  })
  IF status == "keep":
    score += 3

  # Age decay: subtract 1 per week of age
  weeks_old = floor((today - note_date) / 7)
  score -= weeks_old
```

#### Archive Threshold

Notes with `relevance score <= 0` are archived. Notes tagged `autonexus/important` are **NEVER** archived regardless of score or age.

#### MCP Call Budget

Track the number of MCP calls made during relevance scoring. If the count exceeds 20, stop scoring and fall back to simple 30-day rule for remaining notes:

```
IF mcp_call_count > 20:
  WARN: "Relevance scoring exceeded MCP budget (20 calls). Falling back to 30-day rule for remaining notes."
  FOR remaining unscored notes:
    IF (today - note_date) > 30 days:
      Mark for archival
```

#### Archival Process

```
1. List date folders:
   obsidian_list_notes({ dirPath: "Projects/{ProjectName}/Iterations/", recursionDepth: 0 })

2. For each folder with name matching YYYY-MM-DD pattern:
   Parse the date. If (today - date) > 14 days → score notes for archival.

3. Cap at 5 folders per session start (avoid long Phase 0).

4. For each note with relevance score <= 0 (or matching 30-day fallback):
   a. List files in the folder:
      obsidian_list_notes({ dirPath: "Projects/{ProjectName}/Iterations/{date}/", recursionDepth: 0 })

   b. For each file marked for archival:
      i.   Read: obsidian_read_note({ filePath: "Projects/{ProjectName}/Iterations/{date}/{file}" })
      ii.  Write to archive: obsidian_update_note({
             targetType: "filePath",
             targetIdentifier: "Projects/{ProjectName}/Iterations/Archive/{date}/{file}",
             content: {read content},
             modificationType: "wholeFile",
             wholeFileMode: "overwrite",
             createIfNeeded: true
           })
      iii. Tag: obsidian_manage_tags({
             filePath: "Projects/{ProjectName}/Iterations/Archive/{date}/{file}",
             operation: "add",
             tags: ["autonexus/archived"]
           })
      iv.  Delete original: obsidian_delete_note({
             filePath: "Projects/{ProjectName}/Iterations/{date}/{file}"
           })

      Each step uses retry-then-queue. If any step fails, skip this file and continue.

5. Log: "Archived {N} notes from {M} sessions (relevance-scored: {S}, fallback 30-day: {F})"
```

### Step 4: Generate Session ID

```
session_id = "{YYYYMMDD}-{HHMM}-{4 random alphanumeric chars}"
Example: "20260322-1430-a7x2"
```

### Step 5: Create Session Folder

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/.session-{session_id}",
  content: "---\ntags: [autonexus/session]\nsession: {session_id}\ngoal: {goal}\nstarted: {ISO timestamp}\n---\nSession initialized.",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

---

## Per-Iteration Obsidian Reads (Phase 1)

After the standard Phase 1 steps (read files, git log, TSV), perform 2 Obsidian searches. Only if `OBSIDIAN_AVAILABLE = true`.

### Query 1: Failed Approaches for Target Files

Before modifying files, check what has been tried before and failed:

```
FOR each file likely to be modified this iteration:
  obsidian_global_search({
    query: "{filename}",
    searchInPath: "Projects/{ProjectName}/Iterations/",
    contextLength: 200,
    pageSize: 10
  })

Parse results:
  - Look for notes where frontmatter contains status: discard or status: crash
  - Extract the "Change" and "Rationale" lines from matching notes
  - Feed into ideation: "Avoid: {description} — previously discarded because {reason}"
```

### Query 2: Predict Persona Warnings for Target Files

Check if any predict run flagged concerns about files about to be modified:

```
FOR each file likely to be modified this iteration:
  obsidian_global_search({
    query: "{filename}",
    searchInPath: "Projects/{ProjectName}/Predictions/",
    contextLength: 300,
    pageSize: 5
  })

Parse results:
  - Extract persona name, severity, and recommendation from matching notes
  - Feed into ideation: "Warning from {persona}: {severity} — {recommendation}"
```

Both queries use the retry-then-queue protocol. If both fail, the ideation phase proceeds with git history + TSV only. No warning needed — this is expected graceful degradation.

### How to Use Read Results

- **Failed approach found for target file:** Avoid the same strategy. Try a fundamentally different angle. Mention in the iteration note: "Avoided {X} (previously discarded in iter {N})."
- **Predict warning found for target file:** Factor the severity into risk assessment. If HIGH/CRITICAL warning exists, prefer conservative changes. Mention in iteration note: "Heeded {persona} warning about {issue}."
- **No results found:** Proceed normally. Fresh territory.

---

## Per-Iteration Obsidian Write (Phase 7)

After writing the TSV log entry, write a concise iteration note to Obsidian. Only if `OBSIDIAN_AVAILABLE = true`.

### Iteration Note Template (5-8 lines)

```markdown
---
tags: [autonexus/iteration, autonexus/{project}]
date: {YYYY-MM-DD}
iteration: {N}
status: {keep|discard|crash|no-op|hook-blocked}
metric: {value}
delta: {change from previous best}
commit: {7-char git hash or "-"}
session: {session-id}
---

**Change:** {one-sentence description of what was tried}
**Result:** {status} — metric {old_value} -> {new_value} ({delta})
**Files:** {comma-separated list of modified files}
**Rationale:** {why this change was tried — what hypothesis or pattern motivated it}
**Next:** {what this result suggests for the next iteration}
```

### Write Call

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{session-id}-iter-{N}.md",
  content: "{rendered template}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Uses retry-then-queue protocol. If fails, the TSV log still has the data — no information is lost.

---

## Session End Protocol

Run these steps when the loop terminates (user interrupt, bounded iteration limit reached, or goal achieved).

### Step 1: Write Session Summary Note

```markdown
---
tags: [autonexus/session, autonexus/{project}]
date: {YYYY-MM-DD}
session: {session-id}
goal: {goal text}
baseline: {baseline metric value}
final: {final metric value}
delta: {total improvement}
iterations: {total count}
keeps: {keep count}
discards: {discard count}
crashes: {crash count}
---

# Session: {goal summary}

**Baseline:** {value} -> **Final:** {value} ({delta}, {percentage}%)
**Iterations:** {total} (Keeps: {n} | Discards: {n} | Crashes: {n})
**Duration:** {session duration}

## Top Wins
{List top 3-5 kept iterations by delta, with one-sentence descriptions}

## Key Failed Approaches
{List 2-3 most informative discards — approaches that seemed promising but didn't work, with reasons}

## Observations
{1-2 sentences about patterns observed: what types of changes worked, what didn't}

## Next Session Suggestions
{2-3 concrete suggestions for what to try in the next session based on results}
```

Write to: `Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{session-id}-summary.md`

### Step 2: Append Daily Note Mini-Summary (#25 Enhanced)

```
obsidian_update_note({
  targetType: "periodicNote",
  targetIdentifier: "daily",
  content: "

---

## AutoNexus Session ({HH:MM})

**Project:** [[Projects/{ProjectName}/Dashboard|{ProjectName}]]
**Goal:** {goal}
**Result:** {baseline} → {final} ({delta})
**Iterations:** {total} ({keeps} keeps, {discards} discards)

**Key Wins:**
- [[iter-link-1|{description}]] (+{delta})
- [[iter-link-2|{description}]] (+{delta})

**Key Learning:** {most informative failed approach}

**Decisions Made:** {count} — [[decision-link|{title}]]

[[Projects/{ProjectName}/Iterations/{date}/{session-id}-summary|Full session details]]
",
  modificationType: "wholeFile",
  wholeFileMode: "append",
  createIfNeeded: true
})
```

The enhanced daily note adds:
- Backlinks to individual kept iterations with their deltas
- Link to the project Dashboard (MOC) instead of plain project name
- Decision note links with count
- Unicode arrow (→) for better readability

### Step 3: Final Queue Drain

Attempt full drain of `obsidian-sync-queue.md`. Log results.

---

## Decision Notes

### When to Create

Write a Decision note when:
- A significant architectural choice is made (not every keep — only meaningful decisions)
- 2+ iterations explored the same question from different angles
- The plan wizard completes a configuration
- A predict run produces consensus on a recommendation

### ADR Template

```markdown
---
tags: [autonexus/decision, autonexus/{project}]
date: {YYYY-MM-DD}
status: accepted
iteration: {N or "plan" or "predict"}
commit: {hash if applicable}
---

# Decision: {concise title}

## Context
{What problem or question prompted this decision. 1-3 sentences.}

## Options Considered
1. **{Option A}** — {brief description}. {Why it was chosen OR why rejected.} {If chosen: "← CHOSEN"}
2. **{Option B}** — {brief description}. Rejected: {reason}.
3. **{Option C}** — {brief description}. Rejected: {reason}.

## Decision
{Which option was chosen and the key reason why.}

## Consequences
- {Positive consequence}
- {Trade-off or risk accepted}

## Evidence
- Iteration {N}: metric went from {X} to {Y} ({delta})
- {Any other supporting data}
```

Write to: `Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-{slug}.md`

---

## Knowledge Center Notes

Three notes that model the current codebase. Updated during pre-push rebuild and optionally during predict runs.

### Architecture.md

```markdown
---
tags: [autonexus/knowledge, autonexus/{project}]
commit_hash: {git rev-parse HEAD}
updated_at: {ISO timestamp}
---

# Architecture — {ProjectName}

## Overview
{2-3 sentence high-level description of what this project does}

## Tech Stack
{Languages, frameworks, key libraries, runtime}

## Components
| Component | Path | Purpose | Key Dependencies |
|-----------|------|---------|-----------------|

## Data Flow
{Description of how data moves through the system: entry points → processing → storage → output}

## Key Architectural Decisions
{Links to relevant Decision notes: [[Decisions/{slug}]]}
```

### Components.md

```markdown
---
tags: [autonexus/knowledge, autonexus/{project}]
commit_hash: {git rev-parse HEAD}
updated_at: {ISO timestamp}
---

# Components — {ProjectName}

## Component: {Name}
**Path:** {directory or file glob}
**Purpose:** {what it does}
**Key Files:**
| File | Purpose | Exports |
|------|---------|---------|

**Dependencies:** {what it imports from}
**Dependents:** {what imports from it}
**Risk Areas:** {known fragile points, complexity hotspots}

{Repeat for each major component}
```

### Dependencies.md

```markdown
---
tags: [autonexus/knowledge, autonexus/{project}]
commit_hash: {git rev-parse HEAD}
updated_at: {ISO timestamp}
---

# Dependencies — {ProjectName}

## External Dependencies
| Package | Version | Purpose | Risk |
|---------|---------|---------|------|

## Import Graph (Key Flows)
| Source | Imports From | Symbols Used |
|--------|-------------|-------------|

## Data Flows
| Source | Transform | Sink | Notes |
|--------|-----------|------|-------|
```

---

## Pre-Push Knowledge Rebuild

Run this protocol when the user intends to push (says "push", "git push", or the loop ends and user confirms push). Takes 2-5 minutes.

```
1. Announce: "Running Obsidian knowledge rebuild before push (~2-5 min)..."

2. Read current Knowledge Center from Obsidian:
   - obsidian_read_note({ filePath: "Projects/{ProjectName}/Knowledge/Architecture.md" })
   - obsidian_read_note({ filePath: "Projects/{ProjectName}/Knowledge/Components.md" })
   - obsidian_read_note({ filePath: "Projects/{ProjectName}/Knowledge/Dependencies.md" })
   (If any don't exist yet, that's fine — they'll be created.)

3. Scan the current codebase:
   - Read project structure (directories, key files)
   - Identify components, entry points, exports
   - Map import graph and data flows
   - Detect tech stack and dependencies
   (Same reconnaissance logic as predict Phase 2)

4. Generate updated Knowledge Center content using the templates above.

5. Write all three notes:
   FOR each of [Architecture.md, Components.md, Dependencies.md]:
     obsidian_update_note({
       targetType: "filePath",
       targetIdentifier: "Projects/{ProjectName}/Knowledge/{file}",
       content: {generated content},
       modificationType: "wholeFile",
       wholeFileMode: "overwrite",
       createIfNeeded: true
     })

6. Update frontmatter on each note:
   obsidian_manage_frontmatter({
     filePath: "Projects/{ProjectName}/Knowledge/{file}",
     operation: "set",
     key: "commit_hash",
     value: "{current git rev-parse HEAD}"
   })
   obsidian_manage_frontmatter({
     filePath: "Projects/{ProjectName}/Knowledge/{file}",
     operation: "set",
     key: "updated_at",
     value: "{current ISO timestamp}"
   })

7. Drain obsidian-sync-queue.md

8. Announce: "Knowledge rebuild complete. Safe to push."

9. Proceed with git push.
```

---

## Predict Persona Notes

When `/autonexus:predict` runs, write persona analysis and debate to Obsidian.

### Persona Note Template (1 per persona)

```markdown
---
tags: [autonexus/predict, autonexus/persona, autonexus/{project}]
date: {YYYY-MM-DD}
persona: {persona-name}
role: {persona-role}
findings_count: {N}
session: {predict-session-slug}
commit_hash: {git rev-parse HEAD}
---

# {Persona Name} — {Role}

**Focus:** {focus areas}
**Bias:** {bias direction}
**Scope:** {files analyzed}

## Findings

### {ID}: {title}
- **Severity:** {CRITICAL|HIGH|MEDIUM|LOW}
- **Confidence:** {HIGH|MEDIUM|LOW}
- **Location:** `{file}:{line}`
- **Evidence:** {exact code or flow}
- **Recommendation:** {concrete action}

{Repeat for each finding}
```

Write to: `Projects/{ProjectName}/Predictions/{YYYY-MM-DD}-{slug}/persona-{name}.md`

### Debate Note Template (1 per predict run)

```markdown
---
tags: [autonexus/predict, autonexus/debate, autonexus/{project}]
date: {YYYY-MM-DD}
rounds: {N}
anti_herd_status: {passed|warning}
session: {predict-session-slug}
commit_hash: {git rev-parse HEAD}
---

# Predict Debate — {slug}

## Round {N}

### {Persona Name}
**Target:** {Finding ID}
**Position:** {agree|disagree|concede-with-conditions}
**Counter-evidence:** {evidence}
**Revised position:** {if changed}

{Repeat for each persona's debate contributions per round}

## Consensus

| Finding | Title | Final Severity | Confirmed By | Disputed By | Status |
|---------|-------|---------------|-------------|-------------|--------|

## Anti-Herd Check
- **Flip rate:** {value} (threshold: 0.8)
- **Entropy:** {value} (threshold: 0.3)
- **Status:** {passed|warning — high convergence detected}
```

Write to: `Projects/{ProjectName}/Predictions/{YYYY-MM-DD}-{slug}/debate.md`

---

## Cross-Project Notes

When a lesson from one project applies broadly, write to Cross-Project/.

### When to Create
- A pattern works (or fails) in 2+ projects
- A technique discovered in one project is generalizable
- Agent recognizes a recurring decision pattern across projects

### Template

```markdown
---
tags: [autonexus/cross-project]
projects: [{project1}, {project2}]
discovered: {YYYY-MM-DD}
---

# {Pattern/Lesson Title}

## Pattern
{Description of the generalizable pattern or lesson}

## Evidence
- **{Project A}:** {what happened, iteration reference}
- **{Project B}:** {what happened, iteration reference}

## When to Apply
{Conditions under which this pattern is relevant}

## When NOT to Apply
{Conditions where this pattern would be wrong}
```

Write to: `Cross-Project/{slug}.md`

---

## Pipeline Notes

### Pipeline Definition

Stored at `Projects/{ProjectName}/Pipelines/{name}.md`:

```markdown
---
tags: [autonexus/pipeline]
name: {pipeline-name}
created: {ISO timestamp}
last_run: {ISO timestamp or null}
run_count: {N}
---

# Pipeline: {Human-readable Name}

## Description
{What this pipeline does and when to use it}

## Steps

1. **{command}** — `{args}`
   - Condition: {on-success|on-failure|always|if-metric-above:N}
   - On failure: {skip|halt|retry:N}

{Repeat for each step}

## Variables
- `$SCOPE` = "{value}"
```

### Pipeline Run Log

Written after every pipeline execution at `Projects/{ProjectName}/Pipelines/runs/{run-id}.md`:

```markdown
---
tags: [autonexus/pipeline-run, autonexus/{project}]
pipeline: {name or "inline"}
run_id: {run-id}
started: {ISO timestamp}
completed: {ISO timestamp}
status: {completed|halted|failed}
steps_total: {N}
steps_completed: {N}
---

# Pipeline Run: {name}

## Steps

### Step {N}: /autonexus:{command}
- **Status:** {success|failed|skipped}
- **Duration:** {duration}
- **Summary:** {brief summary}
- **Artifacts:** {links to Obsidian notes created}

{Repeat for each step}
```

---

## Hook Notes

Stored at `Projects/{ProjectName}/Hooks/{name}.md`:

```markdown
---
tags: [autonexus/hook]
name: {hook-name}
enabled: true
priority: 50
created: {ISO timestamp}
last_triggered: {ISO timestamp or null}
trigger_count: {N}
---

# Hook: {Human-readable Name}

## Description
{What this hook does and why}

## Trigger
- **Event:** {event-type}
- **Condition:** {condition expression}

## Action
- **Type:** {action-type}
- **Command:** {if run-command}
- **Message:** {template with $VARIABLES}

## Execution History

| Date | Event | Context | Action Taken | Result |
|------|-------|---------|-------------|--------|
```

Hook notes are updated in place — the `last_triggered`, `trigger_count` frontmatter fields and the Execution History table are appended to after each trigger.

### Local Hook Cache

When hooks are loaded from Obsidian, they are cached to `autonexus-hooks.json` (gitignored). This allows hook evaluation to work even when Obsidian is unavailable in subsequent sessions.

---

## Canvas Notes

Canvas files are written using `obsidian_update_note` with `.canvas` extension. They use the Obsidian Canvas JSON format with `nodes` and `edges` arrays. See `references/canvas-workflow.md` for full canvas type specifications.

Canvas output paths:
- Architecture: `Projects/{ProjectName}/Canvases/architecture.canvas`
- Sprint board: `Projects/{ProjectName}/Canvases/sprint-{N}-board.canvas`
- Dependencies: `Projects/{ProjectName}/Canvases/dependencies.canvas`
- Timeline: `Projects/{ProjectName}/Canvases/timeline.canvas`
- Knowledge map: `Projects/{ProjectName}/Canvases/knowledge-map.canvas`
- Data flow: `Projects/{ProjectName}/Canvases/data-flow.canvas`
- Pipeline: `Projects/{ProjectName}/Canvases/pipeline-{name}.canvas`

All canvas writes use retry-then-queue protocol. Canvas generation is best-effort.

---

## Dashboard Note (Map of Content) (#23)

A Dashboard note is auto-generated/updated at `Projects/{ProjectName}/Dashboard.md` at session end. This serves as a Map of Content (MOC) for the project, aggregating key stats and links.

### Dashboard Template

```markdown
---
tags: [autonexus/dashboard, autonexus/{project}]
updated_at: {ISO timestamp}
---

# {ProjectName} — AutoNexus Dashboard

## Quick Stats
- **Total Sessions:** {count from Iterations/}
- **Total Iterations:** {sum from all sessions}
- **Keep Rate:** {keeps / total}%
- **Knowledge Center:** {fresh|stale} (updated {date})

## Recent Sessions
| Date | Goal | Baseline → Final | Iterations | Keeps |
|------|------|-----------------|------------|-------|
| [[session-link]] | {goal} | {b} → {f} ({delta}) | {n} | {k} |

## Key Decisions
{Links to 5 most recent Decision notes}

## Unresolved Predict Findings
{Links to predict findings with status != resolved}

## Top Strategies
{From strategy learning: which change types succeed most}

## Knowledge Center
- [[Knowledge/Architecture|Architecture]] — {freshness}
- [[Knowledge/Components|Components]] — {freshness}
- [[Knowledge/Dependencies|Dependencies]] — {freshness}
```

### Update Protocol

The Dashboard note is written at session end, AFTER the session summary and daily note have been written:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Dashboard.md",
  content: "{rendered dashboard template}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

To populate the dashboard:
1. List all session summaries: `obsidian_list_notes({ dirPath: "Projects/{ProjectName}/Iterations/", recursionDepth: 2 })`
2. Read frontmatter from recent summaries (up to 10) to extract stats
3. List recent decisions: `obsidian_list_notes({ dirPath: "Projects/{ProjectName}/Decisions/", recursionDepth: 0 })`
4. Check Knowledge Center freshness from Architecture.md frontmatter `commit_hash` and `updated_at`
5. Search for unresolved predict findings: `obsidian_global_search({ query: "status: open", searchInPath: "Projects/{ProjectName}/Predictions/" })`

Uses retry-then-queue protocol. If dashboard generation fails, it does not block session end.

---

## Bookmarks Integration (#24)

AutoNexus uses Obsidian tags to mark notable notes for easy discovery. Users can filter by the `autonexus/bookmarked` tag in Obsidian's search or bookmarks plugin.

### Bookmark Triggers

Add the `autonexus/bookmarked` tag when any of these events occur:

| Event | Condition | When Tagged |
|-------|-----------|-------------|
| Outstanding iteration | `delta > 2x average_delta` for the session | After iteration note is written (Phase 7) |
| Decision created | Any Decision note is written | After Decision note write |
| Critical predict finding | Finding with `severity: CRITICAL` | After persona note write |
| Goal-achieving session | Session achieves the stated goal | After session summary write |

### Implementation

```
obsidian_manage_tags({
  filePath: "{path to the note}",
  operation: "add",
  tags: ["autonexus/bookmarked"]
})
```

This call follows the retry-then-queue protocol. Bookmark tagging is best-effort — if it fails, the note content is still intact.

### User Experience

Users can leverage Obsidian's built-in features to work with bookmarked notes:
- **Search:** `tag:autonexus/bookmarked` to find all notable items
- **Bookmarks plugin:** Star notes with this tag for quick access
- **Dataview queries:** Filter and sort bookmarked notes by date, project, or type

---

## Obsidian Canvas Generation (#20)

AutoNexus generates `.canvas` files (JSON format that Obsidian Canvas understands) for visual representations of predict debates, session flows, and project overviews.

### When Generated

| Trigger | Canvas Type | Path |
|---------|-------------|------|
| After `/autonexus:predict` completes | Persona debate canvas | `Projects/{ProjectName}/Predictions/{date}-{slug}/debate.canvas` |
| After `/autonexus:review` completes | Session flow canvas | `Projects/{ProjectName}/Iterations/{date}/{session-id}-flow.canvas` |
| After `/autonexus:status` | Project overview canvas | `Projects/{ProjectName}/overview.canvas` |

### Canvas JSON Format

Obsidian Canvas files use a JSON format with `nodes` and `edges`:

```json
{
  "nodes": [
    {
      "id": "node-1",
      "type": "file",
      "file": "Projects/MyApp/Predictions/2026-03-22/persona-security-analyst.md",
      "x": 0, "y": 0, "width": 400, "height": 300
    },
    {
      "id": "node-2",
      "type": "text",
      "text": "## Finding SA-1\n**JWT secret missing**\nSeverity: HIGH",
      "x": 500, "y": 0, "width": 300, "height": 200
    }
  ],
  "edges": [
    {
      "id": "edge-1",
      "fromNode": "node-1",
      "toNode": "node-2",
      "label": "flagged"
    }
  ]
}
```

### Predict Debate Canvas

Visual representation of the multi-persona predict debate.

- **Nodes:** Persona notes (type: `file`) + individual findings (type: `text`)
- **Edges:** Challenges and agreements between personas, labeled with the interaction type
- **Color coding:**
  - Red = CRITICAL severity
  - Orange = HIGH severity
  - Yellow = MEDIUM severity
  - Blue = LOW severity
- **Layout:** Personas arranged in a circle, findings radiating outward from each persona

### Session Flow Canvas

Visual representation of an iteration session's progression.

- **Nodes:** Iterations (type: `file` linking to iteration notes)
  - Green background = keep
  - Red background = discard
  - Gray background = crash
- **Edges:** Sequential flow between iterations (iteration N → iteration N+1)
- **Decision nodes:** Highlighted with a border and larger size when a Decision note was created at that point
- **Layout:** Left-to-right flow, with keeps on the top track and discards on the bottom track

### Write Protocol

Canvas files are written using `obsidian_update_note` with the `.canvas` extension:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Predictions/{date}-{slug}/debate.canvas",
  content: "{canvas JSON string}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Uses retry-then-queue protocol. Canvas generation is best-effort — if it fails, the underlying markdown notes still contain all the data.

---

## Obsidian Unavailable Fallback

There are two distinct unavailable states:

### State A: Obsidian MCP Not Configured (`OBSIDIAN_NOT_CONFIGURED = true`)

The user installed AutoNexus but has no Obsidian MCP server in their `.mcp.json`. The Obsidian tools don't exist in Claude's tool set at all.

1. ALL Obsidian operations are skipped entirely (no tool calls attempted, no retry, no queue)
2. Local TSV logging + git-as-memory work exactly as normal
3. The single warning from Phase 0 Step 1 is sufficient
4. Do NOT create `obsidian-sync-queue.md` (there's nothing to queue — the tools don't exist)
5. AutoNexus runs as a fully functional iteration engine identical to autoresearch
6. When session ends, note: "Obsidian MCP not configured. Running in local-only mode."

### State B: Obsidian MCP Configured But Unavailable (`OBSIDIAN_AVAILABLE = false`)

The tools exist but the connection failed (Obsidian closed, wrong API key, etc.).

1. ALL Obsidian MCP calls become immediate no-ops (return null, no retry, no queue)
2. Local TSV logging + git-as-memory continue exactly as normal
3. The single warning from Phase 0 Step 1b is sufficient
4. Failed writes DO get queued to `obsidian-sync-queue.md` (tools exist, connection may recover)
5. The loop runs identically to a non-Obsidian system
6. When session ends, note: "Obsidian was unavailable this session. Knowledge notes were not written. Queued operations saved for next session."

### In Both States, Do NOT:

- Repeat the warning every iteration
- Ask the user to fix it mid-session
- Suggest restarting Obsidian
- Degrade loop behavior in any other way
- Refuse to run because Obsidian isn't available

**AutoNexus is a fully functional autonomous iteration engine with or without Obsidian. Obsidian adds persistent knowledge — it is never a prerequisite.**
