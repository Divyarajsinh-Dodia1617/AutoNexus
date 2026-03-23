# /autonexus:auto-agent — Intelligent Auto-Scaling

Analyzes task complexity and automatically selects the right number and type of agents. Instead of manually choosing between single-agent, SPARC, or swarm modes, let the system assess the task and pick the optimal team. Includes a dry-run mode to preview the plan before execution.

## Syntax

```
# Let the system decide everything
/autonexus:auto-agent Implement dark mode across the app

# Dry-run to preview the team selection
/autonexus:auto-agent Implement dark mode across the app --dry-run

# Override complexity assessment
/autonexus:auto-agent Fix typo in README --complexity low

# Set a token budget constraint
/autonexus:auto-agent Build real-time notifications --budget 50000

# Limit scope to specific directories
/autonexus:auto-agent Refactor auth module --scope src/auth/**

# Prefer a specific workflow style
/autonexus:auto-agent Add search feature --prefer sparc
```

## What It Does

1. **Complexity assessment** — Analyzes the task description, affected codebase scope, dependency graph, and historical patterns from the ReasoningBank.
2. **Team selection** — Based on complexity, selects the workflow mode (single-agent, SPARC, swarm) and assigns specific agents with roles.
3. **Resource estimation** — Estimates token usage, time, and number of iterations needed.
4. **Execution** — Launches the selected workflow, passing through all configuration automatically.
5. **Post-mortem** — After completion, records the actual complexity vs. estimated for future calibration.

## Complexity Levels

| Level | Agents | Workflow | Typical Task |
|-------|--------|----------|--------------|
| `trivial` | 1 | Direct fix | Typos, config changes, small tweaks |
| `low` | 1–2 | `/autonexus:fix` or `/autonexus` | Bug fixes, small features |
| `medium` | 3–5 | `/autonexus:sparc` | New features, module refactors |
| `high` | 4–6 | `/autonexus:swarm --topology hierarchical` | Cross-cutting changes, architecture work |
| `extreme` | 6–8 | `/autonexus:swarm --topology mesh` | Full system redesigns, migrations |

## Flags

| Flag | Purpose |
|------|---------|
| `--complexity <level>` | Override the auto-assessed complexity (trivial, low, medium, high, extreme) |
| `--budget <tokens>` | Maximum token budget — team size is constrained to fit |
| `--dry-run` | Show the assessment and team plan without executing |
| `--scope <glob>` | Limit analysis and execution to files matching the glob |
| `--prefer <mode>` | Prefer a specific workflow mode (single, sparc, swarm) |

## Examples

```bash
# Auto-assess and execute
/autonexus:auto-agent Implement dark mode across the app

# Output:
# ┌─────────────────────────────────────────────┐
# │ Complexity Assessment                        │
# ├─────────────────────────────────────────────┤
# │ Task:            Implement dark mode          │
# │ Assessed scope:  14 components, 3 themes      │
# │ Complexity:      medium                        │
# │ Confidence:      0.87                          │
# ├─────────────────────────────────────────────┤
# │ Selected Workflow: SPARC (5-phase)             │
# │ Team:                                          │
# │   Spec Writer    — requirements + theme spec   │
# │   Architect      — theme system design         │
# │   Code Agent     — implementation              │
# │   QA Agent       — visual regression tests     │
# │   CTO            — quality gates               │
# │ Est. tokens:     ~32,000                       │
# │ Est. time:       ~8 minutes                    │
# └─────────────────────────────────────────────┘
# Proceeding with SPARC workflow...

# Dry-run to preview without executing
/autonexus:auto-agent Build real-time notifications --dry-run

# Output:
# [DRY RUN — no agents will be launched]
# Complexity: high (WebSocket + pub/sub + UI + persistence)
# Recommended: Swarm (hierarchical, 5 agents)
# Team: Backend Agent, Frontend Agent, Database Agent, QA Agent, CTO (queen)
# Est. tokens: ~48,000
# Est. time: ~12 minutes
# Run without --dry-run to execute.

# Override complexity when assessment is wrong
/autonexus:auto-agent Fix typo in README --complexity trivial
```

## Obsidian Integration

### What Gets Written

Auto-agent delegates to the selected workflow (SPARC, swarm, etc.), so Obsidian artifacts depend on which workflow is chosen. Additionally:

| What | Obsidian Path | When |
|------|---------------|------|
| Complexity assessment | `Projects/{ProjectName}/Sessions/{session}/auto-agent-assessment.md` | After assessment |
| Post-mortem calibration | `Projects/{ProjectName}/ReasoningBank/calibration.md` | After completion |

### What Gets Read
- ReasoningBank patterns for historical complexity calibration
- Project structure for scope analysis

## Without Obsidian

- Complexity assessment and team selection work fully without Obsidian
- `--dry-run` works without Obsidian
- Historical calibration data is not available (each assessment is independent)
- Delegated workflow behavior depends on that workflow's Obsidian requirements

## Tips

- **Use `--dry-run` first** to see what team would be assembled. This is especially useful for expensive tasks where you want to verify the plan.
- **Override with `--complexity`** if the assessment seems wrong — the system can underestimate for novel tasks or overestimate for tasks similar to past work.
- **`--budget` is useful for cost control** — if you have a token limit, the system will pick a smaller team that fits within budget.
- **`--scope` improves accuracy** — limiting the analysis scope helps the system better estimate complexity for focused changes.
- **Check the post-mortem** — after execution, the system records actual vs. estimated complexity, improving future assessments.
- **Combine with `/autonexus:optimize`** afterward to see actual token usage vs. the estimate.
