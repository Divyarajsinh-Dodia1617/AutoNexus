# /autonexus:chain — Workflow Composition

Chains multiple autonexus commands into a sequential pipeline. Define steps inline, use built-in templates, or load saved pipelines from Obsidian. Each step's output becomes context for the next. Supports conditions, failure handling, and resumable runs.

## Syntax

```
# Inline pipeline with arrow notation
/autonexus:chain predict → agile → review → ship

# Use a built-in template
/autonexus:chain --template full-sprint

# Load a saved pipeline from Obsidian
/autonexus:chain --pipeline my-release-pipeline

# With step arguments
/autonexus:chain predict --depth deep → security → fix → ship --auto

# Dry-run (validate without executing)
/autonexus:chain --template quality-audit --dry-run

# Resume an interrupted pipeline
/autonexus:chain --resume 20260322-1430-chain-a7x2

# List saved pipelines
/autonexus:chain --list

# Execute and save for reuse
/autonexus:chain predict → agile → ship --save "sprint-pipeline"
```

## What It Does

1. **Resolves** the pipeline (inline, template, saved, or interactive wizard)
2. **Validates** all steps — checks command names, conditions, failure handlers
3. **Executes** each step sequentially:
   - Checks the step's condition (on-success, on-failure, always, metric threshold)
   - Runs the autonexus command with its arguments
   - Captures the result (success/failed/skipped)
   - Logs the step result
4. **Summarizes** — prints a table of all step results with durations
5. **Persists** — writes a run log to Obsidian, optionally saves the pipeline definition

## Built-in Templates

| Template | Pipeline | Best For |
|----------|----------|----------|
| `full-sprint` | predict → agile → review → ship | Full development cycle |
| `quality-audit` | security → debug → fix → learn | Comprehensive quality check |
| `release-prep` | fix → security → learn → ship --dry-run | Release preparation |
| `deep-analysis` | learn → predict → scenario → security | Deep understanding before major work |
| `continuous-quality` | fix → security → debug → review --last | Ongoing quality maintenance |

## Step Conditions

| Condition | Meaning |
|-----------|---------|
| `?on-success` | Run only if previous step succeeded (default) |
| `?on-failure` | Run only if previous step failed |
| `?always` | Always run regardless of previous step |
| `?if-metric-above:N` | Run if last metric value > N |
| `?if-metric-below:N` | Run if last metric value < N |

## Failure Handling

| Handler | Meaning |
|---------|---------|
| `!skip` | Log failure, continue to next step (default) |
| `!halt` | Stop the entire pipeline |
| `!retry:N` | Retry the step up to N times |

## Flags

| Flag | Purpose |
|------|---------|
| `--pipeline <name>` | Load a saved pipeline from Obsidian |
| `--template <name>` | Use a built-in template |
| `--dry-run` | Validate and preview without executing |
| `--halt-on-error` | Stop on first step failure |
| `--resume <run-id>` | Resume an interrupted pipeline |
| `--save <name>` | Save the pipeline to Obsidian for reuse |
| `--list` | List all saved pipelines |

## Examples

```bash
# Simple 3-step pipeline
/autonexus:chain security → fix → ship

# With conditions: always debug, only fix if debug found bugs
/autonexus:chain debug ?always → fix ?on-success → review --last ?always

# Halt on security issues, retry fixes
/autonexus:chain security !halt → fix !retry:2 → learn → ship

# Full sprint with auto-ship
/autonexus:chain --template full-sprint

# Save a custom pipeline for the team
/autonexus:chain predict --depth deep → agile --team standard → ship --auto --save "team-sprint"
```

## Obsidian Integration

### What Gets Read
- **Saved pipelines:** `Projects/{ProjectName}/Pipelines/{name}.md`
- **Run logs (for resume):** `Projects/{ProjectName}/Pipelines/runs/{run-id}.md`

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Pipeline run log | `Projects/{ProjectName}/Pipelines/runs/{run-id}.md` | After every pipeline execution |
| Pipeline definition | `Projects/{ProjectName}/Pipelines/{name}.md` | When `--save` is used |

### Run Log Contains
- Pipeline name, run ID, timestamps, overall status
- Per-step: command, status, duration, attempt count, summary, artifact links
- Pipeline context: last metric, session IDs, finding counts

## Without Obsidian

- Inline pipelines and built-in templates work fully without Obsidian
- `--pipeline`, `--resume`, `--list`, and `--save` require Obsidian
- No run logs are written, but steps execute normally
- Context passing between steps works via git state and TSV

## Tips

- **Start with templates** — `--template full-sprint` is a great default for most development work.
- **Use `--dry-run` first** for complex pipelines to verify the execution plan.
- **Save pipelines** that you run repeatedly — avoids retyping steps every time.
- **Conditions are powerful** — `?on-failure` lets you build fallback paths (e.g., "if security audit fails, run fix").
- **Pipeline steps share git state** — changes from step 1 are visible to step 2. This is what makes `fix → ship` work.
- **Resumable runs** are useful for long pipelines that get interrupted — no need to re-run completed steps.
