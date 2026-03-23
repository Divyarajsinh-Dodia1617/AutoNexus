# Three-Tier Task Routing — AutoNexus

Route tasks to cheapest capable handler.

## Tiers

| Tier | Handler | Use Case |
|------|---------|----------|
| 1 Direct Edit | Edit/Write tools | Deterministic: rename, import, typo, format |
| 2 Sonnet Agent | Spawn sonnet | Simple: single-file function, test, docs |
| 3 Opus/Swarm | Spawn opus or swarm | Complex: architecture, debug, cross-cutting |

## Complexity Assessment (0-50)
File breadth + Domain depth + Novelty + Risk + Reasoning depth (each 0-10).
Score 0-5: Tier 1. Score 6-20: Tier 2. Score 21-50: Tier 3.

## Fallback Chain
Tier 1 fails -> Tier 2. Tier 2 fails twice -> Tier 3. Tier 3 fails -> Human.

## Cost Tracking
Per-session in Optimization/routing-log.md.

## Integration Points
Phase 3 (Modify), agile story assignment, SPARC Refinement. Tracked in Obsidian.
