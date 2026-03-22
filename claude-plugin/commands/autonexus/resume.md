---
name: autonexus:resume
description: Resume a previous AutoNexus session — reloads goal, scope, metric config from a past session and continues iterating from current codebase state. Links new session to the resumed one in Obsidian.
argument-hint: "[--session <session-id>] [--last]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--last` — if present, skip session picker and resume the most recent session
- `--session <session-id>` or `Session:` — resume a specific session by its ID
- `Iterations:` or `--iterations N` — integer for bounded mode (CRITICAL: run exactly N iterations then stop)

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`. Track `current_iteration` starting at 0. After iteration N, print final summary and STOP.

All remaining text in $ARGUMENTS is additional context — use it to understand intent but do not treat it as flags.

## Execution

1. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
2. Read the resume workflow: `.claude/skills/autonexus/references/resume-workflow.md`
3. Read the autonomous loop protocol: `.claude/skills/autonexus/references/autonomous-loop-protocol.md`
4. Execute Phase 0 (Session Initialization) from obsidian-integration.md — connectivity check, queue drain, session ID
5. Execute Step 1 (Discover Sessions) from resume-workflow.md — query Obsidian or fall back to local TSV + git
6. Execute Step 2 (Present Options) from resume-workflow.md — use `--last`, `--session`, or interactive picker
7. Execute Step 3 (Load Configuration) from resume-workflow.md — extract goal, scope, verify, guard, direction
8. Execute Step 4 (Verify Current State) from resume-workflow.md — run verify command, establish new baseline
9. Execute Step 5 (Resume the Autonomous Loop) — enter the standard loop from autonomous-loop-protocol.md
10. Execute Step 6 (Link Sessions) from resume-workflow.md — write bidirectional links in Obsidian
11. If bounded: after each iteration, check `current_iteration < max_iterations`. If not, STOP and print summary.
12. Session end: Write session summary (with `resumed_from` field) to Obsidian, drain sync queue.

Stream all output live — never run in background.
