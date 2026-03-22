# /autonexus:status — Project Health Dashboard

Read-only dashboard that shows the current state of AutoNexus for your project. Displays session history, iteration statistics, strategy effectiveness, Knowledge Center freshness, and unresolved findings.

## Syntax

```
# Show status for the current project
/autonexus:status

# Show status for a specific project
/autonexus:status --project my-api
```

## What It Does

Gathers data from Obsidian and local sources, computes statistics, and prints a dashboard to the terminal. Makes no modifications to anything.

## Example Output

### With Obsidian

```
=== AutoNexus Status: my-api ===

Knowledge Center:  stale (12 commits behind, updated 2026-03-18)
Sessions:          7 total, 2 this week
Iterations:        142 (89 keeps, 48 discards, 5 crashes)
Keep Rate:         62.7%
Sync Queue:        empty

Recent Sessions:
  2026-03-21 — reduce bundle size — 245KB->198KB (8/12)
  2026-03-19 — improve test coverage — 72%->88% (19/20)
  2026-03-15 — fix auth latency — 340ms->120ms (5/9)

Recent Decisions:
  2026-03-21 — switch from webpack to esbuild
  2026-03-19 — add integration test layer

Unresolved Findings:
  HIGH — SQL injection in search endpoint (Security Analyst)
  MEDIUM — missing error boundary in dashboard (Reliability Engineer)

Top Strategies: testing (95%), caching (80%), error-handling (67%)
```

### Without Obsidian (Local Data Only)

```
=== AutoNexus Status: my-api (local data only) ===

Sessions:          ~4 (from TSV gaps)
Iterations:        87 (52 keeps, 32 discards, 3 crashes)
Keep Rate:         59.8%
Sync Queue:        3 pending items

Recent Activity (from git log):
  2026-03-21 — experiment(bundle): switch to esbuild for faster builds
  2026-03-21 — experiment(bundle): enable tree shaking on utils

Note: Connect Obsidian for full status (Knowledge Center, Decisions, Predictions, Strategy Analysis).
```

## Obsidian vs Local-Only Differences

| Feature | With Obsidian | Without Obsidian |
|---------|---------------|------------------|
| Session count | Exact (from session summaries) | Estimated (from TSV time gaps) |
| Session details | Goal, baseline, final metric, keeps/total | Commit messages only |
| Knowledge Center | Freshness check (commits behind) | Unavailable |
| Decisions | Recent decision titles + dates | Unavailable |
| Predict findings | Unresolved findings with severity | Unavailable |
| Strategy analysis | Keep rate per category | Unavailable |
| Keep rate | Exact | Exact (from TSV) |
| Sync queue | Pending item count | Pending item count |

## Tips

- **Run before starting a session** to see what strategies have worked best and whether there are unresolved predict findings to address.
- **Check Knowledge Center freshness** — if it shows "stale," run `/autonexus:learn` to update before your next session.
- **Monitor keep rate** over time. A declining keep rate suggests the low-hanging fruit has been picked and more creative strategies are needed.
- **Review unresolved findings** — these are warnings from `/autonexus:predict` that haven't been addressed yet. They may point to the highest-impact changes.
- **No flags needed** for the common case. Just run `/autonexus:status` and get the full picture.
