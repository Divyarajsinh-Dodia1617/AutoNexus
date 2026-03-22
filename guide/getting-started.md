# Getting Started with AutoNexus

## Prerequisites

1. **Claude Code** — [Install](https://docs.anthropic.com/en/docs/claude-code)
2. **Obsidian** — [Download](https://obsidian.md) desktop app
3. **Local REST API plugin** — Install from Obsidian's Community Plugins:
   - Open Obsidian → Settings → Community Plugins → Browse
   - Search "Local REST API" → Install → Enable
   - Copy the API key from the plugin settings

## Step 1: Configure Obsidian MCP

Add to your Claude Code MCP config. The file location depends on your setup:

- **Per-project:** `.mcp.json` in your project root
- **Global:** `~/.claude/.mcp.json`

```json
{
  "mcpServers": {
    "obsidian-mcp-server": {
      "command": "npx",
      "args": ["obsidian-mcp-server"],
      "env": {
        "OBSIDIAN_API_KEY": "<paste your API key here>",
        "OBSIDIAN_BASE_URL": "http://127.0.0.1:27123",
        "OBSIDIAN_VERIFY_SSL": "false",
        "OBSIDIAN_ENABLE_CACHE": "true"
      }
    }
  }
}
```

Restart Claude Code after adding the config.

## Step 2: Install AutoNexus

### Option A: Plugin install

```
/plugin marketplace add divya/autonexus
/plugin install autonexus@autonexus
```

### Option B: Manual copy

```bash
git clone https://github.com/divya/autonexus.git

# Project-level
cp -r autonexus/claude-plugin/skills/autonexus .claude/skills/autonexus
cp -r autonexus/claude-plugin/commands/autonexus .claude/commands/autonexus
cp autonexus/claude-plugin/commands/autonexus.md .claude/commands/autonexus.md

# Or global
cp -r autonexus/claude-plugin/skills/autonexus ~/.claude/skills/autonexus
cp -r autonexus/claude-plugin/commands/autonexus ~/.claude/commands/autonexus
cp autonexus/claude-plugin/commands/autonexus.md ~/.claude/commands/autonexus.md
```

## Step 3: Verify Installation

In Claude Code, type:

```
/autonexus:plan
Goal: Test that AutoNexus is working
```

If the wizard starts asking about scope and metrics, you're set.

## Step 4: Verify Obsidian Connection

Run a quick 3-iteration test:

```
/autonexus
Goal: Add a comment to any file
Scope: README.md
Metric: line count (higher)
Verify: wc -l README.md | awk '{print $1}'
Iterations: 3
```

After it completes, check your Obsidian vault for:
- `Projects/{YourProject}/Iterations/{today}/` — 3 iteration notes
- A session summary note
- An entry in your daily note

## Core Concepts

### The Loop

AutoNexus runs an autonomous iteration loop:

1. **Review** — Read files + git history + TSV log + Obsidian (failed approaches, predict warnings)
2. **Ideate** — Pick next change, informed by all knowledge sources
3. **Modify** — Make ONE atomic change
4. **Commit** — `experiment(<scope>): <description>`
5. **Verify** — Run mechanical metric
6. **Guard** — Optional safety net command
7. **Decide** — Keep (improved) or revert (same/worse)
8. **Log** — TSV (always) + Obsidian note (best-effort)
9. **Repeat** — Forever or N times

### Metric vs Guard

- **Metric** (Verify) = "Did it get better?" — the goal you're optimizing
- **Guard** = "Did anything else break?" — optional safety net

### Obsidian Integration

Every iteration:
- **Reads** from Obsidian: past failed approaches + predict persona warnings
- **Writes** to Obsidian: concise 5-8 line iteration note

Session boundaries:
- **Session summary** — top wins, key failed approaches, next suggestions
- **Daily note** — mini summary appended
- **Pre-push** — full Knowledge Center rebuild

### Bounded vs Unbounded

| Mode | Behavior |
|------|----------|
| Unbounded (default) | Loops forever until Ctrl+C |
| Bounded (`Iterations: N`) | Loops exactly N times, then prints summary |

## 9 Principles

1. **Constraint = Enabler** — Narrow scope, single metric, fixed cost
2. **Separate Strategy from Tactics** — Humans set direction, agents iterate
3. **Metrics Must Be Mechanical** — If a command can't verify it, you can't iterate
4. **Verification Must Be Fast** — <30 seconds enables rapid experimentation
5. **Iteration Cost Shapes Behavior** — Cheap = bold; expensive = conservative
6. **Git as Memory** — Every experiment committed, history informs next iteration
7. **Honest Limitations** — State what you can't do clearly
8. **Knowledge Persists Beyond Code** — Obsidian captures WHY, not just WHAT
9. **Obsidian Is Best-Effort** — Never block the loop on a knowledge write

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Obsidian MCP unavailable" | Ensure Obsidian is open + Local REST API plugin enabled + API key correct in .mcp.json |
| Commands not found | Run `/reload-plugins` or restart Claude Code |
| Verify command fails dry run | Test the command manually first: `npm test -- --coverage` |
| Too many files in scope | Narrow your glob: `src/api/**/*.ts` instead of `src/**/*.ts` |
| Obsidian notes not appearing | Check vault path — notes go to `Projects/{name}/Iterations/` |
