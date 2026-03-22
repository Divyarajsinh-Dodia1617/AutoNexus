---
name: autonexus:agile
description: Full Agile sprint lifecycle with autonomous agent team. Orchestrates specialized agents through Obsidian-backed sprints with planning, execution, review, and retrospective ceremonies.
argument-hint: "[sprint goal] [--team auto|lean|standard|full] [--stories 'S1,S2,...'] [--sprint N] [--phase planning|review|retro] [--resume] [--parallel] [--dry-run]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--team <config>` or `Team:` — auto (default), lean, standard, full
- `--stories "S1,S2,..."` or `Stories:` — specific story IDs for this sprint
- `--sprint <N>` or `Sprint:` — override sprint number
- `--phase <name>` or `Phase:` — run specific phase only (planning, review, retro)
- `--resume` — resume interrupted sprint from Obsidian state
- `--parallel` — enable parallel story execution via worktrees
- `--dry-run` — plan sprint without executing

Remaining unmatched text = sprint goal description.

## Execution

1. Read the agile workflow: `.claude/skills/autonexus/references/agile-workflow.md`
2. Read the ceremonies protocol: `.claude/skills/autonexus/references/agile-ceremonies.md`
3. Read the quality gates: `.claude/skills/autonexus/references/agile-gates.md`
4. Read the agent roster: `.claude/skills/autonexus/references/agents/index.md`
5. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
6. If `--resume` → Read sprint board from Obsidian, determine resume point, continue from there
7. If `--phase` set → execute only that ceremony phase
8. If sprint goal and team are provided → proceed directly to Phase 0
9. If any context missing → use `AskUserQuestion` with batched questions per agile-workflow.md
10. Execute the 5-phase agile workflow:
    - Phase 0: Sprint Setup (Obsidian check, load roster, load backlog)
    - Phase 1: Sprint Planning (PM prioritizes, Architect assesses, Planning Poker, EM assigns, CTO reviews)
    - Phase 2: Sprint Execution (per-story: Design → Implement → Review → QA → Accept)
    - Phase 3: Sprint Review (PM acceptance, CEO business assessment)
    - Phase 4: Retrospective (metrics, reflections, action items)
    - Phase 5: Next Sprint (backlog update, loop or end)

IMPORTANT: The orchestrator is a PURE SCHEDULER. ALL work is delegated to subagent spawns (Agent tool). Each agent runs in its own isolated context window. Only brief return summaries enter the main context. All agent communication happens through Obsidian notes.
