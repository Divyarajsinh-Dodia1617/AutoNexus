# Hook Workflow — /autonexus:hook

Event-driven trigger system for AutoNexus. Define rules in Obsidian that automatically fire actions when events occur during iteration loops, sprints, or any autonexus workflow. Hooks make the system reactive — instead of manually watching for problems, hooks detect and respond to them automatically.

**Core idea:** Hooks separate detection from response. Define "when X happens, do Y" rules once, and they fire across all future sessions. The iteration loop evaluates hooks at specific integration points — after each iteration, after crashes, at thresholds. Hooks never block the loop — actions are best-effort, just like Obsidian writes.

## Trigger

- User invokes `/autonexus:hook`
- User says "add a hook", "create a trigger", "when crashes happen run debug", "list hooks"
- User says "auto-fix on errors", "alert me when keep rate drops", "auto-create stories from findings"

## Architecture

```
/autonexus:hook (management)
  ├── --list     → Show all hooks and status
  ├── --add      → Interactive hook creation wizard
  ├── --template → Create from built-in template
  ├── --enable   → Enable a disabled hook
  ├── --disable  → Disable a hook
  ├── --remove   → Delete a hook
  ├── --history  → Show execution history
  └── --dry-run  → Simulate event → show which hooks fire

Hook Evaluation (integrated into other workflows)
  ├── After each iteration (in /autonexus loop)
  ├── At session end (in /autonexus loop)
  ├── After findings (in /autonexus:security, /autonexus:debug)
  ├── After story completion (in /autonexus:agile)
  ├── After gate failure (in /autonexus:agile)
  └── At sprint end (in /autonexus:agile)
```

---

## Event Types

| Event | Fires When | Context Variables |
|-------|-----------|-------------------|
| `iteration-complete` | After any iteration in the loop | `$STATUS`, `$METRIC`, `$DELTA`, `$ITERATION`, `$SESSION` |
| `consecutive-discards` | After N consecutive discards | `$COUNT`, `$THRESHOLD`, `$SESSION`, `$SCOPE` |
| `consecutive-crashes` | After N consecutive crashes | `$COUNT`, `$THRESHOLD`, `$SESSION`, `$SCOPE` |
| `keep-rate-drop` | Session keep rate falls below threshold | `$KEEP_RATE`, `$THRESHOLD`, `$SESSION` |
| `metric-threshold` | Metric crosses a threshold value | `$METRIC`, `$THRESHOLD`, `$DIRECTION`, `$SESSION` |
| `goal-achieved` | Metric reaches the stated goal value | `$METRIC`, `$GOAL`, `$SESSION`, `$ITERATION` |
| `session-end` | Any session ends | `$SESSION`, `$KEEPS`, `$DISCARDS`, `$CRASHES`, `$KEEP_RATE`, `$BASELINE`, `$FINAL` |
| `finding-created` | Security/debug creates a finding | `$SEVERITY`, `$FINDING_TYPE`, `$TITLE`, `$FILE`, `$SESSION` |
| `story-complete` | Agile story reaches Done | `$STORY_ID`, `$STORY_TITLE`, `$SPRINT`, `$STATUS` |
| `gate-failed` | Quality gate fails in agile | `$GATE_NAME`, `$STORY_ID`, `$SPRINT`, `$FAILURE_COUNT` |
| `sprint-end` | Sprint completes | `$SPRINT`, `$STORIES_COMPLETED`, `$STORIES_TOTAL`, `$VELOCITY` |
| `pipeline-step-failed` | A chain pipeline step fails | `$PIPELINE`, `$STEP`, `$COMMAND`, `$ERROR` |

---

## Action Types

| Action | What It Does | Required Fields |
|--------|-------------|----------------|
| `run-command` | Execute an autonexus command | `command`, `args` (with variable substitution) |
| `create-note` | Write a note to Obsidian | `path`, `content` (template with variables) |
| `create-story` | Add a story to the backlog | `title`, `description`, `priority` |
| `tag-note` | Add a tag to a note | `path`, `tags` |
| `alert` | Print highlighted message to terminal | `message` (template with variables) |
| `pause` | Pause the loop and ask user for input | `message` |
| `log` | Append to hook execution log | `message` (auto-logged always, this adds extra info) |

### Variable Substitution

All action fields support `$VARIABLE` substitution from the event context. Variables are replaced at execution time:

```
message: "Keep rate dropped to $KEEP_RATE% (threshold: $THRESHOLD%). Consider running /autonexus:predict."
command: "/autonexus:debug --scope $SCOPE"
title: "[$SEVERITY] $TITLE"
```

---

## Hook Definition Format (Obsidian)

Stored at `Projects/{ProjectName}/Hooks/{name}.md`:

```markdown
---
tags: [autonexus/hook]
name: {hook-name}
enabled: true
priority: 50
created: {ISO timestamp}
last_triggered: {ISO timestamp or null}
trigger_count: {N}
---

# Hook: {Human-readable Name}

## Description
{What this hook does and why it exists}

## Trigger
- **Event:** {event-type}
- **Condition:** {condition expression}

## Action
- **Type:** {action-type}
- **Command:** {if run-command — autonexus command with args}
- **Message:** {template with $VARIABLES}
- **Path:** {if create-note or tag-note — Obsidian path}
- **Tags:** {if tag-note — comma-separated tags}
- **Title:** {if create-story}
- **Description:** {if create-story}
- **Priority:** {if create-story — high, medium, low}

## Execution History

| Date | Event | Context | Action Taken | Result |
|------|-------|---------|-------------|--------|
```

---

## Condition Expressions

Conditions use simple comparison syntax:

```
# Threshold conditions
count >= 3
count >= 5

# Metric conditions
metric > 90
metric < 50
keep_rate < 20

# Severity conditions
severity == "CRITICAL"
severity in ["CRITICAL", "HIGH"]

# Status conditions
status == "crash"
gate_name == "QA Gate"

# Always (fire every time the event occurs)
always
```

Multiple conditions can be combined with AND:

```
severity == "CRITICAL" AND finding_type == "security"
count >= 3 AND keep_rate < 30
```

---

## Built-in Hook Templates

### auto-debug-on-crashes

```yaml
Event: consecutive-crashes
Condition: count >= 3
Action: run-command
Command: /autonexus:debug --scope $SCOPE
Message: "$COUNT consecutive crashes detected. Launching debug workflow."
Priority: 10
```

**Use when:** The iteration loop hits repeated crashes — likely a systemic issue that needs investigation.

### backlog-from-findings

```yaml
Event: finding-created
Condition: severity in ["CRITICAL", "HIGH"]
Action: create-story
Title: "[$SEVERITY] $TITLE"
Description: "Auto-created from $FINDING_TYPE finding in $FILE.\n\nFinding: $TITLE\nSeverity: $SEVERITY\nSession: $SESSION"
Priority: high
Message: "Auto-created backlog story from $SEVERITY finding: $TITLE"
Priority: 30
```

**Use when:** Security or debug findings should automatically become actionable backlog items.

### pause-on-low-keep-rate

```yaml
Event: keep-rate-drop
Condition: keep_rate < 20
Action: pause
Message: "Keep rate has dropped to $KEEP_RATE%. Current approach may be exhausted.\n\nSuggestions:\n- Run /autonexus:predict for fresh perspectives\n- Narrow the scope\n- Try a fundamentally different strategy"
Priority: 20
```

**Use when:** The loop is spinning without progress — human intervention needed.

### celebrate-goal

```yaml
Event: goal-achieved
Condition: always
Action: alert
Message: "Goal achieved! Metric reached $METRIC (goal was $GOAL) at iteration $ITERATION.\n\nSession $SESSION is complete."
Priority: 50
```

**Use when:** Positive reinforcement — know immediately when the goal is hit.

### auto-fix-on-guard-fail

```yaml
Event: iteration-complete
Condition: status == "discard" AND guard == "fail"
Action: run-command
Command: /autonexus:fix --scope $SCOPE --iterations 5
Message: "Guard failed on iteration $ITERATION. Running quick fix loop."
Priority: 40
```

**Use when:** Guard failures suggest regressions that the fix loop can handle.

### auto-review-on-session-end

```yaml
Event: session-end
Condition: keeps >= 5
Action: run-command
Command: /autonexus:review --last
Message: "Session ended with $KEEPS keeps. Running automatic review."
Priority: 60
```

**Use when:** Automatically analyze productive sessions.

### escalate-gate-failures

```yaml
Event: gate-failed
Condition: failure_count >= 3
Action: alert
Message: "$GATE_NAME has failed $FAILURE_COUNT times for story $STORY_ID in sprint $SPRINT. Consider manual intervention or story reassignment."
Priority: 15
```

**Use when:** Quality gates are repeatedly failing — may need architectural rethink.

---

## Hook Management Operations

### --add (Interactive Creation)

Use `AskUserQuestion` with batched questions:

**Batch 1 — Core definition (3 questions):**

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Event` | "What event should trigger this hook?" | "Iteration complete", "Consecutive discards", "Consecutive crashes", "Keep rate drops", "Metric threshold", "Goal achieved", "Session end", "Finding created", "Story complete", "Gate failed", "Sprint end", "Pipeline step failed" |
| 2 | `Condition` | "What condition must be met?" | Context-dependent options based on Event selection + "Custom condition" |
| 3 | `Action` | "What should happen when triggered?" | "Run an autonexus command", "Create Obsidian note", "Create backlog story", "Alert me (print message)", "Pause the loop", "Tag a note" |

**Batch 2 — Action details (1-3 questions based on action type):**

| Action Type | Questions |
|-------------|-----------|
| run-command | Command: "Which command?" (list all autonexus commands) |
| create-note | Path: "Where in Obsidian?", Content: "Note template?" |
| create-story | Title: "Story title?", Priority: "high/medium/low" |
| alert | Message: "Alert message?" |
| pause | Message: "Pause message?" |
| tag-note | Path: "Which note?", Tags: "Tags to add?" |

After collecting input, generate hook name from event + action, write to Obsidian.

### --list

```
IF OBSIDIAN_AVAILABLE:
  obsidian_list_notes({
    dirPath: "Projects/{ProjectName}/Hooks/",
    recursionDepth: 0
  })

  FOR each hook note:
    Read frontmatter for name, enabled, priority, last_triggered, trigger_count

  PRINT:
  ┌──────────────────────────┬──────────┬────────────────────┬──────┬────────┬──────────────────┐
  │ Hook                     │ Event    │ Condition          │ Prio │ Status │ Last Triggered   │
  ├──────────────────────────┼──────────┼────────────────────┼──────┼────────┼──────────────────┤
  │ auto-debug-on-crashes    │ crashes  │ count >= 3         │ 10   │ ON     │ 2026-03-20 14:30 │
  │ backlog-from-findings    │ finding  │ severity CRIT/HIGH │ 30   │ ON     │ 2026-03-22 09:15 │
  │ pause-on-low-keep-rate   │ keep-rate│ rate < 20%         │ 20   │ OFF    │ never            │
  └──────────────────────────┴──────────┴────────────────────┴──────┴────────┴──────────────────┘

  {count} hooks ({enabled_count} enabled, {disabled_count} disabled)
ELSE:
  Check autonexus-hooks.json for cached hooks
  IF exists: show cached hooks with warning "Showing cached hooks — Obsidian unavailable"
  ELSE: "No hooks found. Run /autonexus:hook --add to create one."
```

### --enable / --disable

```
obsidian_manage_frontmatter({
  filePath: "Projects/{ProjectName}/Hooks/{name}.md",
  operation: "set",
  key: "enabled",
  value: {true or false}
})

PRINT: "Hook '{name}' {enabled|disabled}."
```

### --remove

```
AskUserQuestion:
  header: "Confirm"
  question: "Remove hook '{name}'? This cannot be undone."
  options: ["Yes, remove it", "No, keep it"]

IF confirmed:
  obsidian_delete_note({
    filePath: "Projects/{ProjectName}/Hooks/{name}.md"
  })
  Also remove from autonexus-hooks.json if cached
  PRINT: "Hook '{name}' removed."
```

### --history

```
IF OBSIDIAN_AVAILABLE:
  obsidian_global_search({
    query: "autonexus/hook-execution",
    searchInPath: "Projects/{ProjectName}/Hooks/",
    contextLength: 300,
    pageSize: 50
  })

  Also read "Execution History" tables from each hook note.

  PRINT:
  ┌──────────────────────┬──────────────────────────┬──────────────────────┬───────────────────────────┬─────────┐
  │ Date                 │ Hook                     │ Event                │ Action Taken              │ Result  │
  ├──────────────────────┼──────────────────────────┼──────────────────────┼───────────────────────────┼─────────┤
  │ 2026-03-22 14:30     │ auto-debug-on-crashes    │ 3 consecutive crashes│ Ran /autonexus:debug      │ success │
  │ 2026-03-22 09:15     │ backlog-from-findings    │ CRITICAL finding     │ Created STORY-47          │ success │
  └──────────────────────┴──────────────────────────┴──────────────────────┴───────────────────────────┴─────────┘
```

### --dry-run

```
AskUserQuestion:
  header: "Event"
  question: "Which event to simulate?"
  options: [list all event types]

Load all enabled hooks.
FOR each hook where event matches:
  Evaluate condition with mock context values.
  PRINT: "Hook '{name}' WOULD fire → {action_type}: {action_details}"

IF no hooks match:
  PRINT: "No hooks would fire for event '{event}'."
```

---

## Hook Evaluation Protocol (Integration Points)

This section defines how OTHER autonexus workflows evaluate hooks. Hooks are loaded once at session start and cached for the session.

### Session Start: Load Hooks

```
FUNCTION load_hooks():
  hooks = []

  IF OBSIDIAN_AVAILABLE:
    hook_files = obsidian_list_notes({
      dirPath: "Projects/{ProjectName}/Hooks/",
      recursionDepth: 0
    })

    FOR each .md file:
      content = obsidian_read_note({ filePath: file })
      Parse frontmatter + trigger + action sections
      IF enabled == true:
        hooks.append(parsed_hook)

    # Cache locally for Obsidian-unavailable fallback
    Write hooks to autonexus-hooks.json (gitignored)

  ELIF autonexus-hooks.json exists:
    Load cached hooks from local file

  RETURN hooks sorted by priority (ascending — lower number fires first)
```

### Event Firing

```
FUNCTION fire_event(event_type, context):
  IF hooks is empty:
    RETURN  # No hooks loaded — skip evaluation

  FOR each hook WHERE hook.event == event_type:
    IF evaluate_condition(hook.condition, context):
      LOG: "Hook '{hook.name}' triggered by {event_type}"

      TRY:
        execute_action(hook.action, context)
        result = "success"
      CATCH error:
        result = "failed: {error}"
        # Hook failures NEVER block the loop

      # Update hook metadata (best-effort)
      IF OBSIDIAN_AVAILABLE:
        obsidian_manage_frontmatter({
          filePath: "Projects/{ProjectName}/Hooks/{hook.name}.md",
          operation: "set",
          key: "last_triggered",
          value: "{ISO timestamp}"
        })
        obsidian_manage_frontmatter({
          filePath: "Projects/{ProjectName}/Hooks/{hook.name}.md",
          operation: "set",
          key: "trigger_count",
          value: "{incremented}"
        })

        # Append to execution history table
        obsidian_update_note({
          targetType: "filePath",
          targetIdentifier: "Projects/{ProjectName}/Hooks/{hook.name}.md",
          content: "\n| {date} | {event_type} | {context_summary} | {action_summary} | {result} |",
          modificationType: "wholeFile",
          wholeFileMode: "append"
        })

      # Update local cache
      Update autonexus-hooks.json trigger_count and last_triggered
```

### Condition Evaluation

```
FUNCTION evaluate_condition(condition, context):
  IF condition == "always":
    RETURN true

  # Parse condition into (field, operator, value)
  # Support: ==, !=, >, <, >=, <=, in

  field_value = context[field]

  SWITCH operator:
    CASE "==": RETURN field_value == value
    CASE "!=": RETURN field_value != value
    CASE ">":  RETURN float(field_value) > float(value)
    CASE "<":  RETURN float(field_value) < float(value)
    CASE ">=": RETURN float(field_value) >= float(value)
    CASE "<=": RETURN float(field_value) <= float(value)
    CASE "in": RETURN field_value in parse_list(value)

  # AND combination
  IF condition contains " AND ":
    parts = condition.split(" AND ")
    RETURN all(evaluate_condition(part, context) for part in parts)
```

### Action Execution

```
FUNCTION execute_action(action, context):
  # Variable substitution in all string fields
  resolved_action = substitute_variables(action, context)

  SWITCH action.type:
    CASE "run-command":
      # Invoke the skill — this runs the full autonexus command
      Skill(skill="autonexus:{command}", args=resolved_action.args)

    CASE "create-note":
      IF OBSIDIAN_AVAILABLE:
        obsidian_update_note({
          targetType: "filePath",
          targetIdentifier: resolved_action.path,
          content: resolved_action.content,
          modificationType: "wholeFile",
          wholeFileMode: "overwrite",
          createIfNeeded: true
        })

    CASE "create-story":
      IF OBSIDIAN_AVAILABLE:
        # Create story in backlog following the backlog format
        obsidian_update_note({
          targetType: "filePath",
          targetIdentifier: "Projects/{ProjectName}/Backlog/stories/STORY-{next_id}.md",
          content: "{story template with resolved_action.title, description, priority}",
          modificationType: "wholeFile",
          wholeFileMode: "overwrite",
          createIfNeeded: true
        })
        # Update backlog index
        obsidian_update_note({
          targetType: "filePath",
          targetIdentifier: "Projects/{ProjectName}/Backlog/_index.md",
          content: "\n- [[stories/STORY-{next_id}|{resolved_action.title}]] — Priority: {priority} — Auto-created by hook",
          modificationType: "wholeFile",
          wholeFileMode: "append"
        })

    CASE "tag-note":
      IF OBSIDIAN_AVAILABLE:
        obsidian_manage_tags({
          filePath: resolved_action.path,
          operation: "add",
          tags: resolved_action.tags
        })

    CASE "alert":
      PRINT "╔══════════════════════════════════════════════╗"
      PRINT "║ HOOK ALERT: {hook.name}"
      PRINT "║ {resolved_action.message}"
      PRINT "╚══════════════════════════════════════════════╝"

    CASE "pause":
      PRINT "╔══════════════════════════════════════════════╗"
      PRINT "║ HOOK PAUSE: {hook.name}"
      PRINT "║ {resolved_action.message}"
      PRINT "╚══════════════════════════════════════════════╝"
      AskUserQuestion:
        header: "Hook Pause"
        question: resolved_action.message + "\n\nWhat would you like to do?"
        options: ["Continue the loop", "Change strategy", "Stop the loop"]

    CASE "log":
      # Already logged by the fire_event function
      # This adds extra info to the log entry
      PASS
```

---

## Integration Points in Other Workflows

### In /autonexus (main loop)

```
# Phase 0: Load hooks
hooks = load_hooks()

# Phase 7 (after each iteration):
fire_event("iteration-complete", {
  status: iteration_status,
  metric: current_metric,
  delta: delta,
  iteration: iteration_number,
  session: session_id,
  scope: scope,
  guard: guard_status
})

# Check consecutive discards
IF last N statuses are all "discard":
  fire_event("consecutive-discards", {
    count: N,
    threshold: N,
    session: session_id,
    scope: scope
  })

# Check consecutive crashes
IF last N statuses are all "crash":
  fire_event("consecutive-crashes", {
    count: N,
    threshold: N,
    session: session_id,
    scope: scope
  })

# Check keep rate
session_keep_rate = keeps / total * 100
fire_event("keep-rate-drop", {
  keep_rate: session_keep_rate,
  threshold: 20,  # Will be matched by hook conditions
  session: session_id
})

# Check goal
IF metric meets goal:
  fire_event("goal-achieved", {
    metric: current_metric,
    goal: goal_value,
    session: session_id,
    iteration: iteration_number
  })

# Check metric threshold
fire_event("metric-threshold", {
  metric: current_metric,
  direction: metric_direction,
  session: session_id
})

# Session end:
fire_event("session-end", {
  session: session_id,
  keeps: keep_count,
  discards: discard_count,
  crashes: crash_count,
  keep_rate: keep_rate,
  baseline: baseline,
  final: final_metric
})
```

### In /autonexus:security and /autonexus:debug

```
# After each finding is created:
fire_event("finding-created", {
  severity: finding.severity,
  finding_type: "security" or "debug",
  title: finding.title,
  file: finding.file,
  session: session_id
})
```

### In /autonexus:agile

```
# After story completion:
fire_event("story-complete", {
  story_id: story.id,
  story_title: story.title,
  sprint: sprint_number,
  status: "done"
})

# After gate failure:
fire_event("gate-failed", {
  gate_name: gate.name,
  story_id: story.id,
  sprint: sprint_number,
  failure_count: gate.failures
})

# After sprint end:
fire_event("sprint-end", {
  sprint: sprint_number,
  stories_completed: completed_count,
  stories_total: total_count,
  velocity: velocity_points
})
```

### In /autonexus:chain

```
# After a pipeline step fails:
fire_event("pipeline-step-failed", {
  pipeline: pipeline_name,
  step: step_number,
  command: step.command,
  error: error_message
})
```

---

## Local Cache: autonexus-hooks.json

When Obsidian is unavailable, hooks are loaded from a local cache:

```json
{
  "project": "{ProjectName}",
  "cached_at": "{ISO timestamp}",
  "hooks": [
    {
      "name": "auto-debug-on-crashes",
      "enabled": true,
      "priority": 10,
      "event": "consecutive-crashes",
      "condition": "count >= 3",
      "action": {
        "type": "run-command",
        "command": "debug",
        "args": "--scope $SCOPE"
      },
      "message": "$COUNT consecutive crashes detected. Launching debug workflow.",
      "last_triggered": null,
      "trigger_count": 0
    }
  ]
}
```

This file is:
- Created/updated at session start when hooks are loaded from Obsidian
- Read as fallback when Obsidian is unavailable
- Gitignored (local-only, like autonexus-results.tsv)
- Updated with trigger_count and last_triggered after each hook fires

---

## Flags Summary

| Flag | Purpose |
|------|---------|
| `--add` | Interactive hook creation wizard |
| `--list` | Show all hooks (default when no flags) |
| `--enable <name>` | Enable a disabled hook |
| `--disable <name>` | Disable without deleting |
| `--remove <name>` | Permanently delete a hook |
| `--history` | Show hook execution history |
| `--dry-run <event>` | Simulate which hooks would fire |
| `--template <name>` | Create from built-in template |

## Error Handling

| Scenario | Response |
|----------|----------|
| Obsidian unavailable for management ops | "Hook management requires Obsidian." STOP |
| Hook name not found | "Hook '{name}' not found. Run --list to see available hooks." |
| Invalid condition syntax | Print error, suggest correct syntax, do NOT create hook |
| Hook action fails during evaluation | Log failure, NEVER block the loop, continue |
| Circular hook (hook triggers command that triggers same hook) | Detect and prevent: max 1 trigger per hook per iteration |
| Too many hooks firing (>5 per iteration) | Warn: "High hook activity — {N} hooks fired this iteration." Continue. |
| run-command action fails | Log failure, mark as "failed" in history, loop continues |
