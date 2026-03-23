# Background Workers Protocol — AutoNexus

Auto-dispatched context-triggered workers alongside primary workflows.

## Worker Types (8)

| Worker | Trigger | Action |
|--------|---------|--------|
| Security Scanner | File modified | Quick security pattern scan |
| Performance Monitor | Iteration complete | Check perf anti-patterns |
| Pattern Learner | Every 5 iterations | Update ReasoningBank |
| Session Persister | Every iteration | TSV + Obsidian sync |
| Knowledge Updater | Every 10 iterations | Check Knowledge Center freshness |
| Dependency Checker | Import-file modified | Verify dependency health |
| Style Enforcer | After code mod | Linter/formatter check |
| Documentation Sync | After keep | Check if docs need update |

## Constraints
Max 3 concurrent. Max 1 per type. Never block primary workflow. Never modify source code. Minimal claims.

## Obsidian Layout
Projects/{ProjectName}/Workers/ -- per-worker reports.

## Integration Points
Triggered by main loop, agile, swarm events. Fire worker-finding event.
