---
name: autonexus:status
description: Read-only dashboard showing AutoNexus project health — session history, iteration stats, keep rate, strategy effectiveness, Knowledge Center freshness, and unresolved findings. Makes no modifications.
argument-hint: "[--project <name>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--project <name>` or `Project:` — override auto-detected project name

All remaining text in $ARGUMENTS is additional context — use it to understand intent but do not treat it as flags.

## Execution

1. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
2. Read the status workflow: `.claude/skills/autonexus/references/status-workflow.md`
3. Execute Obsidian connectivity check from obsidian-integration.md (SET OBSIDIAN_AVAILABLE)
4. Execute Step 1 (Project Detection) from status-workflow.md — resolve ProjectName
5. Execute Step 2 (Query Obsidian) from status-workflow.md — gather session, iteration, decision, prediction, and sync data
6. Execute Step 3 (Calculate Statistics) from status-workflow.md — compute keep rate, top strategies, etc.
7. Execute Step 4 (Print Dashboard) from status-workflow.md — display the dashboard to the terminal

This is a READ-ONLY command. Do NOT modify any files, git state, or Obsidian notes.

Stream all output live — never run in background.
