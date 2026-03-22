---
name: autonexus:canvas
description: On-demand Obsidian Canvas generation — architecture diagrams, sprint boards, dependency graphs, session timelines, knowledge maps, data flows, and pipeline visualizations from project data.
argument-hint: "[--type architecture|sprint|dependency|timeline|knowledge|flow|pipeline] [--scope <glob>] [--sprint <N>] [--session <id>] [--pipeline <name>] [--output <path>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--type <type>` or `Type:` — canvas type: architecture, sprint, dependency, timeline, knowledge, flow, pipeline
- `--scope <glob>` or `Scope:` — file scope for architecture/dependency/flow canvases
- `--sprint <N>` or `Sprint:` — sprint number for sprint board canvas
- `--session <id>` or `Session:` — session ID for timeline canvas
- `--pipeline <name>` or `Pipeline:` — pipeline name for pipeline canvas
- `--output <path>` or `Output:` — custom Obsidian path for the canvas file
- `--open` — if set, print a link to the canvas for easy Obsidian navigation

Remaining unmatched text = additional context for the canvas.

## Execution

1. Read the canvas workflow: `.claude/skills/autonexus/references/canvas-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. Execute Obsidian connectivity check from obsidian-integration.md (SET OBSIDIAN_AVAILABLE)
4. If OBSIDIAN_AVAILABLE = false → STOP with: "Canvas generation requires Obsidian. Run /autonexus:setup to configure."
5. If `--type` is missing → use `AskUserQuestion` with canvas type picker per canvas-workflow.md
6. Execute the canvas generation workflow:
   - Phase 1: Gather data (source depends on canvas type — codebase, Obsidian, git)
   - Phase 2: Calculate layout (node positions based on canvas type's layout algorithm)
   - Phase 3: Generate nodes array (with colors, sizes, labels)
   - Phase 4: Generate edges array (with labels, directions)
   - Phase 5: Serialize to Obsidian Canvas JSON format
   - Phase 6: Write canvas file via obsidian_update_note
7. Print canvas path and summary of nodes/edges generated

Stream all output live — never run in background.
