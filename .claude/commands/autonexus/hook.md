---
name: autonexus:hook
description: Event-driven triggers — define rules that automatically fire actions when events occur during autonexus workflows. Manage hooks via add/list/enable/disable/remove/history sub-commands.
argument-hint: "[--add] [--list] [--enable <name>] [--disable <name>] [--remove <name>] [--history] [--dry-run <event>] [--template <name>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--add` — interactive hook creation wizard
- `--list` — show all hooks and their enabled/disabled status
- `--enable <name>` or `Enable:` — enable a disabled hook
- `--disable <name>` or `Disable:` — disable a hook without deleting it
- `--remove <name>` or `Remove:` — permanently delete a hook
- `--history` — show hook execution history across sessions
- `--dry-run <event>` or `Dry-run:` — simulate which hooks would fire for a given event type
- `--template <name>` or `Template:` — create a hook from a built-in template

If no flags → default to `--list`.

Remaining unmatched text = additional context.

## Execution

1. Read the hook workflow: `.claude/skills/autonexus/references/hook-workflow.md`
2. Read the Obsidian integration layer: `.claude/skills/autonexus/references/obsidian-integration.md`
3. Execute Obsidian connectivity check from obsidian-integration.md (SET OBSIDIAN_AVAILABLE)
4. If `--list` → query Obsidian for hooks at `Projects/{ProjectName}/Hooks/`, print table, STOP
5. If `--add` → run interactive hook creation wizard per hook-workflow.md
6. If `--template` → load built-in template, create hook in Obsidian, STOP
7. If `--enable` / `--disable` → update hook frontmatter in Obsidian, STOP
8. If `--remove` → delete hook note from Obsidian (with confirmation), STOP
9. If `--history` → search Obsidian for hook execution logs, print table, STOP
10. If `--dry-run <event>` → load all enabled hooks, evaluate conditions against the event, print which would fire and what actions they would take, STOP

For hook management operations, Obsidian is REQUIRED. If unavailable, print: "Hook management requires Obsidian. Run /autonexus:setup to configure."

Hook EVALUATION during loops/workflows works in local-only mode too — hooks can be cached in `autonexus-hooks.json` (gitignored) for sessions where Obsidian is unavailable.

Stream all output live — never run in background.
