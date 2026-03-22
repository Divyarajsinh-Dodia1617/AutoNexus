# /autonexus:hook — Event-Driven Triggers

Define rules that automatically fire actions when events occur during autonexus workflows. Hooks make the system reactive — when crashes pile up, a hook can auto-launch debug. When a critical security finding appears, a hook can auto-create a backlog story.

## Syntax

```
# List all hooks (default)
/autonexus:hook

# Interactive hook creation
/autonexus:hook --add

# Create from built-in template
/autonexus:hook --template auto-debug-on-crashes

# Enable/disable a hook
/autonexus:hook --enable auto-debug-on-crashes
/autonexus:hook --disable pause-on-low-keep-rate

# Remove a hook
/autonexus:hook --remove old-hook

# View execution history
/autonexus:hook --history

# Test which hooks would fire for an event
/autonexus:hook --dry-run consecutive-crashes
```

## What It Does

**Management mode** (when you invoke `/autonexus:hook`):
- Create, list, enable, disable, remove, and inspect hooks
- View execution history across sessions
- Dry-run events to test which hooks would fire

**Evaluation mode** (automatic, during other workflows):
- Hooks are loaded at session start and cached locally
- After each iteration, session end, finding, story completion, etc. — relevant hooks are evaluated
- If conditions are met, actions fire automatically
- Hook actions never block the loop

## Event Types

| Event | When It Fires | Key Variables |
|-------|--------------|---------------|
| `iteration-complete` | After any loop iteration | `$STATUS`, `$METRIC`, `$DELTA` |
| `consecutive-discards` | After N discards in a row | `$COUNT`, `$SCOPE` |
| `consecutive-crashes` | After N crashes in a row | `$COUNT`, `$SCOPE` |
| `keep-rate-drop` | Keep rate falls below threshold | `$KEEP_RATE` |
| `metric-threshold` | Metric crosses a value | `$METRIC`, `$DIRECTION` |
| `goal-achieved` | Metric reaches the goal | `$METRIC`, `$GOAL` |
| `session-end` | Any session ends | `$KEEPS`, `$DISCARDS`, `$KEEP_RATE` |
| `finding-created` | Security/debug creates finding | `$SEVERITY`, `$TITLE`, `$FILE` |
| `story-complete` | Agile story reaches Done | `$STORY_ID`, `$SPRINT` |
| `gate-failed` | Quality gate fails | `$GATE_NAME`, `$FAILURE_COUNT` |
| `sprint-end` | Sprint completes | `$SPRINT`, `$VELOCITY` |
| `pipeline-step-failed` | Chain pipeline step fails | `$PIPELINE`, `$COMMAND` |

## Action Types

| Action | What Happens |
|--------|-------------|
| `run-command` | Execute an autonexus command (e.g., `/autonexus:debug`) |
| `create-note` | Write a note to Obsidian |
| `create-story` | Add a story to the backlog |
| `tag-note` | Add tags to an Obsidian note |
| `alert` | Print a highlighted message to the terminal |
| `pause` | Pause the loop and ask the user what to do |
| `log` | Append extra info to the hook execution log |

## Built-in Templates

| Template | Event | Action | Use When |
|----------|-------|--------|----------|
| `auto-debug-on-crashes` | 3 consecutive crashes | Run /autonexus:debug | Systematic issues need investigation |
| `backlog-from-findings` | CRITICAL/HIGH finding | Create backlog story | Findings should become actionable items |
| `pause-on-low-keep-rate` | Keep rate < 20% | Pause + suggest alternatives | Loop is spinning without progress |
| `celebrate-goal` | Goal achieved | Alert message | Positive reinforcement |
| `auto-fix-on-guard-fail` | Guard fails on iteration | Run /autonexus:fix | Regressions need automated repair |
| `auto-review-on-session-end` | Session ends with 5+ keeps | Run /autonexus:review | Auto-analyze productive sessions |
| `escalate-gate-failures` | Gate fails 3+ times | Alert message | Manual intervention needed |

## Condition Syntax

```
# Comparisons
count >= 3
keep_rate < 20
metric > 90

# Equality
severity == "CRITICAL"
status == "crash"

# Set membership
severity in ["CRITICAL", "HIGH"]

# Combination
count >= 3 AND keep_rate < 30

# Always fire
always
```

## Flags

| Flag | Purpose |
|------|---------|
| `--add` | Interactive hook creation wizard |
| `--list` | Show all hooks and status (default) |
| `--enable <name>` | Enable a disabled hook |
| `--disable <name>` | Disable without deleting |
| `--remove <name>` | Permanently delete a hook |
| `--history` | Show hook execution history |
| `--dry-run <event>` | Test which hooks fire for an event |
| `--template <name>` | Create from built-in template |

## Obsidian Integration

### What Gets Read
- **Hook definitions:** `Projects/{ProjectName}/Hooks/{name}.md`
- **Execution history:** From each hook's history table

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Hook definition | `Projects/{ProjectName}/Hooks/{name}.md` | On `--add` or `--template` |
| Frontmatter updates | `last_triggered`, `trigger_count` | After each hook fires |
| History table row | Appended to the hook note | After each hook fires |
| Backlog stories | `Projects/{ProjectName}/Backlog/stories/STORY-{id}.md` | When `create-story` action fires |

### Local Cache

Hooks are cached to `autonexus-hooks.json` (gitignored) at session start. This enables hook evaluation even when Obsidian is unavailable — hooks still fire based on the cached definitions.

## Without Obsidian

- **Hook management** (add/list/enable/disable/remove) requires Obsidian
- **Hook evaluation** works offline using the local `autonexus-hooks.json` cache
- Actions that target Obsidian (create-note, tag-note, create-story) become no-ops when offline
- `run-command` and `alert` actions work regardless of Obsidian status

## Safety

- **Non-blocking:** Hook actions never block the iteration loop. If a hook action fails, it's logged and skipped.
- **Circular protection:** Each hook can fire at most once per iteration. A `run-command` action that triggers the same event won't re-fire the same hook.
- **Priority ordering:** Hooks fire in priority order (lower number = fires first). Default priority is 50.

## Tips

- **Start with templates** — `--template auto-debug-on-crashes` and `--template backlog-from-findings` are the most immediately useful.
- **Use `--dry-run`** to test your hook logic before enabling it in production sessions.
- **Combine hooks with chains** — hooks handle reactive automation (when X happens), chains handle proactive pipelines (do X then Y).
- **Keep conditions specific** — `severity == "CRITICAL"` is better than `always` for finding hooks. You don't want noise.
- **Check `--history`** periodically to see which hooks are actually firing and whether they're useful.
- **Disable before removing** — `--disable` lets you test whether you miss the hook before permanently deleting it.
