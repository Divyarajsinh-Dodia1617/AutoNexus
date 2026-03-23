# /autonexus:optimize — Token Optimization Dashboard

Provides visibility into API token usage across agents, sessions, and workflows. Shows a breakdown of where tokens went, how smart routing decisions performed, cache hit rates, and actionable recommendations to reduce costs. Run after sessions to understand spending and before sessions to apply optimizations.

## Syntax

```
# Full optimization dashboard
/autonexus:optimize

# Show just the dashboard summary
/autonexus:optimize --dashboard

# View smart routing decisions log
/autonexus:optimize --routing-log

# View cache statistics
/autonexus:optimize --cache-stats

# Get cost reduction recommendations
/autonexus:optimize --recommendations

# Reset statistics for a fresh start
/autonexus:optimize --reset
```

## What It Does

1. **Dashboard** — Aggregates token usage across all recent sessions, broken down by agent, workflow, and model tier.
2. **Routing log** — Shows how the smart router assigned tasks to model tiers (e.g., simple tasks to faster models, complex tasks to stronger models).
3. **Cache stats** — Reports how often cached results were reused, avoiding redundant API calls.
4. **Recommendations** — Analyzes usage patterns and suggests specific optimizations: better routing rules, cache opportunities, workflow adjustments.

## Flags

| Flag | Purpose |
|------|---------|
| `--dashboard` | Show the summary dashboard with tier breakdown |
| `--routing-log` | Show the smart routing decision log |
| `--cache-stats` | Show cache hit/miss statistics |
| `--recommendations` | Generate actionable cost reduction suggestions |
| `--reset` | Reset all statistics counters |

## Examples

```bash
# Full dashboard after a session
/autonexus:optimize

# Output:
# ┌─────────────────────────────────────────────────────────┐
# │ Token Optimization Dashboard                             │
# ├─────────────────────────────────────────────────────────┤
# │ Total tokens (last 7 days):  284,500                     │
# │ Estimated cost:              $4.27                        │
# │ Tokens saved by cache:       41,200 (14.5%)               │
# │ Tokens saved by routing:     28,800 (10.1%)               │
# ├─────────────────────────────────────────────────────────┤
# │ Tier Breakdown:                                           │
# │   Tier 1 (fast):    112,000 tokens  (39%)  — $0.56       │
# │   Tier 2 (balanced): 98,500 tokens  (35%)  — $1.48       │
# │   Tier 3 (strong):   74,000 tokens  (26%)  — $2.23       │
# ├─────────────────────────────────────────────────────────┤
# │ By Agent:                                                 │
# │   Code Agent:       89,000 tokens  (31%)                  │
# │   QA Agent:         62,000 tokens  (22%)                  │
# │   Debug Agent:      48,000 tokens  (17%)                  │
# │   Architect:        41,500 tokens  (15%)                  │
# │   Other:            44,000 tokens  (15%)                  │
# └─────────────────────────────────────────────────────────┘

# View routing log
/autonexus:optimize --routing-log

# Output:
# Recent routing decisions:
#   [14:32] Code Agent → Tier 3 (complex implementation) — correct
#   [14:33] QA Agent → Tier 2 (test generation) — correct
#   [14:34] Debug Agent → Tier 1 (simple log analysis) — could downgrade
#   [14:35] Ship Agent → Tier 1 (git operations) — correct

# Get recommendations
/autonexus:optimize --recommendations

# Output:
# Recommendations (est. 35% cost reduction):
#   1. Route QA test boilerplate to Tier 1 (save ~18,000 tokens/week)
#   2. Enable caching for repeated architecture queries (save ~12,000 tokens/week)
#   3. Use --scope flags to reduce context size for Code Agent (save ~8,000 tokens/week)
#   4. Batch similar debug tasks instead of running individually (save ~5,000 tokens/week)
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Dashboard snapshot | `Projects/{ProjectName}/Optimization/dashboard-{date}.md` | After --dashboard |
| Routing log | `Projects/{ProjectName}/Optimization/routing-log.md` | After --routing-log |
| Recommendations | `Projects/{ProjectName}/Optimization/recommendations.md` | After --recommendations |

### What Gets Read
- Session token data from `Projects/{ProjectName}/Sessions/`
- Historical optimization data for trend analysis

## Without Obsidian

- `--dashboard` shows current session data only (no historical aggregation)
- `--routing-log` works for the current session
- `--cache-stats` works for the current session
- `--recommendations` are less accurate without historical data but still functional
- `--reset` has no effect without Obsidian (session data is ephemeral)

## Tips

- **Run after sessions** to see where tokens went. Even a quick glance at the dashboard reveals whether agents are spending efficiently.
- **Follow `--recommendations`** — applying the top 2-3 suggestions typically reduces costs 30-50%.
- **Check `--routing-log` if costs seem high** — incorrect tier assignments (complex tasks on expensive tiers when they could use cheaper ones) are the most common waste.
- **`--cache-stats` reveals redundancy** — if cache hit rate is below 10%, you may be re-running similar analyses unnecessarily.
- **Run `--dashboard` weekly** to track cost trends. Sudden spikes usually indicate a workflow change that needs tuning.
- **Pair with `/autonexus:auto-agent --budget`** to enforce spending limits based on insights from the dashboard.
