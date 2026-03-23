---
name: autonexus:mle-star
description: Machine Learning Engineering workflow — Search, Train, Ablate, Refine (STAR). Structured ML experiment lifecycle with Obsidian tracking.
argument-hint: "[ML goal] [--phase <search|train|ablate|refine|ensemble|validate|deploy>] [--dataset <path>] [--metric <name>] [--budget <iterations>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--phase <name>` or `Phase:` — run specific phase (search|train|ablate|refine|ensemble|validate|deploy)
- `--dataset <path>` or `Dataset:` — path to training data
- `--metric <name>` or `Metric:` — ML metric to optimize (accuracy, f1, loss, bleu, perplexity, etc.)
- `--budget <N>` or `Budget:` — iteration budget per phase. Default: 10
- `--direction <higher|lower>` or `Direction:` — metric direction. Default: higher_is_better
- `--model-file <path>` or `Model:` — path to model definition file
- `--config <path>` or `Config:` — path to training configuration

Remaining unmatched text = ML goal description.

## Execution

1. Read core principles (ML metric section): `.claude/skills/autonexus/references/core-principles.md`
2. Read autonomous loop protocol: `.claude/skills/autonexus/references/autonomous-loop-protocol.md`
3. Read reasoning bank: `.claude/skills/autonexus/references/reasoning-bank.md`
4. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
5. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
6. If no ML goal → use `AskUserQuestion`:
   - Q1 (Goal): "What ML goal are you pursuing?" with [Improve classification accuracy, Reduce training loss, Increase F1 score, Optimize inference latency, Improve BLEU/perplexity, Custom]
   - Q2 (Dataset): "Where is your training data?" with detected paths from project
   - Q3 (Metric): "Which metric to optimize?" with [accuracy, loss, f1_score, bleu, perplexity, custom]
   - Q4 (Budget): "How many iterations per phase?" with [5, 10 (Recommended), 25, 50]
7. Create experiment folder in Obsidian: `Projects/{ProjectName}/MLE-STAR/{experiment}/`
8. Execute 7-phase MLE-STAR workflow:

   **Phase 1 — Search (Literature & SOTA):**
   - Web search for state-of-the-art approaches for the goal
   - Identify top 3-5 promising techniques
   - Write findings to `MLE-STAR/{experiment}/Search.md`

   **Phase 2 — Train (Baseline):**
   - Run baseline training with current/standard configuration
   - Record baseline metric as iteration #0
   - Write baseline to `MLE-STAR/{experiment}/Baseline.md`

   **Phase 3 — Ablate (Systematic Ablation):**
   - Systematically remove/modify components
   - Measure impact of each component
   - Identify which components contribute most
   - Write ablation results to `MLE-STAR/{experiment}/Ablation.md`

   **Phase 4 — Refine (Autoresearch Loop):**
   - Run the main autoresearch loop on best configuration
   - Each iteration: modify → train → verify → keep/discard
   - Budget: N iterations (from --budget flag)
   - Track results in `autonexus-results.tsv`

   **Phase 5 — Ensemble (Combine Winners):**
   - Take top-performing variants from Refine phase
   - Combine via averaging, stacking, or voting
   - Verify ensemble outperforms individual models
   - Write ensemble results to `MLE-STAR/{experiment}/Ensemble.md`

   **Phase 6 — Validate (Cross-Validation):**
   - Run k-fold cross-validation on best model
   - Test on holdout set
   - Statistical significance testing
   - Write validation report to `MLE-STAR/{experiment}/Validation.md`

   **Phase 7 — Deploy (Export & Pipeline):**
   - Export best model (save weights/checkpoint)
   - Create inference pipeline/script
   - Document model card (architecture, metrics, limitations)
   - Write deployment guide to `MLE-STAR/{experiment}/Deploy.md`

9. Final report: all phases, best metric achieved, configuration, artifacts, model card

Stream all output live — never run in background.
