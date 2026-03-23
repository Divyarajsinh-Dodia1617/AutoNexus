# Core Principles — AutoNexus

14 universal principles for autonomous goal-directed iteration with persistent knowledge.
Principles 1-7 adapted from Karpathy's autoresearch. Principles 8-9 are AutoNexus additions. Principles 10-14 are ruflo-inspired additions.

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

## 10. Swarm Intelligence Over Solo Execution

**Ruflo-inspired addition.** When tasks exceed single-agent capacity, coordinate multiple agents through structured swarm topologies rather than sequential handoffs.

| Solo Agent | Swarm |
|------------|-------|
| One context window, one perspective | Multiple perspectives, parallel exploration |
| Sequential bottleneck | Parallel execution with consensus |
| Single point of failure | Fault tolerance through redundancy |

**Apply:** For tasks spanning >3 domains or >10 files, activate swarm mode with queen coordination. Use hierarchical topology for coding, mesh for research.

## 11. Least Privilege by Default

**Ruflo-inspired addition.** Every agent operates with the minimum permissions needed for its task. Claims auto-revoke on completion.

- Junior agents get minimal scope (their assigned files only)
- Senior agents get standard scope (project-wide read, in-scope write)
- Executive agents get elevated scope (time-limited)
- No agent gets admin by default

**Apply:** At agent spawn, assign claims based on tier and task. Monitor for violations. Revoke on completion.

## 12. Route to the Cheapest Capable Handler

**Ruflo-inspired addition.** Not every task needs an opus-tier agent. Route deterministic transforms to direct edits, simple tasks to haiku/sonnet, and reserve opus for genuine complexity.

| Task Complexity | Handler | Cost |
|----------------|---------|------|
| Deterministic (rename, format) | Direct Edit tool | $0 |
| Simple (single-file, well-understood) | Haiku/Sonnet agent | Low |
| Complex (cross-file, novel) | Opus agent or Swarm | High |

**Apply:** Assess complexity before spawning agents. Track routing decisions and savings.

## 13. Self-Learning Through RETRIEVE-JUDGE-DISTILL

**Ruflo-inspired addition.** The system improves with every execution. Past patterns inform current decisions through a structured learning cycle.

1. **RETRIEVE** — Before ideating, search for similar past patterns
2. **JUDGE** — Evaluate retrieved patterns against current context
3. **DISTILL** — After execution, extract and store new patterns

**Apply:** Every iteration consults the ReasoningBank. Every result feeds back into it. Confidence scores decay without re-validation.

## 14. Anti-Drift by Design

**Ruflo-inspired addition.** Multi-agent systems naturally drift from goals. Prevent this structurally, not reactively.

| Prevention Layer | Mechanism |
|-----------------|-----------|
| Structural | Hierarchical topology, capped agents, specialized roles |
| Checkpoints | Goal alignment check every 3 worker completions |
| Gates | Sequential quality gates at key transitions |
| Recovery | Pause → re-align → resume (or halt if severe) |

**Apply:** Default to hierarchical topology. Cap concurrent agents at 8. Check alignment frequently. Treat drift as a system failure, not an agent failure.

## The Meta-Principle

> Autonomy scales when you constrain scope, clarify success, mechanize verification, persist knowledge, coordinate through swarms, enforce least privilege, optimize cost routing, learn from every execution, prevent drift by design, and let agents optimize tactics while humans optimize strategy.

This isn't "removing humans." It's reassigning human effort from execution to direction — and ensuring every insight survives the session that discovered it.
