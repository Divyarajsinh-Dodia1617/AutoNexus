# Core Principles — AutoNexus

9 universal principles for autonomous goal-directed iteration with persistent knowledge.
Principles 1-7 adapted from Karpathy's autoresearch. Principles 8-9 are AutoNexus additions.

## 1. Constraint = Enabler

Autonomy succeeds through intentional constraint, not despite it.

| Constraint | Why It Helps |
|------------|-------------|
| Bounded scope that fits agent context | Full context understood — no guessing |
| Fixed iteration cost | Rapid feedback loops |
| Single mechanical success criterion | No ambiguity in keep/discard decisions |

**Apply:** Before starting, define: what files are in-scope? What's the ONE metric? What's the time budget per iteration?

## 2. Separate Strategy from Tactics

Humans set direction. Agents execute iterations.

| Strategic (Human) | Tactical (Agent) |
|-------------------|------------------|
| "Improve page load speed" | "Lazy-load images, code-split routes" |
| "Increase test coverage" | "Add tests for uncovered edge cases" |
| "Refactor auth module" | "Extract middleware, simplify handlers" |

**Why:** Humans understand WHY. Agents handle HOW. Mixing these roles wastes both human creativity and agent iteration speed.

## 3. Metrics Must Be Mechanical

If you can't verify with a command, you can't iterate autonomously.

- Tests pass/fail (exit code 0)
- Benchmark time in milliseconds
- Coverage percentage
- Lighthouse score
- File size in bytes
- Lines of code count

**Anti-pattern:** "Looks better", "probably improved", "seems cleaner" — these KILL autonomous loops because there's no decision function.

**Apply:** Define the command that extracts your metric BEFORE starting.

### ML Accuracy Metric — Complete Configuration

```
/autonexus
Goal: Improve model accuracy from 85% to 95%
Scope: model.py, config.yaml, data/preprocessing.py
Metric: validation accuracy (higher is better)
Verify: python train.py --eval-only 2>&1 | grep 'val_accuracy' | awk '{print $2}'
Guard: python -c "import torch; m=torch.load('model.pt'); assert m is not None"
Iterations: 20
```

**Python metric extraction patterns:**

```bash
# Classification accuracy
python train.py --eval 2>&1 | grep 'accuracy' | awk '{print $NF}'

# Validation loss (lower is better)
python train.py 2>&1 | grep 'val_loss' | tail -1 | awk '{print $NF}'

# F1 score
python evaluate.py --metric f1 2>&1 | grep 'f1_score' | awk '{print $2}'

# BLEU score (NLP)
python evaluate.py 2>&1 | grep 'BLEU' | grep -oP '[\d.]+'

# Custom metric extraction
python -c "
import json
with open('eval_results.json') as f:
    results = json.load(f)
print(results['accuracy'])
"
```

**Error handling for ML verification:**

```bash
# Wrap verify command with timeout and error handling
timeout 300 python train.py --eval-only 2>&1 | grep 'val_accuracy' | awk '{print $2}' || echo "0.0"

# Verify the metric is a valid number
METRIC=$(python train.py --eval 2>&1 | grep 'accuracy' | awk '{print $NF}')
echo "$METRIC" | grep -qP '^\d+\.?\d*$' || { echo "WARN: metric '$METRIC' is not a number"; METRIC="0.0"; }
```

## 4. Verification Must Be Fast

If verification takes longer than the work itself, incentives misalign.

| Fast (enables iteration) | Slow (kills iteration) |
|-------------------------|----------------------|
| Unit tests (seconds) | Full E2E suite (minutes) |
| Type check (seconds) | Manual QA (hours) |
| Lint check (instant) | Code review (async) |

**Apply:** Use the FASTEST verification that still catches real problems. Save slow verification for after the loop.

## 5. Iteration Cost Shapes Behavior

- Cheap iteration → bold exploration, many experiments
- Expensive iteration → conservative, few experiments

Software: 10-second test → 360 experiments/hour.

**Apply:** Minimize iteration cost. Use fast tests, incremental builds, targeted verification. Every minute saved = more experiments run.

## 6. Git as Memory and Audit Trail

Every successful change is committed. This enables:
- **Causality tracking** — which change drove improvement?
- **Stacking wins** — each commit builds on prior successes
- **Pattern learning** — agent sees what worked in THIS codebase
- **Human review** — developer inspects agent's decision sequence

**Apply:** Commit before verify. Revert on failure. Agent reads its own git history to inform next experiment.

**Key commands the agent runs every iteration:**
```bash
git log --oneline -20           # see experiment sequence (kept vs reverted)
git diff HEAD~1                 # inspect last kept change to understand WHY it worked
git show <hash> --stat          # deep-dive specific commit to see which files drove improvement
```

**Without Git Memory (agent has no history — repeats failures):**
```
Iteration 1: Try increasing batch size → OOM crash → reverted
Iteration 5: Try increasing batch size → OOM crash → REPEATED!
Iteration 9: Try increasing batch size → OOM crash → WASTED!
```

**With Git Memory (agent reads git log — learns and adapts):**
```
Iteration 1: Try increasing batch size → OOM crash → git revert (preserved in history)
Iteration 2: git log shows "experiment: increase batch size — REVERTED"
             → Agent tries DIFFERENT approach: reduce model layers → metric improves → KEPT
Iteration 3: git diff HEAD~1 shows which layers were removed
             → Agent tries removing another layer → metric improves → KEPT
```

## 7. Honest Limitations

State what the system can and cannot do. Don't oversell.

**AutoNexus CANNOT:** replace human strategic direction, guarantee meaningful improvements, solve problems that require external access it doesn't have.

**Apply:** At setup, explicitly state constraints. If agent hits a wall it can't solve (missing permissions, external dependency, needs human judgment), say so clearly instead of guessing.

## 8. Knowledge Persists Beyond Code

**AutoNexus addition.** Decisions, rationale, failed approaches, and codebase understanding persist in Obsidian across sessions, branches, and projects.

Git captures WHAT changed. Obsidian captures WHY it changed, what alternatives were rejected, and what patterns emerge across projects.

| Without Persistent Knowledge | With Persistent Knowledge |
|------------------------------|--------------------------|
| New session starts from scratch | New session reads past decisions + failed approaches |
| Same failed approach retried | Agent queries Obsidian: "this was tried and discarded" |
| Lessons siloed per project | Cross-project patterns inform new work |
| Rationale lost after squash/rebase | Decisions survive all git operations |

**Apply:** Every iteration writes a concise note to Obsidian. Every ideation phase reads from Obsidian. Knowledge accumulates, never resets.

## 9. Obsidian Is Best-Effort

**AutoNexus addition.** The Obsidian knowledge layer is valuable but NEVER critical-path.

- If Obsidian MCP fails, the loop continues using git + local TSV
- If Obsidian is unavailable, a single warning is logged — no retries, no prompts
- The local TSV results log is ALWAYS the mechanical source of truth
- Obsidian enriches the loop but never gates it

**Apply:** Every Obsidian MCP call uses the retry-then-queue protocol. Failed writes queue locally and drain later. The loop never blocks, never crashes, never pauses for Obsidian.

## The Meta-Principle

> Autonomy scales when you constrain scope, clarify success, mechanize verification, persist knowledge, and let agents optimize tactics while humans optimize strategy.

This isn't "removing humans." It's reassigning human effort from execution to direction — and ensuring every insight survives the session that discovered it.
