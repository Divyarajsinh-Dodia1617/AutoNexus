---
name: autonexus:optimize
description: Token optimization dashboard — view cost tracking, routing efficiency, cache hit rates, and optimization recommendations.
argument-hint: "[--dashboard] [--routing-log] [--cache-stats] [--recommendations] [--reset] [--session <id>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--dashboard` or no flags — show full optimization dashboard
- `--routing-log` — show three-tier routing decisions and savings
- `--cache-stats` — show result cache hit rates and efficiency
- `--recommendations` — get optimization recommendations based on usage patterns
- `--reset` — reset optimization counters for new tracking period
- `--session <id>` or `Session:` — show optimization data for a specific session
- `--compare <id1> <id2>` — compare optimization metrics between two sessions

## Execution

1. Read token optimization: `.claude/skills/autonexus/references/token-optimization.md`
2. Read three-tier routing: `.claude/skills/autonexus/references/three-tier-routing.md`
3. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
4. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
5. Route based on flags:
   - `--dashboard` or no flags → Display full dashboard:
     ```
     ╔══════════════════════════════════════╗
     ║   AutoNexus Optimization Dashboard   ║
     ╠══════════════════════════════════════╣
     ║ Sessions Tracked:  {N}              ║
     ║ Total Iterations:  {N}              ║
     ║ Tokens Used:       {N}              ║
     ║ Estimated Savings: {N}% ({tokens})  ║
     ╠══════════════════════════════════════╣
     ║ Tier 1 (Direct):   {N} tasks ({%}) ║
     ║ Tier 2 (Sonnet):   {N} tasks ({%}) ║
     ║ Tier 3 (Opus):     {N} tasks ({%}) ║
     ╠══════════════════════════════════════╣
     ║ Cache Hit Rate:    {%}              ║
     ║ Pattern Reuse:     {%}              ║
     ╚══════════════════════════════════════╝
     ```
   - `--routing-log` → Query Obsidian `Projects/{ProjectName}/Optimization/routing-log.md`, display:
     | Task | Complexity Score | Tier Assigned | Tokens | Savings |
   - `--cache-stats` → Display cache hit rates by category (verify, Obsidian, pattern)
   - `--recommendations` → Analyze patterns and suggest:
     - Tasks being over-routed (opus for simple tasks)
     - Tasks being under-routed (sonnet failing, needing opus)
     - Cache opportunities being missed
     - Batch operations that could be combined
   - `--reset` → Confirm with user, then clear optimization counters
   - `--compare` → Side-by-side comparison of two sessions
6. Print formatted results

Stream all output live — never run in background.
