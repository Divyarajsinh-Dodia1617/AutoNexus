# /autonexus:resume — Resume a Previous Session

Pick up where you left off. Reloads the goal, scope, metric, and verification config from a past session and continues iterating from the current codebase state.

## Syntax

```
# Interactive session picker (default)
/autonexus:resume

# Resume the most recent session
/autonexus:resume --last

# Resume a specific session by ID
/autonexus:resume --session 20260320-1430-x7q2

# Resume with bounded iterations
/autonexus:resume --last --iterations 15
```

## What It Does

1. **Discovers** past sessions from Obsidian (or local TSV + git as fallback)
2. **Presents** the last 3-5 sessions with goal, date, metric progress, and keep/discard counts
3. **Loads** the selected session's configuration (goal, scope, verify, guard, direction)
4. **Verifies** the current metric (the codebase may have changed since last session)
5. **Resumes** the autonomous loop using the loaded config with the current metric as the new baseline
6. **Links** the new session to the resumed session in Obsidian for traceability

## Flags

| Flag | Purpose |
|------|---------|
| `--last` | Skip the session picker, resume the most recent session |
| `--session <id>` | Resume a specific session by its session ID |
| `--iterations N` | Run exactly N iterations then stop |

## Examples

```bash
# Interrupted yesterday, want to continue (interactive picker)
/autonexus:resume

# Quick continuation of the last session
/autonexus:resume --last

# Resume a specific session with a bounded iteration count
/autonexus:resume --session 20260318-0900-k3m1 --iterations 20
```

## Obsidian Integration

### What Gets Read
- **Session summaries:** Queries `Projects/{ProjectName}/Iterations/` for past session summary notes to populate the session picker and load configuration.

### What Gets Written
- **Session linking:** The new session summary includes a `resumed_from` field pointing to the old session. The old session summary gets a forward link to the new session.
- **Standard iteration notes:** All iteration notes, decision notes, and the session summary are written as normal during the resumed loop.

### Session Chain
Sessions form a linked chain in Obsidian:
```
Session A (original) → Session B (resumed from A) → Session C (resumed from B)
```
Each session summary links forward and backward, making it easy to trace the full history of an optimization effort.

## Without Obsidian

If Obsidian is unavailable, the resume workflow falls back to:
- Parsing `autonexus-results.tsv` for recent metric data
- Reading `git log` for experiment commit messages
- Asking the user to fill in any missing config fields (goal, verify command, etc.)

The loop runs identically once configuration is loaded — only the discovery and linking steps are affected.

## Tips

- **Use `--last` for speed** when you know you want to continue the most recent session.
- **Check the metric** after resuming — if the codebase changed between sessions (manual edits, other developers), the baseline will reflect the current state, not the old session's final metric.
- **Chain multiple resumes** for long-running optimization efforts that span days or weeks. The Obsidian session chain tracks the full journey.
- **Combine with `--iterations`** to run a bounded continuation: "give me 10 more iterations on the same goal."
- If the verify command has changed since the original session, the resume workflow will detect the failure and let you provide an updated command.
