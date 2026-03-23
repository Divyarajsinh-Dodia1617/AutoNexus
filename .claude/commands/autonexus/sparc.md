---
name: autonexus:sparc
description: SPARC 5-phase development workflow — Specification → Pseudocode → Architecture → Refinement → Completion with agent handoffs at each phase.
argument-hint: "[feature/goal description] [--phase <S|P|A|R|C>] [--skip-to <phase>] [--parallel] [--agents <team>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--phase <S|P|A|R|C>` or `Phase:` — run only a specific phase (Specification, Pseudocode, Architecture, Refinement, Completion)
- `--skip-to <phase>` or `Skip:` — skip earlier phases (assumes they're already done in Obsidian)
- `--parallel` — enable parallel tracks where possible (P+A overlap, implementation+testing overlap)
- `--agents <team>` or `Team:` — agent team config (lean/standard/full/auto) for the Refinement phase
- `--feature <name>` or `Feature:` — feature name for Obsidian folder naming

Remaining unmatched text = feature/goal description for the SPARC workflow.

## Execution

1. Read the SPARC methodology: `.claude/skills/autonexus/references/sparc-methodology.md`
2. Read the agent roster: `.claude/skills/autonexus/references/agents/index.md`
3. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
4. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
5. If no feature/goal provided → use `AskUserQuestion` with:
   - Question 1 (Feature): "What feature or goal should SPARC build?" with options from backlog or custom
   - Question 2 (Scope): "Which files/modules are in scope?" with detected options from project structure
   - Question 3 (Phase): "Run all 5 phases or start from a specific one?" with options [All phases (Recommended), Specification only, Pseudocode only, Architecture only, Refinement only, Completion only]
6. Create SPARC folder in Obsidian: `Projects/{ProjectName}/SPARC/{feature}/`
7. Execute phases sequentially (or parallel where flagged):
   - **Phase S:** Spawn sparc-spec agent → writes Specification.md → gate: all criteria measurable
   - **Phase P:** Spawn sparc-pseudo agent → writes Pseudocode.md → gate: all requirements covered
   - **Phase A:** Spawn sparc-arch agent → writes Architecture.md → gate: CTO approval
   - **Phase R:** Run mini autoresearch loop or spawn implementation team → working code → gate: all tests pass
   - **Phase C:** Spawn sparc-complete agent → writes Completion.md → gate: full sign-off
8. After each phase: fire `sparc-phase-complete` event, write phase note to Obsidian
9. Final summary: all phases, artifacts produced, time taken, acceptance criteria status

Stream all output live — never run in background.
