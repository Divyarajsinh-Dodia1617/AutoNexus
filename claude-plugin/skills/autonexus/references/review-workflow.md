# Review Workflow — /autonexus:review

Post-session analysis that identifies patterns in what worked and what didn't. Reads all iteration notes from a completed session, performs pattern analysis, generates actionable insights, and writes a Lessons Learned note to Obsidian.

**Core idea:** Learn from past sessions systematically. Which files drove improvement? Which approaches consistently failed? Where was the turning point? What are the diminishing returns? Turn raw iteration data into strategic knowledge for future sessions.

## Trigger

- User invokes `/autonexus:review`
- User says "review the last session", "what worked", "analyze my session", "lessons learned"

## Architecture

```
/autonexus:review
  ├── Step 1: Select session
  ├── Step 2: Load all iteration notes
  ├── Step 3: Pattern analysis
  ├── Step 4: Generate insights
  ├── Step 5: Write Lessons Learned note (Obsidian)
  ├── Step 6: Optionally write Cross-Project note
  └── Step 7: Generate Session Flow Canvas
```

## Step 1: Select Session

### If `--last` flag is set:
Select the most recent session automatically.

### If `--session <id>` flag is set:
Find the specific session by ID.

### If no flag (default — interactive):

Query Obsidian for recent sessions:

```
IF OBSIDIAN_AVAILABLE:
  obsidian_global_search({
    query: "autonexus/session",
    searchInPath: "Projects/{ProjectName}/Iterations/",
    contextLength: 400,
    pageSize: 10
  })
```

Use `AskUserQuestion` to present the last 3-5 sessions:

```
AskUserQuestion with a single question:

Header: "Review Session"
Question: "Which session would you like to review?"
Options:
  - "{date} — {goal} — {baseline}->{final} ({keeps}/{total} keeps)" (for each session)
```

### Fallback (Obsidian unavailable):
Parse `autonexus-results.tsv` to identify sessions by time gaps. Present available sessions based on local data.

## Step 2: Load All Iteration Notes

### If OBSIDIAN_AVAILABLE:

Read all iteration notes for the selected session:

```
obsidian_global_search({
  query: "{session-id}",
  searchInPath: "Projects/{ProjectName}/Iterations/",
  contextLength: 500,
  pageSize: 50
})
```

For each iteration note, extract:
- Iteration number
- Status (keep / discard / crash / no-op / hook-blocked)
- Change description
- Change category (caching, refactoring, testing, etc.)
- Files modified
- Metric before and after
- Rationale for keep/discard
- Commit hash (if keep)

Also read the session summary for the goal, scope, and overall stats.

### Fallback (Obsidian unavailable):
Read `autonexus-results.tsv` and filter rows belonging to the selected session. Extract iteration, commit, metric, status, and description columns.

## Step 3: Pattern Analysis

Analyze the loaded iteration data to identify patterns:

### 3a. Group by Outcome

```
keeps = [iterations where status == "keep" or "keep (reworked)"]
discards = [iterations where status == "discard"]
crashes = [iterations where status == "crash"]
no_ops = [iterations where status == "no-op" or "hook-blocked"]
```

### 3b. File Impact Analysis

Identify which files appeared most frequently in keeps vs discards:

```
file_keep_count = {}
file_discard_count = {}

FOR each kept iteration:
  FOR each file modified:
    file_keep_count[file] += 1

FOR each discarded iteration:
  FOR each file modified:
    file_discard_count[file] += 1

# Files that drove improvement (high keep rate)
improvement_drivers = files where keep_count / (keep_count + discard_count) > 0.6

# Files that resisted improvement (high discard rate)
resistant_files = files where discard_count / (keep_count + discard_count) > 0.7
```

### 3c. Category Analysis

Group iterations by change category and calculate success rates:

```
category_stats = {}
FOR each iteration:
  category = categorize(change_description)  # caching, refactoring, testing, etc.
  category_stats[category].total += 1
  IF status == "keep":
    category_stats[category].keeps += 1

# Sort by keep rate
successful_categories = sort(category_stats, by: keep_rate, descending)
failed_categories = sort(category_stats, by: keep_rate, ascending)
```

### 3d. Turning Point Detection

Find the iteration where the session shifted from failing to succeeding (or vice versa):

```
# Look for longest consecutive keep streak
# The start of that streak is the "turning point"
# Also look for the point where discards started dominating (diminishing returns)

consecutive_keeps = find_longest_streak(iterations, status == "keep")
turning_point = consecutive_keeps.start_iteration

# Diminishing returns: point where keep rate drops below 30% for remaining iterations
FOR i in range(len(iterations)):
  remaining = iterations[i:]
  remaining_keep_rate = count(remaining, status=="keep") / len(remaining)
  IF remaining_keep_rate < 0.3:
    diminishing_returns_point = i
    BREAK
```

### 3e. Metric Trajectory

Track how the metric evolved over the session:

```
metric_trajectory = []
FOR each iteration (in order):
  metric_trajectory.append({
    iteration: N,
    metric: value,
    delta_from_baseline: value - baseline,
    status: keep/discard/crash
  })

# Calculate rate of improvement
early_rate = avg improvement per keep in first half
late_rate = avg improvement per keep in second half
```

## Step 4: Generate Insights

Synthesize the pattern analysis into human-readable insights:

```
Insights to generate:

1. "Files that drove improvement: {list of improvement_drivers with keep rates}"

2. "Files that resisted improvement: {list of resistant_files}"
   (only if any exist)

3. "Approaches that consistently worked: {successful_categories with rates}"

4. "Approaches that consistently failed: {failed_categories with rates}"

5. "The breakthrough came at iteration {turning_point} when {description}"
   (only if a clear turning point exists)

6. "Diminishing returns started at iteration {N} — keep rate dropped to {rate}%"
   (only if diminishing returns detected)

7. "Metric trajectory: {baseline} -> {final} ({delta}),
    early rate: {early_rate}/keep, late rate: {late_rate}/keep"

8. "Session efficiency: {keeps}/{total} iterations were productive ({keep_rate}%)"

9. "Recommendation for next session: {based on analysis}"
   - If high keep rate in a category → "Continue {category} approach"
   - If diminishing returns → "Try a different strategy or scope"
   - If crashes were common → "Improve guard coverage before next session"
```

Print all insights to the terminal.

## Step 5: Write Lessons Learned Note

### If OBSIDIAN_AVAILABLE:

Write a Lessons Learned note to Obsidian:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-review-{session-id}.md",
  content: "{rendered lessons learned note}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Note template:

```markdown
---
tags: [autonexus/review, autonexus/{project}]
date: {YYYY-MM-DD}
session: {session-id}
goal: {goal}
iterations: {total}
keeps: {count}
discards: {count}
crashes: {count}
keep_rate: {rate}%
baseline: {baseline}
final: {final}
---

# Session Review: {goal}

**Session:** {session-id} | **Date:** {date}
**Iterations:** {total} | **Keep Rate:** {keep_rate}%
**Metric:** {baseline} -> {final} ({delta})

## What Worked
{List of successful strategies with keep rates}
{Files that drove improvement}

## What Didn't Work
{List of failed strategies with keep rates}
{Files that resisted improvement}

## Turning Point
{Description of the breakthrough moment, if any}

## Diminishing Returns
{Where improvement slowed down, if detected}

## Metric Trajectory
{Summary of early vs late improvement rate}

## Recommendations
{Actionable suggestions for future sessions}

## Related
- Session: [[Projects/{ProjectName}/Iterations/{date}/{session-id}-summary]]
- Iterations: [[Projects/{ProjectName}/Iterations/{date}/]]
```

### Fallback (Obsidian unavailable):
Print the full analysis to the terminal only. No Obsidian writes.

## Step 6: Cross-Project Note (Optional)

If `--cross-project` flag is set AND the patterns are generalizable (not project-specific), write a Cross-Project note:

```
Generalizable patterns:
- Strategy keep rates that are consistent across projects
- Anti-patterns that fail regardless of project
- Techniques with broad applicability (e.g., "caching always works for API latency")
```

Only write if the insight is genuinely cross-project — not project-specific quirks.

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Cross-Project/{slug}.md",
  content: "{rendered cross-project note}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Cross-project note template:

```markdown
---
tags: [autonexus/cross-project]
date: {YYYY-MM-DD}
source_project: {ProjectName}
source_session: {session-id}
---

# {Insight Title}

**Observed in:** {ProjectName} ({date})

## Pattern
{Description of the generalizable pattern}

## Evidence
{Keep rate, iteration count, and specific examples from the session}

## Recommendation
{How to apply this pattern in other projects}
```

## Step 7: Generate Session Flow Canvas

If OBSIDIAN_AVAILABLE, generate a visual canvas representation of the session flow per the canvas protocol in `obsidian-integration.md`.

The canvas shows:
- Each iteration as a node (color-coded: green=keep, red=discard, orange=crash)
- Arrows connecting iterations in sequence
- Metric value displayed on each node
- Turning point highlighted
- Branch points (if parallel exploration was used)

Canvas path: `Projects/{ProjectName}/Iterations/{date}/{session-id}-flow.canvas`

If Obsidian is unavailable or canvas creation fails, skip this step silently.

## Flags

| Flag | Purpose |
|------|---------|
| `--last` | Review the most recent session (skip picker) |
| `--session <id>` | Review a specific session by its ID |
| `--cross-project` | Also write a Cross-Project note if patterns are generalizable |

## Error Handling

| Scenario | Response |
|----------|----------|
| No past sessions found | Print "No sessions found to review. Run /autonexus first." and stop. |
| Session ID not found | Print "Session {id} not found." and fall back to interactive picker. |
| Not enough iterations (<3) | Print "Session has too few iterations for meaningful analysis." Show raw data instead. |
| Obsidian read fails | Fall back to local TSV analysis. Print insights to terminal only. |
| Obsidian write fails | Use retry-then-queue protocol. Insights are already printed to terminal. |
