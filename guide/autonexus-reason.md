# /autonexus:reason — ReasoningBank Self-Learning

Manages AutoNexus's ReasoningBank — a persistent store of strategies, patterns, and outcomes learned across sessions. View effectiveness stats, search for patterns, prune stale entries, and import/export knowledge between projects. The ReasoningBank is what allows AutoNexus to improve over time.

## Syntax

```
# View learning statistics
/autonexus:reason --stats

# Search for learned patterns
/autonexus:reason --search "caching strategies"

# Show top strategies by effectiveness
/autonexus:reason --top 10

# Run meta-analysis across all patterns
/autonexus:reason --meta-analysis

# Prune low-effectiveness patterns
/autonexus:reason --prune --threshold 0.3

# Export patterns for sharing
/autonexus:reason --export reasoning-bank-2026.json

# Import patterns from another project
/autonexus:reason --import /path/to/other-project/reasoning-bank.json
```

## What It Does

1. **`--stats`** — Shows a dashboard of ReasoningBank health: total patterns, average effectiveness, category breakdown, recent additions, decay rate.
2. **`--search`** — Finds patterns matching a query, ranked by effectiveness. Shows the strategy, when it was learned, success rate, and last used date.
3. **`--top`** — Lists the N most effective strategies with their scores and usage counts.
4. **`--meta-analysis`** — Runs a cross-pattern analysis to find meta-insights: which categories are strongest, where gaps exist, contradictory patterns.
5. **`--prune`** — Removes patterns below the effectiveness threshold. Asks for confirmation before deleting.
6. **`--export / --import`** — Serializes or loads ReasoningBank data for cross-project knowledge transfer.

## Flags

| Flag | Purpose |
|------|---------|
| `--stats` | Display ReasoningBank statistics dashboard |
| `--search <query>` | Search for patterns matching the query |
| `--top <n>` | Show the top N strategies by effectiveness (default: 5) |
| `--meta-analysis` | Run cross-pattern meta-analysis |
| `--prune` | Remove low-effectiveness patterns |
| `--threshold <n>` | Effectiveness threshold for pruning (0.0–1.0, default: 0.2) |
| `--export <file>` | Export ReasoningBank to a JSON file |
| `--import <file>` | Import ReasoningBank from a JSON file |

## Examples

```bash
# Check learning stats
/autonexus:reason --stats

# Output:
# ┌─────────────────────────────────────────────┐
# │ ReasoningBank Statistics                     │
# ├─────────────────────────────────────────────┤
# │ Total patterns:          142                 │
# │ Avg effectiveness:       0.72                │
# │ Active (used <30d):      98                  │
# │ Stale (unused >30d):     44                  │
# │ Added this week:         12                  │
# ├─────────────────────────────────────────────┤
# │ Category Breakdown:                          │
# │   debugging       38 patterns  (avg 0.81)    │
# │   architecture    27 patterns  (avg 0.74)    │
# │   testing         24 patterns  (avg 0.69)    │
# │   security        19 patterns  (avg 0.77)    │
# │   performance     16 patterns  (avg 0.65)    │
# │   refactoring     18 patterns  (avg 0.68)    │
# └─────────────────────────────────────────────┘

# Search for caching-related patterns
/autonexus:reason --search "caching strategies"

# Output:
# 1. [0.91] Redis cache-aside with TTL decay (learned 2026-02-14, used 8x)
# 2. [0.84] Memoization for pure function hot paths (learned 2026-01-20, used 12x)
# 3. [0.73] CDN cache invalidation on deploy (learned 2026-03-01, used 3x)

# Import patterns from a similar project
/autonexus:reason --import /projects/api-gateway/reasoning-bank.json

# Output:
# Imported 67 patterns from api-gateway
# Merged 12 overlapping patterns (kept higher effectiveness)
# Skipped 5 project-specific patterns
# New total: 192 patterns
```

## Obsidian Integration

### What Gets Read/Written

| What | Obsidian Path | When |
|------|---------------|------|
| ReasoningBank data | `Projects/{ProjectName}/ReasoningBank/` | All operations |
| Pattern entries | `Projects/{ProjectName}/ReasoningBank/patterns/{category}/{id}.md` | On add/prune |
| Stats snapshot | `Projects/{ProjectName}/ReasoningBank/stats.md` | After --stats |
| Import/export log | `Projects/{ProjectName}/ReasoningBank/transfers.md` | After --import/--export |

### What Gets Read
- Pattern files when searching, analyzing, or pruning
- Stats history for trend analysis in `--meta-analysis`

## Without Obsidian

- `--stats` and `--search` work using in-memory ReasoningBank from the current session
- `--export` and `--import` work with JSON files on disk (no Obsidian needed)
- `--prune` and `--meta-analysis` require Obsidian for persistent pattern access
- Patterns learned during a session are not persisted without Obsidian

## Tips

- **Run `--stats` after every few sessions** to track learning progress and identify which categories are growing strongest.
- **Prune regularly** — patterns with effectiveness below 0.3 are likely noise. Run `--prune --threshold 0.3` monthly.
- **Import patterns from similar projects** — if you have a backend API project, import its patterns into a new backend project to bootstrap learning.
- **Use `--meta-analysis` quarterly** to find cross-cutting insights: which strategies generalize well, where the bank has blind spots.
- **`--top 10` before starting a session** gives you a quick view of the most proven strategies the system has learned.
- **Export before major refactors** — if you're restructuring the project, export the bank first as a backup.
