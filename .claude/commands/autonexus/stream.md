---
name: autonexus:stream
description: Stream chaining — pipe agent outputs directly into next agent's context without intermediate storage. Fast agent-to-agent handoffs.
argument-hint: "[agent1 → agent2 → ...] [--pipeline <name>] [--dry-run] [--verbose]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--pipeline <name>` or `Pipeline:` — use predefined stream pipeline (analysis|refactor|debug|optimize|sparc)
- `--dry-run` — show the stream chain without executing
- `--verbose` — show full stream context at each handoff (default: summary only)
- `--task <description>` or `Task:` — task description passed to first agent in chain
- `--scope <glob>` or `Scope:` — file scope for the stream chain

Remaining unmatched text = inline stream chain definition. Parse agents separated by `→`, `->`, or `--stream→`.

## Execution

1. Read stream chaining: `.claude/skills/autonexus/references/stream-chaining.md`
2. Read agent roster: `.claude/skills/autonexus/references/agents/index.md`
3. Resolve chain:
   - If `--pipeline` → load predefined pipeline:
     - **analysis:** scout-explorer → security-analyst → sr-qa-eng
     - **refactor:** architect → sr-fullstack-eng → code-review-swarm → sr-qa-eng
     - **debug:** scout-explorer → sr-backend-eng → sr-qa-eng → qa-manager
     - **optimize:** perf-monitor → topology-optimizer → sr-backend-eng → benchmark-runner
     - **sparc:** sparc-spec → sparc-pseudo → sparc-arch → sparc-refine → sparc-complete
   - If inline chain → parse agent sequence, validate all agents exist in roster
   - If no chain → use `AskUserQuestion`:
     - Q1 (Pipeline): "Which stream pipeline?" with [Analysis, Refactor, Debug, Optimize, SPARC, Custom]
     - Q2 (Task): "What task for the stream?" with custom input
4. If `--dry-run` → display execution plan with agent sequence, models, estimated handoffs → STOP
5. Execute stream chain:
   - **Step 1:** Construct first agent's prompt with task + scope
   - **Step 2:** Spawn Agent 1 → capture structured return (max 500 tokens)
   - **Step 3:** Construct Agent 2's prompt with:
     - Agent 2's definition
     - Task description
     - **STREAM CONTEXT from Agent 1:** (verbatim structured output)
   - **Step 4:** Spawn Agent 2 → capture return
   - **Step 5:** Repeat, accumulating stream context from all prior agents
   - Each agent receives: its own context + ALL prior agent outputs in sequence
6. After chain completes:
   - Print final result from last agent
   - Print chain summary: agents involved, total time, artifacts produced
   - If Obsidian available: write stream log to `Projects/{ProjectName}/Streams/`

Stream all output live — never run in background.
