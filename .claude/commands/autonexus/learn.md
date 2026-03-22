---
name: autonexus:learn
description: Autonomous codebase documentation engine — scout, learn, generate/update docs with validation-fix loop and Obsidian Knowledge Center sync.
argument-hint: "[goal/focus] [--mode init|update|check|summarize] [--scope <glob>] [--depth quick|standard|deep] [--file <name>] [--scan] [--topics <list>] [--no-fix] [--format markdown|html|json|rst] [--iterations N]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--mode <mode>` or `Mode:` — init, update, check, summarize
- `--scope <glob>` or `Scope:` — limit codebase learning to specific dirs
- `--depth <level>` or `Depth:` — quick, standard, deep
- `--file <name>` — selective update targeting one doc file
- `--scan` — force fresh scout in summarize mode
- `--topics <list>` — focus summarize on specific topics
- `--no-fix` — skip validation-fix loop
- `--format <fmt>` — output format: markdown (default), html, json, rst
- `Iterations:` or `--iterations N` — integer for bounded mode (CRITICAL: run exactly N iterations then stop)

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`. Track `current_iteration` starting at 0. After iteration N, print final summary and STOP.

All remaining text in $ARGUMENTS is additional context — use it to understand the problem but do not treat it as flags.

## Execution

1. Read the learn workflow: `.claude/skills/autonexus/references/learn-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. If scope or goal is missing — use `AskUserQuestion` with batched questions per learn-workflow.md
4. Execute Phase 0 from obsidian-integration.md (session initialization, connectivity check, queue drain)
5. Execute the learn workflow:
   - Phase 1: Scout (+ check Obsidian Knowledge Center cache)
   - Phase 2: Analyze
   - Phase 3: Map
   - Phase 4: Generate (+ sync to Obsidian Knowledge Center)
   - Phase 5: Validate
   - Phase 6: Fix (loop)
   - Phase 7: Finalize (+ update Obsidian Knowledge Center metadata)
   - Phase 8: Log
6. If bounded: after each iteration, check `current_iteration < max_iterations`. If not, STOP and print summary.

Stream all output live — never run in background.
