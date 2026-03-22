---
name: autonexus:refine-backlog
description: Interactive backlog grooming with agent assistance. Create, refine, reprioritize, split, or archive stories in the Obsidian-backed product backlog.
argument-hint: "[--new 'story title'] [--story STORY-id] [--epic epic-slug] [--review] [--archive STORY-id]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--new "title"` or `New:` — quick-add a story with this title
- `--story <id>` or `Story:` — refine a specific story
- `--epic <slug>` or `Epic:` — scope to an epic
- `--review` — review full backlog status
- `--archive <id>` — archive a specific story

Remaining unmatched text = story description or refinement guidance.

## Execution

1. Read the refine-backlog workflow: `.claude/skills/autonexus/references/refine-backlog-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. If `--new` provided → go directly to "Add New Stories" phase with title
4. If `--story` provided → go directly to "Refine Existing" phase for that story
5. If `--review` → go to "Review Full Backlog" phase
6. If `--archive` → go to "Archive Stories" phase
7. If no arguments → use `AskUserQuestion` per refine-backlog-workflow.md
8. Execute the 5-phase workflow:
   - Phase 1: Load current backlog from Obsidian
   - Phase 2: Determine user action
   - Phase 3: Execute with agent assistance (PM, Architect, UX as needed)
   - Phase 4: Validate testability (all stories must have ACs + test contracts)
   - Phase 5: Persist to Obsidian

IMPORTANT: All stories must be testable before being marked "ready" for sprint. Each story must have ≥3 acceptance criteria, a verify_command, and a test_file path. Agent spawns handle the structuring work — the user provides intent, agents structure it into proper stories.
