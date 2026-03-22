---
name: autonexus:scenario
description: Scenario-driven use case generator — explores situations, edge cases, and derivative scenarios from a seed scenario using autonomous iteration with Obsidian persistence.
argument-hint: "[scenario description] [--scope <glob>] [--depth shallow|standard|deep] [--domain <type>] [--format <type>] [--focus <area>] [--iterations N]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST)

Extract these from $ARGUMENTS — the user may provide extensive context alongside flags. Ignore prose and extract ONLY flags/config:

- `--domain <type>` or `Domain:` — software, product, business, security, marketing
- `--depth <level>` or `Depth:` — shallow (10 iterations), standard (25), deep (50+)
- `--scope <glob>` or `Scope:` — file globs
- `--format <type>` — use-cases, user-stories, test-scenarios, threat-scenarios
- `--focus <area>` — edge-cases, failures, security, scale
- `Iterations:` or `--iterations N` — integer for bounded mode (CRITICAL: run exactly N iterations then stop). Overrides depth preset.
- `Scenario:` — text after "Scenario:" keyword

All remaining text not matching flags is the scenario description.

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`. Track `current_iteration`. After N, STOP.

## Execution

1. Read the scenario workflow: `.claude/skills/autonexus/references/scenario-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. If scenario or domain is missing — use `AskUserQuestion` with adaptive questions per scenario-workflow.md
4. Execute Phase 0 from obsidian-integration.md (session initialization, connectivity check, queue drain)
5. Execute the 7-phase scenario loop:
   - Phase 1: Seed (+ Obsidian dedup query for past explorations)
   - Phase 2: Decompose
   - Phase 3: Generate
   - Phase 4: Classify
   - Phase 5: Expand
   - Phase 6: Log (+ write high-severity scenarios to Obsidian Decisions/)
   - Phase 7: Repeat
6. If bounded: after each iteration, check `current_iteration < max_iterations`. If not, STOP and print summary.
7. On session end: write exploration summary with dimension coverage map to Obsidian (per scenario-workflow.md session end protocol).

Stream all output live — never run in background.
