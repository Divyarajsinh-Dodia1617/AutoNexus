---
name: autonexus:fix
description: Autonomous fix loop — iteratively repairs errors until zero remain. One fix per iteration, atomic, auto-reverted on failure. Each fix logged to Obsidian.
argument-hint: "[--target <cmd>] [--guard <cmd>] [--scope <glob>] [--category <type>] [--skip-lint] [--from-debug] [--iterations N]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--target <cmd>` or `Target:` — explicit verify command
- `--guard <cmd>` or `Guard:` — safety command that must always pass
- `--scope <glob>` or `Scope:` — file globs to fix
- `--category <type>` — only fix: test, type, lint, or build
- `--skip-lint` — skip lint fixes, focus on tests/types/build only
- `--from-debug` — read findings from latest debug session
- `Iterations:` or `--iterations N` — integer for bounded mode (CRITICAL: run exactly N iterations then stop)

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`. Track `current_iteration` starting at 0. After iteration N, print final summary and STOP. Also stops when error count = 0.

All remaining text in $ARGUMENTS is additional context — use it to understand the problem but do not treat it as flags.

## Execution

1. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
2. Read the fix workflow: `.claude/skills/autonexus/references/fix-workflow.md`
3. Execute Phase 0 (Session Initialization) from obsidian-integration.md — connectivity check, queue drain, session ID
4. If target and scope are missing — use `AskUserQuestion` with batched questions per fix-workflow.md
5. Execute the 8-phase fix loop: ONE fix per iteration, never suppress errors, auto-revert on regression
6. Phase 1 (Detect): Query Obsidian for recurring issues per fix-workflow.md
7. Phase 3 (Fix): Query Obsidian for failed approaches + predict warnings before modifying files
8. Phase 8 (Log): Write concise iteration notes to Obsidian (5-8 lines per template)
9. If bounded: after each iteration, check `current_iteration < max_iterations`. If not, STOP and print summary.
10. When error count hits zero: Write "Clean State" Decision note to Obsidian per fix-workflow.md
11. Session end: Write fix summary to Obsidian, drain sync queue.

Stream all output live — never run in background.
