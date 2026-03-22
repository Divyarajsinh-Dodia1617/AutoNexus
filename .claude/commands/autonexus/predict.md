---
name: autonexus:predict
description: Multi-persona swarm prediction with persona notes and debate transcripts written to Obsidian.
argument-hint: "[goal/focus] [--scope <glob>] [--depth shallow|standard|deep] [--personas N] [--rounds N] [--chain <tools>] [--adversarial]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--scope <glob>` or `Scope:` — file globs to analyze
- `--goal <text>` or `Goal:` — focus area for personas
- `--depth <level>` or `Depth:` — shallow (3 personas, 1 round), standard (5, 2), deep (8, 3)
- `--chain <targets>` or `Chain:` — comma-separated, no spaces (debug, security, fix, ship, scenario)
- `--personas <N>` or `Personas:` — override persona count (3-8)
- `--rounds <N>` or `Rounds:` — override debate rounds (1-3)
- `--adversarial` — use adversarial persona set
- `--budget <N>` or `Budget:` — max total findings (default: 40)
- `--fail-on <severity>` — exit non-zero for CI/CD gating
- `--incremental` — reuse existing knowledge files
- `Iterations:` or `--iterations N` — bounded mode

Remaining unmatched text = goal description.

If `Iterations: N` or `--iterations N` is found, set `max_iterations = N`.

## Execution

1. Read the predict workflow: `.claude/skills/autonexus/references/predict-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. If scope, goal, and depth are all extracted — proceed directly to Phase 1
4. If any are missing — use `AskUserQuestion` with adaptive batched questions per predict-workflow.md
5. Execute the 8-phase predict workflow:
   - Phase 1: Setup (config validation)
   - Phase 2: Reconnaissance (check Obsidian cache → build knowledge files → update Obsidian Knowledge Center)
   - Phase 3: Persona Generation
   - Phase 4: Independent Analysis
   - Phase 5: Debate (1-3 rounds)
   - Phase 6: Consensus (voting + anti-herd check)
   - Phase 7: Report (local files + Obsidian persona notes + debate note)
   - Phase 8: Handoff (optional chain)
6. If bounded: check after each iteration.
7. If `--chain` set: hand off to each chained command sequentially.

IMPORTANT: Start executing immediately. Stream all output live. Each persona note uses retry-then-queue for Obsidian writes.
