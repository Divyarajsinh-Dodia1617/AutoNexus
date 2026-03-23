---
name: autonexus:reason
description: ReasoningBank management — view, search, prune, and analyze learned patterns. The self-learning system that makes AutoNexus smarter with every session.
argument-hint: "[--search <query>] [--stats] [--prune] [--export] [--import <project>] [--meta-analysis] [--top <N>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--search <query>` or `Search:` — search patterns by keyword/context
- `--stats` — show strategy scoring statistics and effectiveness
- `--prune` — remove low-confidence and stale patterns (confidence < 0.3 or age > 30 days without re-validation)
- `--export` — export patterns for backup/sharing to local file
- `--import <project>` or `Import:` — import patterns from another project (transfer learning, 0.5x confidence)
- `--meta-analysis` or `--meta` — run meta-learning analysis on pattern effectiveness
- `--top <N>` — show top N patterns by confidence. Default: 10
- `--history` — show pattern creation/update timeline

Remaining unmatched text = search query.

## Execution

1. Read the reasoning bank protocol: `.claude/skills/autonexus/references/reasoning-bank.md`
2. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
3. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
4. If Obsidian unavailable → STOP with message: "ReasoningBank requires Obsidian for pattern storage. Run /autonexus:setup to configure."
5. Route based on flags:
   - `--stats` → Query `Projects/{ProjectName}/ReasoningBank/Strategies/`, compute and display:
     | Category | Attempts | Keeps | Keep Rate | Avg Delta | Confidence |
   - `--search` → Query `Projects/{ProjectName}/ReasoningBank/Patterns/` with search term, display matches with confidence scores
   - `--prune` → Find patterns with confidence < 0.3 or stale, show list, use `AskUserQuestion` to confirm deletion
   - `--export` → Read all patterns, format as portable markdown, write to `autonexus-patterns-export.md`
   - `--import` → Read patterns from source project, apply 0.5x confidence reduction, store as transfer patterns
   - `--meta-analysis` → Spawn meta-learner agent to analyze pattern effectiveness across sessions
   - `--top` → Display top N patterns sorted by confidence with context tags
   - `--history` → Show chronological pattern creation/update timeline
   - No flags → Display overview: total patterns, top 5 strategies, recent additions, health metrics (staleness, coverage)
6. Print formatted results

Stream all output live — never run in background.
