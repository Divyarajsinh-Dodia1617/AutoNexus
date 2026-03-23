# Anti-Drift Protocol — AutoNexus

Structural enforcement to prevent multi-agent drift. The #1 failure mode of autonomous agent systems.

## Drift Types

| Type | Detection |
|------|-----------|
| Goal Drift | Compare output vs goal every N completions |
| Scope Drift | Monitor git diff vs approved scope |
| Quality Drift | Track rolling keep rate |
| Style Drift | Run linter after each completion |
| Consensus Drift | Diff agent outputs for contradictions |

## Enforcement Layers

Layer 1 Structural (Always): Hierarchical topology. Max 8 agents. Specialized roles.

Layer 2 Checkpoints (Every N completions): Read goal. Compare outputs. Check scope. Verify metric.
N = 1 (strict), 3 (standard), 5 (relaxed).

Layer 3 Gates: Design->Implementation, Implementation->Review, Review->QA, QA->Done.

## Detection Metrics

| Metric | Threshold | Action |
|--------|-----------|--------|
| Goal alignment | < 70 | Pause and re-align |
| Scope violations | > 0 | Immediate revert |
| Keep rate rolling 10 | < 30% | Strategy reset |
| Agent conflicts | > 2 | Queen mediates |

## Recovery
MILD (50-70): Queen clarifies. MODERATE (30-50): Pause, re-decompose. SEVERE (<30): Halt, revert, restart.

## Integration Points
Active during swarm and agile. Fires: drift-detected, drift-recovered, drift-severe.
