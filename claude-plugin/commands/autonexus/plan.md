---
name: autonexus:plan
description: Interactive wizard to build Scope, Metric, Direction & Verify from a Goal. Writes plan decision to Obsidian.
argument-hint: "[goal description]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract goal from $ARGUMENTS:

- Look for `Goal:` keyword — text after it is the goal
- If no `Goal:` keyword, treat entire $ARGUMENTS text as the goal
- If $ARGUMENTS is empty, the goal is missing — will ask via AskUserQuestion

## Execution

1. Read the plan workflow: `.claude/skills/autonexus/references/plan-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. If goal is extracted — proceed to Phase 2 (Analyze Context)
4. If goal is missing — use `AskUserQuestion` to capture it (Phase 1)
5. Execute the 7-step planning wizard:
   - Phase 1: Capture Goal
   - Phase 2: Analyze Context (+ query Obsidian Knowledge Center if available)
   - Phase 3: Define Scope
   - Phase 4: Define Metric
   - Phase 4.5: Define Guard (optional)
   - Phase 5: Define Direction
   - Phase 6: Define Verify (with mandatory dry run)
   - Phase 7: Confirm & Launch (+ write Decision note to Obsidian)

IMPORTANT: Start executing immediately. Every metric must pass a dry run before being accepted.
