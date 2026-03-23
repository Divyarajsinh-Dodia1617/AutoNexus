# /autonexus:mle-star — ML Engineering Workflow

Runs a structured 7-phase ML engineering workflow: literature search, data preparation, model implementation, training, evaluation, ablation study, and refinement. Designed for iterative experiment loops where each phase builds on the previous. Supports targeting a specific metric threshold and budget-controlled iterations.

## Syntax

```
# Full MLE-STAR workflow
/autonexus:mle-star Improve accuracy to 95% --dataset data/train.csv --metric accuracy

# Specify optimization direction
/autonexus:mle-star Reduce inference latency --metric latency_ms --direction minimize

# Run a single phase
/autonexus:mle-star --phase literature-search --dataset data/train.csv

# Set iteration budget per phase
/autonexus:mle-star Improve F1 score --dataset data/train.csv --metric f1 --budget 10

# Target a specific model file
/autonexus:mle-star Optimize model --model-file models/classifier.py --metric accuracy
```

## What It Does

1. **Literature Search** — Searches for state-of-the-art approaches relevant to the task. Uses web search to find recent papers, benchmarks, and techniques. Produces a summary of promising approaches.
2. **Data Preparation** — Analyzes the dataset: statistics, distributions, missing values, class balance. Applies preprocessing, augmentation, and feature engineering as needed.
3. **Model Implementation** — Implements the model architecture based on literature findings and data analysis. Creates training scripts, loss functions, and evaluation harness.
4. **Training** — Runs training with appropriate hyperparameters. Monitors convergence, adjusts learning rate, applies early stopping.
5. **Evaluation** — Evaluates on held-out data. Produces metrics, confusion matrices, error analysis, and comparison against baselines.
6. **Ablation Study** — Systematically removes or varies components to understand what contributes to performance. Tests feature importance, architecture choices, and hyperparameter sensitivity.
7. **Refinement** — Based on evaluation and ablation insights, iterates on the model. Adjusts architecture, hyperparameters, or data pipeline. Repeats until the target metric is reached or budget is exhausted.

## Flags

| Flag | Purpose |
|------|---------|
| `--phase <name>` | Run only the specified phase (literature-search, data-prep, implementation, training, evaluation, ablation, refinement) |
| `--dataset <path>` | Path to the training dataset |
| `--metric <name>` | Target metric to optimize (accuracy, f1, loss, latency_ms, etc.) |
| `--budget <n>` | Maximum iterations per phase (default: 20) |
| `--direction <dir>` | Optimization direction: maximize (default) or minimize |
| `--model-file <path>` | Path to an existing model file to optimize |

## Examples

```bash
# Full workflow targeting 95% accuracy
/autonexus:mle-star Improve accuracy to 95% --dataset data/train.csv --metric accuracy

# Output from each phase:
#
# Phase 1 — Literature Search
#   ✓ Found 8 relevant papers/approaches
#   ✓ Top recommendation: ensemble of gradient boosting + attention-based features
#   ✓ Baseline SOTA on similar datasets: 93.2% accuracy
#   ⏱ 45s
#
# Phase 2 — Data Preparation
#   ✓ Dataset: 12,450 samples, 34 features, 5 classes
#   ✓ Class imbalance detected (ratio 1:8), applied SMOTE
#   ✓ 3 features dropped (>50% missing), 2 engineered
#   ✓ Train/val/test split: 70/15/15
#   ⏱ 30s
#
# Phase 3 — Model Implementation
#   ✓ Implemented: GradientBoostingClassifier + feature attention layer
#   ✓ Created: models/classifier.py, train.py, evaluate.py
#   ✓ Evaluation harness with cross-validation
#   ⏱ 60s
#
# Phase 4 — Training
#   ✓ Training: 50 estimators, lr=0.1, max_depth=6
#   ✓ Converged at epoch 42, val_accuracy=0.928
#   ✓ Early stopping triggered (patience=5)
#   ⏱ 90s
#
# Phase 5 — Evaluation
#   ✓ Test accuracy: 92.8%
#   ✓ Per-class F1: [0.95, 0.91, 0.93, 0.88, 0.90]
#   ✓ Error analysis: class 4 confused with class 2 (38% of errors)
#   ⏱ 20s
#
# Phase 6 — Ablation Study
#   ✓ Feature importance: top 5 features contribute 72% of performance
#   ✓ SMOTE removal: -3.1% accuracy (keep SMOTE)
#   ✓ Depth reduction (6→4): -1.2% accuracy (keep depth=6)
#   ✓ Attention layer removal: -2.4% accuracy (keep attention)
#   ⏱ 120s
#
# Phase 7 — Refinement (3 iterations)
#   Iteration 1: Added class-weighted loss → 93.5% (+0.7%)
#   Iteration 2: Increased estimators to 80 → 94.2% (+0.7%)
#   Iteration 3: Added feature cross-products → 95.1% (+0.9%)
#   ✓ Target reached: 95.1% accuracy (target: 95%)
#   ⏱ 180s
#
# Total: ~9 minutes | Final accuracy: 95.1%

# Run just the literature search phase
/autonexus:mle-star --phase literature-search --dataset data/train.csv

# Minimize latency with a budget
/autonexus:mle-star Reduce inference latency --metric latency_ms --direction minimize --budget 10
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Literature summary | `Projects/{ProjectName}/MLE-STAR/{experiment}/Literature.md` | After Phase 1 |
| Data analysis report | `Projects/{ProjectName}/MLE-STAR/{experiment}/Data-Preparation.md` | After Phase 2 |
| Model architecture | `Projects/{ProjectName}/MLE-STAR/{experiment}/Implementation.md` | After Phase 3 |
| Training log | `Projects/{ProjectName}/MLE-STAR/{experiment}/Training-Log.md` | After Phase 4 |
| Evaluation report | `Projects/{ProjectName}/MLE-STAR/{experiment}/Evaluation.md` | After Phase 5 |
| Ablation results | `Projects/{ProjectName}/MLE-STAR/{experiment}/Ablation-Study.md` | After Phase 6 |
| Refinement log | `Projects/{ProjectName}/MLE-STAR/{experiment}/Refinement-Log.md` | After Phase 7 (updated per iteration) |

### What Gets Read
- Prior experiment results for comparison baselines
- ReasoningBank for ML-specific learned patterns

## Without Obsidian

- All 7 phases execute normally — model code, training, and evaluation work via git and filesystem
- No experiment documentation is persisted to Obsidian
- Prior experiment baselines are not available for comparison
- `--phase` still works (uses filesystem artifacts from prior phases)

## Tips

- **Start with `--budget 10` per phase** to validate your setup before committing to a full run. This limits iterations in the refinement phase so you can verify the pipeline works end-to-end.
- **Use `--phase` to run individual phases** when you want manual control — e.g., run literature search first, review it, then continue.
- **Literature search phase uses web search** to find state-of-the-art approaches, so it produces genuinely current recommendations.
- **`--direction minimize`** is important for metrics like loss, latency, or error rate where lower is better.
- **`--model-file`** lets you optimize an existing model instead of starting from scratch — useful for iterating on a model you've already partially built.
- **Ablation study is the most valuable phase** for understanding your model — don't skip it even if you've already hit your metric target.
- **Pair with `/autonexus:reason --search "ML"`** to see what ML patterns the system has learned from prior experiments.
