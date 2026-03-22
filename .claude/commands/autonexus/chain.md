---
name: autonexus:chain
description: Workflow composition — chain multiple autonexus commands into a sequential pipeline with conditions, built-in templates, and Obsidian-persisted run logs. Define pipelines inline or load from Obsidian.
argument-hint: "[step1 → step2 → ...] [--pipeline <name>] [--dry-run] [--halt-on-error] [--resume <run-id>] [--save <name>] [--list]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--pipeline <name>` or `Pipeline:` — load a named pipeline from Obsidian (`Projects/{ProjectName}/Pipelines/{name}.md`)
- `--dry-run` — validate the pipeline and show what would run, but don't execute
- `--halt-on-error` — stop the pipeline if any step fails (default: continue with `on-success` conditions)
- `--resume <run-id>` or `Resume:` — resume an interrupted pipeline run from last completed step
- `--save <name>` or `Save:` — after executing, save the pipeline definition to Obsidian for reuse
- `--list` — list all saved pipelines from Obsidian and exit
- `--template <name>` or `Template:` — use a built-in pipeline template

Remaining unmatched text = inline pipeline definition. Parse steps separated by `→`, `->`, or newlines with `1.`/`2.` numbering.

## Execution

1. Read the chain workflow: `.claude/skills/autonexus/references/chain-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. Execute Obsidian connectivity check from obsidian-integration.md (SET OBSIDIAN_AVAILABLE)
4. If `--list` → query Obsidian for saved pipelines, print table, STOP
5. If `--resume` → load pipeline run log from Obsidian, determine resume point, continue from there
6. If `--pipeline` → load pipeline definition from Obsidian
7. If `--template` → load built-in template from chain-workflow.md
8. If inline steps → parse the step definitions
9. If no steps resolved → use `AskUserQuestion` with batched questions per chain-workflow.md
10. If `--dry-run` → validate all steps, print execution plan, STOP
11. Execute the pipeline:
    - Phase 0: Initialize pipeline run (session ID, run log in Obsidian)
    - Phase 1: For each step — validate → execute → capture result → log → check conditions
    - Phase 2: Pipeline summary (all step results, total duration, artifacts produced)
    - Phase 3: Write run log to Obsidian + optional save
12. If `--save` → persist pipeline definition to Obsidian for future reuse

Stream all output live — never run in background.
