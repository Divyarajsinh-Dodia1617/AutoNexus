# /autonexus:review — Post-Session Analysis

Analyzes a completed session to find patterns in what worked and what didn't. Identifies turning points, diminishing returns, file impact, and strategy effectiveness. Writes a Lessons Learned note to Obsidian.

## Syntax

```
# Interactive session picker (default)
/autonexus:review

# Review the most recent session
/autonexus:review --last

# Review a specific session
/autonexus:review --session 20260320-1430-x7q2

# Review with cross-project insights
/autonexus:review --last --cross-project
```

## What It Does

1. **Selects** a session to review (interactive picker, `--last`, or `--session`)
2. **Loads** all iteration notes from the selected session
3. **Analyzes patterns:**
   - Which files appeared most in keeps vs discards
   - Which change categories succeeded vs failed
   - Where the "turning point" was (failing to succeeding)
   - When diminishing returns started
4. **Generates insights** — printed to terminal
5. **Writes Lessons Learned** to Obsidian (`Decisions/{date}-review-{session-id}.md`)
6. **Optionally writes Cross-Project note** for patterns that apply beyond this project
7. **Generates Session Flow Canvas** — visual diagram of keeps/discards

## Flags

| Flag | Purpose |
|------|---------|
| `--last` | Review the most recent session (skip picker) |
| `--session <id>` | Review a specific session by its ID |
| `--cross-project` | Also write a Cross-Project note if patterns are generalizable |

## Examples

```bash
# Quick review of the last session
/autonexus:review --last

# Deep review of a specific session with cross-project learning
/autonexus:review --session 20260318-0900-k3m1 --cross-project

# Interactive — pick from recent sessions
/autonexus:review
```

## Obsidian Integration

### What Gets Read
- **Session summary:** Goal, scope, baseline, final metric, keep/discard counts
- **All iteration notes:** Status, change description, files modified, metric values, rationale

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Lessons Learned note | `Projects/{ProjectName}/Decisions/{date}-review-{session-id}.md` | Always (if Obsidian available) |
| Cross-Project note | `Cross-Project/{slug}.md` | Only with `--cross-project` flag |
| Session Flow Canvas | `Projects/{ProjectName}/Iterations/{date}/{session-id}-flow.canvas` | Always (if Obsidian available) |

### Lessons Learned Note Contains
- What worked (strategies + files with high keep rates)
- What didn't work (strategies + files with high discard rates)
- Turning point description
- Diminishing returns analysis
- Metric trajectory (early vs late improvement rate)
- Recommendations for future sessions

### Cross-Project Note Contains
- Generalizable pattern description
- Evidence from the source session
- Recommendation for applying the pattern in other projects

### Session Flow Canvas
Visual representation of the session:
- Green nodes = keeps
- Red nodes = discards
- Orange nodes = crashes
- Arrows show iteration sequence
- Metric value on each node
- Turning point highlighted

## Without Obsidian

If Obsidian is unavailable:
- Session data is loaded from `autonexus-results.tsv` (reduced detail)
- All insights are printed to the terminal
- No Obsidian notes or canvas are written
- Pattern analysis works but with less granularity (no file lists, no rationale text)

## Tips

- **Review after every significant session** to build up strategy knowledge. The Lessons Learned notes inform future `/autonexus` sessions via strategy learning.
- **Use `--cross-project`** when you notice patterns that aren't specific to your project (e.g., "caching always improves API latency" or "refactoring rarely improves metrics directly").
- **Check the turning point** — understanding what caused the breakthrough helps you start future sessions closer to the right approach.
- **Watch for diminishing returns** — if the review shows improvement slowed after iteration 15, consider using `--iterations 15` for future sessions on similar goals.
- **The Session Flow Canvas** in Obsidian gives a visual overview that's faster to scan than reading individual iteration notes.
