# ReasoningBank Protocol — AutoNexus

Self-learning system inspired by ruflo SONA. Every execution feeds back into pattern recognition through RETRIEVE-JUDGE-DISTILL stored in Obsidian. The system gets smarter with every session.

## The RETRIEVE-JUDGE-DISTILL Cycle

### RETRIEVE (Main Loop Phase 1: Review)

Search Obsidian for patterns matching current context: task_type, domain, file_types, metric_type, complexity. Return top 3-5 with confidence scores.

### JUDGE (Main Loop Phase 2: Ideate)

Evaluate retrieved patterns: context_similarity > 70 AND confidence > 0.5 = RECOMMEND. Context > 50 AND confidence > 0.3 = SUGGEST. Otherwise SKIP.

### DISTILL (Main Loop Phase 7: Log)

IF keep: Extract pattern (what, why, context), generalize, tag, score confidence, store.
IF discard: Record negative evidence, reduce matching pattern confidence by 0.1.

## Strategy Scoring

Track per-category: Attempts, Keeps, Keep Rate, Avg Delta, Confidence, Trend. Bias toward categories with keep rate > 60%.

## Confidence Decay

Weekly without re-validation: -0.05. Below 0.3: flag for pruning. Below 0.1: auto-archive. Re-validation success: +0.1 (cap 1.0).

## Meta-Learning (Every 10 Iterations)

Meta-learner analyzes: strategy trends, diminishing returns, routing effectiveness, pattern quality, transfer effectiveness.

## Transfer Learning

Import from other projects at 50% confidence. Tag as transfer. Promote on success.

## Obsidian Layout

Projects/{ProjectName}/ReasoningBank/ — Patterns/, Strategies/scoring.md, Meta/, Transfer/

## Integration Points

- Phase 1: RETRIEVE, Phase 2: JUDGE, Phase 7: DISTILL
- Every 10 iterations: Meta-learning
- /autonexus:reason: Manual management
