---
name: autonexus:review
description: Post-session analysis — identifies patterns in what worked and failed, finds turning points and diminishing returns, writes Lessons Learned to Obsidian, optionally generates Cross-Project insights and Session Flow Canvas.
argument-hint: "[--last] [--session <id>] [--cross-project]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--last` — if present, review the most recent session (skip interactive picker)
- `--session <session-id>` or `Session:` — review a specific session by its ID
- `--cross-project` — if present, also write a Cross-Project note for generalizable patterns

All remaining text in $ARGUMENTS is additional context — use it to understand intent but do not treat it as flags.

## Execution

1. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
2. Read the review workflow: `.claude/skills/autonexus/references/review-workflow.md`
3. Execute Obsidian connectivity check from obsidian-integration.md (SET OBSIDIAN_AVAILABLE)
4. Execute Step 1 (Select Session) from review-workflow.md — use `--last`, `--session`, or interactive picker via `AskUserQuestion`
5. Execute Step 2 (Load All Iteration Notes) from review-workflow.md — read all iteration notes for the selected session
6. Execute Step 3 (Pattern Analysis) from review-workflow.md — group by outcome, file impact, category, turning point, trajectory
7. Execute Step 4 (Generate Insights) from review-workflow.md — synthesize patterns into actionable insights, print to terminal
8. Execute Step 5 (Write Lessons Learned Note) from review-workflow.md — write to Obsidian `Decisions/{date}-review-{session-id}.md`
9. If `--cross-project` flag: Execute Step 6 (Cross-Project Note) from review-workflow.md — write generalizable patterns to `Cross-Project/`
10. Execute Step 7 (Generate Session Flow Canvas) from review-workflow.md — visual flow diagram of the session

Stream all output live — never run in background.
