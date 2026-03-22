---
name: autonexus:debug
description: Autonomous bug-hunting loop — scientific method + Obsidian knowledge backbone. Finds ALL bugs, not just one. Each finding becomes a searchable Obsidian note.
argument-hint: "[--fix] [--scope <glob>] [--symptom <text>] [--severity <level>] [--technique <name>] [--iterations N]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--fix` — if present, auto-switch to /autonexus:fix after finding bugs
- `--scope <glob>` or `Scope:` — file globs to investigate
- `--symptom "<text>"` or `Symptom:` — description of what's broken
- `--severity <level>` — minimum severity to report (critical/high/medium/low)
- `--technique <name>` — force a specific investigation technique (binary-search, differential, minimal-reproduction, trace, pattern-search, working-backwards, rubber-duck)
- `Iterations:` or `--iterations N` — integer for bounded mode (CRITICAL: run exactly N iterations then stop)

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`. Track `current_iteration` starting at 0. After iteration N, print final summary and STOP.

All remaining text in $ARGUMENTS is additional context — use it to understand the problem but do not treat it as flags.

## Execution

1. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
2. Read the debug workflow: `.claude/skills/autonexus/references/debug-workflow.md`
3. Execute Phase 0 (Session Initialization) from obsidian-integration.md — connectivity check, queue drain, session ID
4. If scope or symptom is missing — use `AskUserQuestion` with batched questions per debug-workflow.md
5. Execute the 7-phase debug loop (Gather -> Recon -> Hypothesize -> Test -> Classify -> Log -> Repeat)
6. Phase 2 (Recon): Query Obsidian for past debug sessions and predict warnings per debug-workflow.md
7. Phase 6 (Log): Write iteration notes + bug Decision notes to Obsidian per debug-workflow.md
8. If bounded: after each iteration, check `current_iteration < max_iterations`. If not, STOP and print summary.
9. Session end: Write debug summary to Obsidian, drain sync queue.

Stream all output live — never run in background.
