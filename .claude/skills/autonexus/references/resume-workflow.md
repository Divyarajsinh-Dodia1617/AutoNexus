# Resume Workflow — /autonexus:resume

Resume a previous AutoNexus session from where it left off. Reloads the goal, scope, metric, and configuration from a past session and continues iterating from the current codebase state.

**Core idea:** Pick up where you stopped. No need to re-specify the goal, scope, or verification command — everything is restored from the session record. The new session links back to the resumed one for continuity.

## Trigger

- User invokes `/autonexus:resume`
- User says "continue where I left off", "resume last session", "pick up from yesterday"

## Loop Support

```
# Resume most recent session
/autonexus:resume --last

# Resume a specific session by ID
/autonexus:resume --session 20260320-1430-x7q2

# Interactive session picker (default)
/autonexus:resume
```

## PREREQUISITE: Session Discovery

Before resuming, the workflow must locate past session data. Two sources are available:

1. **Obsidian (primary):** Query for session summaries
2. **Local fallback:** Read `autonexus-results.tsv` + git log

## Step 1: Discover Sessions

### If OBSIDIAN_AVAILABLE:

```
obsidian_global_search({
  query: "autonexus/session",
  searchInPath: "Projects/{ProjectName}/Iterations/",
  contextLength: 400,
  pageSize: 10
})
```

Parse results to extract session summaries. For each session, collect:
- Session ID
- Date
- Goal
- Scope
- Metric command (verify)
- Guard command (if any)
- Direction (higher/lower)
- Baseline metric value
- Final metric value
- Keep count / Discard count / Crash count
- Total iterations

Sort by date descending (most recent first).

### If OBSIDIAN unavailable (fallback):

```
1. Read autonexus-results.tsv — group entries by session gaps (>1 hour between entries = new session)
2. Read git log --oneline -50 — look for "experiment(" commit messages to identify session boundaries
3. Extract what's possible: metric values, keep/discard counts, descriptions
4. Note: goal, scope, verify, guard cannot be fully reconstructed from TSV alone
```

## Step 2: Present Options

### If `--last` flag is set:
Skip the picker. Select the most recent session automatically.

### If `--session <session-id>` flag is set:
Find the specific session by ID. If not found, show an error and fall back to the interactive picker.

### If no flag (default — interactive):

Use `AskUserQuestion` to present the last 3-5 sessions:

```
AskUserQuestion with a single question:

Header: "Resume Session"
Question: "Which session would you like to resume?"
Options:
  - "{date} — {goal} — {baseline}→{final} ({keeps}/{total} keeps)" (for each session)
  - "None — start fresh instead"
```

If user selects "None," inform them to use `/autonexus` to start a new session and stop.

## Step 3: Load Configuration

Read the selected session summary from Obsidian (or reconstruct from local data):

```
IF OBSIDIAN_AVAILABLE:
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{session-id}-summary.md"
  })

Extract from frontmatter and content:
  - goal → Goal for the new session
  - scope → Scope (file globs)
  - metric_command → Verify command
  - guard_command → Guard command (may be null)
  - direction → "higher" or "lower"
  - final_metric → Last known metric value
  - keeps / discards / crashes → Historical context
```

If any critical field is missing (goal, scope, verify), use `AskUserQuestion` to fill in the gaps:

```
AskUserQuestion:
  Header: "Missing Configuration"
  Question: "The session record is incomplete. Please provide the missing fields."
  (List which fields are missing and ask for them)
```

## Step 4: Verify Current State

Run the verify command to get the current metric value. The codebase may have changed since the last session (manual edits, other developers, etc.).

```
current_metric = run(verify_command)

IF current_metric differs significantly from final_metric:
  Print: "Note: Current metric ({current_metric}) differs from last session's final ({final_metric}).
          The codebase has changed since the last session.
          Using current metric as the new baseline."

baseline = current_metric
```

## Step 5: Resume the Autonomous Loop

Start the autonomous loop with the loaded configuration:

```
Goal: {goal from session}
Scope: {scope from session}
Metric: {metric from session}
Direction: {direction from session}
Verify: {verify command from session}
Guard: {guard command from session, if any}
Baseline: {current_metric} (freshly measured)
```

The loop proceeds identically to a fresh `/autonexus` invocation — all phases (Review, Ideate, Modify, Commit, Verify, Guard, Decide, Log) apply normally.

## Step 6: Link Sessions in Obsidian

### If OBSIDIAN_AVAILABLE:

Write a linking note connecting the new session to the resumed session:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{new-session-id}-summary.md",
  content: "... resumed_from: {old-session-id} ...",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

The new session summary frontmatter includes:
```yaml
resumed_from: {old-session-id}
resumed_baseline: {current_metric}
original_baseline: {original baseline from first session}
```

Also update the old session summary to add a forward link:
```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{old-date}/{old-session-id}-summary.md",
  content: "... (append) Resumed in session: [[{new-session-id}-summary]] ...",
  modificationType: "append"
})
```

This creates a bidirectional link chain: old session → new session → next session, making it easy to trace the full history of an optimization effort.

## Fallback: Obsidian Unavailable

If Obsidian is completely unavailable:

1. **Discovery:** Parse `autonexus-results.tsv` for recent session data. Group by time gaps.
2. **Configuration:** Attempt to reconstruct from git log messages (`experiment(scope): description` patterns) and TSV metric columns.
3. **Missing fields:** Use `AskUserQuestion` to fill in goal, verify command, guard command, and direction — these cannot be reliably reconstructed from TSV/git alone.
4. **Linking:** No Obsidian linking. Print a note: "Session resumed without Obsidian — no cross-session linking available."
5. **Proceed:** Start the loop with whatever config was gathered.

## Session End

When the resumed session completes, the Session End Protocol from `autonomous-loop-protocol.md` applies as normal. The session summary will include the `resumed_from` field for traceability.

## Flags

| Flag | Purpose |
|------|---------|
| `--last` | Skip session picker, resume the most recent session |
| `--session <id>` | Resume a specific session by its ID |

## Error Handling

| Scenario | Response |
|----------|----------|
| No past sessions found | Print "No previous sessions found. Use /autonexus to start a new session." and stop. |
| Session ID not found | Print "Session {id} not found." and fall back to interactive picker. |
| Verify command fails | Print error, ask user if the verify command has changed, allow override. |
| Scope files no longer exist | Warn user, ask if scope should be updated, allow override. |
| Obsidian read fails | Fall back to local TSV + git reconstruction. |
