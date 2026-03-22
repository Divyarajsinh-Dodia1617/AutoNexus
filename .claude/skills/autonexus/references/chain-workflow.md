# Chain Workflow — /autonexus:chain

Workflow composition engine that chains multiple autonexus commands into a sequential pipeline. Each step's output becomes context for the next step. Pipelines can be defined inline, loaded from Obsidian, or selected from built-in templates. Run logs persist in Obsidian for resumability and analysis.

**Core idea:** Individual autonexus commands are powerful alone. Chaining them creates compound workflows — predict before building, debug before fixing, learn before shipping. The pipeline engine handles sequencing, conditions, context passing, and logging so you don't have to invoke each command manually.

## Trigger

- User invokes `/autonexus:chain`
- User says "chain commands", "run a pipeline", "run predict then agile then ship", "compose workflows"
- User says "list pipelines", "show saved pipelines", "resume pipeline"

## PREREQUISITE: Interactive Setup (when invoked without flags or inline steps)

**CRITICAL — BLOCKING PREREQUISITE:** If `/autonexus:chain` is invoked with no inline steps, no `--pipeline`, and no `--template`, you MUST use `AskUserQuestion` to gather pipeline configuration.

**Single batched call — all 3 questions at once:**

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Source` | "How would you like to define the pipeline?" | "Built-in template", "Saved pipeline from Obsidian", "Define steps now" |
| 2 | `Template` | "Which template?" (only if Source = "Built-in template") | "Full Sprint (predict → agile → review → ship)", "Quality Audit (security → debug → fix → learn)", "Release Prep (fix → security → learn → ship)", "Deep Analysis (learn → predict → scenario → security)" |
| 3 | `Options` | "Pipeline options?" | "Halt on first error", "Continue on errors (skip failed steps)", "Dry-run only (validate without executing)" |

If user selects "Define steps now", follow up with:

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Steps` | "List the commands to chain (in order). Use arrow notation: predict → agile → ship" | Free text |
| 2 | `Save` | "Save this pipeline for reuse?" | "Yes — save to Obsidian", "No — one-time run" |

## Architecture

```
/autonexus:chain
  ├── Phase 0: Pipeline Resolution (load/parse definition)
  ├── Phase 1: Validation (check all steps are valid commands)
  ├── Phase 2: Pipeline Execution (sequential step runner)
  │     └── Per-step: Validate → Check Condition → Execute → Capture → Log
  ├── Phase 3: Pipeline Summary (results, duration, artifacts)
  └── Phase 4: Persistence (run log to Obsidian, optional save)
```

---

## Pipeline Definition Format

### Inline Syntax

```
/autonexus:chain predict → agile → ship
```

```
/autonexus:chain
Steps:
1. predict --scope "src/**" --depth standard
2. agile --team standard
3. ship --type code-pr --auto
```

### Step Syntax (Full)

Each step can include:

```
{command} [args] [?condition] [!on-failure]
```

- `{command}` — autonexus subcommand name (without `/autonexus:` prefix)
- `[args]` — arguments passed to the command
- `[?condition]` — optional condition: `?on-success` (default), `?on-failure`, `?always`, `?if-metric-above:90`, `?if-metric-below:50`
- `[!on-failure]` — optional failure action: `!skip` (default), `!halt`, `!retry:2`

### Examples

```
# Simple chain
predict → agile → ship

# With arguments
predict --scope "src/**" --depth deep → agile --team standard → ship --auto

# With conditions
predict → debug ?always → fix ?on-failure → agile ?on-success → ship ?on-success

# With failure handling
security !halt → fix !retry:2 → learn → ship
```

### Obsidian Pipeline Definition

Stored at `Projects/{ProjectName}/Pipelines/{name}.md`:

```markdown
---
tags: [autonexus/pipeline]
name: {pipeline-name}
created: {ISO timestamp}
last_run: {ISO timestamp or null}
run_count: {N}
---

# Pipeline: {Human-readable Name}

## Description
{What this pipeline does and when to use it}

## Steps

1. **predict** — `--scope "src/**" --depth standard`
   - Condition: always
   - On failure: skip

2. **agile** — `--team auto`
   - Condition: on-success
   - On failure: halt

3. **review** — `--last --cross-project`
   - Condition: on-success
   - On failure: skip

4. **ship** — `--type code-pr`
   - Condition: on-success
   - On failure: halt

## Variables
- `$SCOPE` = "src/**"
- `$TEAM` = "auto"
```

---

## Built-in Pipeline Templates

### full-sprint

```
predict --depth standard → agile --team auto → review --last → ship --type code-pr
```

**Use when:** Starting a full development cycle from analysis through delivery.

### quality-audit

```
security --depth standard → debug --scope "$SCOPE" → fix → learn --mode update
```

**Use when:** Comprehensive quality check — find vulnerabilities, find bugs, fix everything, update docs.

### release-prep

```
fix → security --depth quick → learn --mode update → ship --dry-run
```

**Use when:** Preparing for a release — clean up errors, security pass, docs update, dry-run the ship.

### deep-analysis

```
learn --mode full → predict --depth deep → scenario --depth deep → security --depth deep
```

**Use when:** Deep understanding before major work — document, get expert opinions, explore edge cases, audit security.

### continuous-quality

```
fix ?always → security ?always !skip → debug ?on-success → review --last ?always
```

**Use when:** Ongoing quality maintenance — fix whatever's broken, audit, hunt remaining bugs, review progress.

---

## Phase 0: Pipeline Resolution

### Step 0a: Resolve Pipeline Source

```
IF --pipeline <name>:
  IF OBSIDIAN_AVAILABLE:
    obsidian_read_note({
      filePath: "Projects/{ProjectName}/Pipelines/{name}.md"
    })
    Parse steps from the "## Steps" section
  ELSE:
    ERROR: "Cannot load pipeline '{name}' — Obsidian is unavailable."

ELIF --template <name>:
  Load the built-in template by name from the templates defined above
  IF name not found: ERROR: "Unknown template '{name}'. Available: full-sprint, quality-audit, release-prep, deep-analysis, continuous-quality"

ELIF --resume <run-id>:
  IF OBSIDIAN_AVAILABLE:
    obsidian_read_note({
      filePath: "Projects/{ProjectName}/Pipelines/runs/{run-id}.md"
    })
    Parse pipeline definition + find last completed step
    SET resume_from = last_completed_step + 1
  ELSE:
    ERROR: "Cannot resume — Obsidian is unavailable."

ELIF inline steps provided:
  Parse inline step definitions (arrow syntax or numbered list)

ELSE:
  Run interactive setup (PREREQUISITE section above)
```

### Step 0b: Parse Steps

For each step in the pipeline definition:

```
FUNCTION parse_step(step_text):
  EXTRACT command_name (first word, strip "autonexus:" prefix if present)
  EXTRACT args (everything between command and condition/failure markers)
  EXTRACT condition (text after "?" — default: "on-success")
  EXTRACT on_failure (text after "!" — default: "skip")
  EXTRACT retry_count (if on_failure starts with "retry:", parse the number)

  VALIDATE command_name is one of the known autonexus subcommands:
    [autonexus, plan, predict, debug, fix, security, ship, scenario,
     learn, setup, resume, status, review, agile, refine-backlog,
     chain, canvas, hook]

  IF command_name == "chain":
    ERROR: "Recursive chain is not allowed. Flatten your pipeline."

  RETURN {
    command: command_name,
    args: args,
    condition: condition,  # on-success | on-failure | always | if-metric-above:N | if-metric-below:N
    on_failure: on_failure,  # skip | halt | retry:N
    retry_count: retry_count or 0
  }
```

---

## Phase 1: Validation

Before execution, validate the entire pipeline:

```
FOR each step in pipeline:
  1. Verify command name is valid
  2. Verify condition syntax is valid
  3. Verify on_failure syntax is valid
  4. Check for circular or recursive chains (no "chain" command in steps)

IF any validation errors:
  Print all errors
  STOP — do not execute

IF --dry-run:
  Print execution plan:
    "Pipeline: {name or 'inline'}"
    "Steps: {count}"
    FOR each step:
      "  {N}. /autonexus:{command} {args} [condition: {condition}] [on-failure: {on_failure}]"
    "Options: halt-on-error={halt_on_error}"
  STOP
```

---

## Phase 2: Pipeline Execution

### Pipeline State

```
pipeline_state = {
  run_id: "{YYYYMMDD}-{HHMM}-chain-{4 random alphanum}",
  pipeline_name: "{name or 'inline'}",
  status: "running",
  started_at: "{ISO timestamp}",
  steps: [...],
  current_step: 0,
  results: [],
  context: {}  # Shared context passed between steps
}
```

### Step Execution Loop

```
FOR step_index, step in enumerate(pipeline.steps):
  # Skip if resuming and step is before resume point
  IF resuming AND step_index < resume_from:
    CONTINUE

  pipeline_state.current_step = step_index

  # --- Condition Check ---
  prev_result = pipeline_state.results[-1] if results else { status: "success" }

  SWITCH step.condition:
    CASE "on-success":
      IF prev_result.status != "success": SKIP step, LOG "skipped (previous step failed)"
    CASE "on-failure":
      IF prev_result.status == "success": SKIP step, LOG "skipped (previous step succeeded)"
    CASE "always":
      PASS (always execute)
    CASE "if-metric-above:N":
      IF pipeline_state.context.last_metric is None OR pipeline_state.context.last_metric <= N:
        SKIP step, LOG "skipped (metric {value} not above {N})"
    CASE "if-metric-below:N":
      IF pipeline_state.context.last_metric is None OR pipeline_state.context.last_metric >= N:
        SKIP step, LOG "skipped (metric {value} not below {N})"

  # --- Execution ---
  PRINT "═══ Pipeline Step {step_index + 1}/{total}: /autonexus:{step.command} ═══"

  attempt = 0
  max_attempts = 1 + step.retry_count

  WHILE attempt < max_attempts:
    attempt += 1
    step_start = now()

    TRY:
      # Execute the autonexus command by invoking the Skill tool
      # Pass step.args as the arguments
      result = Skill(skill="autonexus:{step.command}", args=step.args)

      step_result = {
        step: step_index + 1,
        command: step.command,
        status: "success",
        duration: now() - step_start,
        attempt: attempt,
        summary: "{brief result summary}"
      }
      BREAK  # Success — exit retry loop

    CATCH error:
      IF attempt < max_attempts:
        PRINT "Step {step_index + 1} failed (attempt {attempt}/{max_attempts}). Retrying..."
        CONTINUE
      ELSE:
        step_result = {
          step: step_index + 1,
          command: step.command,
          status: "failed",
          duration: now() - step_start,
          attempt: attempt,
          error: "{error message}",
          summary: "Failed: {error}"
        }

  pipeline_state.results.append(step_result)

  # --- Log Step Result to Obsidian ---
  IF OBSIDIAN_AVAILABLE:
    Update the pipeline run log note with this step's result (see Phase 4)

  # --- Handle Failure ---
  IF step_result.status == "failed":
    SWITCH step.on_failure:
      CASE "halt":
        PRINT "Pipeline halted at step {step_index + 1} (halt-on-error)."
        pipeline_state.status = "halted"
        BREAK out of pipeline loop
      CASE "skip":
        PRINT "Step {step_index + 1} failed. Continuing pipeline..."
        CONTINUE

  # --- Print Step Summary ---
  PRINT "Step {step_index + 1} complete: {step_result.status} ({step_result.duration})"

# After loop completes
IF pipeline_state.status == "running":
  pipeline_state.status = "completed"
```

### Context Passing Between Steps

Steps share context through:

1. **Git state** — Each step operates on the same repo. Changes from step N are visible to step N+1.
2. **Obsidian state** — Notes written by step N are readable by step N+1.
3. **TSV state** — Iteration logs from step N are readable by step N+1.
4. **Pipeline context object** — Captures key outputs:

```
After each step, extract and store in pipeline_state.context:
  - last_metric: the final metric value (if the step produced one)
  - last_session_id: the session ID (if the step created one)
  - last_sprint_id: the sprint number (if agile was run)
  - last_keep_rate: the keep rate (if an iteration loop was run)
  - findings_count: number of findings (if security/debug was run)
  - stories_completed: number of stories done (if agile was run)
```

---

## Phase 3: Pipeline Summary

After all steps complete (or pipeline halts):

```
═══════════════════════════════════════════════════
 Pipeline Complete: {pipeline_name}
═══════════════════════════════════════════════════
 Status: {completed | halted | failed}
 Duration: {total time}
 Steps: {completed}/{total} ({skipped} skipped)

 Step Results:
 ┌────┬────────────┬──────────┬──────────┬──────────────────────────┐
 │ #  │ Command    │ Status   │ Duration │ Summary                  │
 ├────┼────────────┼──────────┼──────────┼──────────────────────────┤
 │ 1  │ predict    │ success  │ 8m 12s   │ 5 personas, 18 findings  │
 │ 2  │ agile      │ success  │ 45m 30s  │ 8/10 stories completed   │
 │ 3  │ review     │ success  │ 2m 45s   │ 65% keep rate, 3 lessons │
 │ 4  │ ship       │ success  │ 3m 10s   │ PR #42 created           │
 └────┴────────────┴──────────┴──────────┴──────────────────────────┘

 Artifacts:
 - Predictions: Projects/{ProjectName}/Predictions/2026-03-22-analysis/
 - Sprint: Projects/{ProjectName}/Sprints/sprint-5/
 - Review: Projects/{ProjectName}/Decisions/2026-03-22-review-{session-id}.md
 - PR: https://github.com/user/repo/pull/42
═══════════════════════════════════════════════════
```

---

## Phase 4: Persistence

### Pipeline Run Log (Obsidian)

Write to `Projects/{ProjectName}/Pipelines/runs/{run-id}.md`:

```markdown
---
tags: [autonexus/pipeline-run, autonexus/{project}]
pipeline: {name or "inline"}
run_id: {run-id}
started: {ISO timestamp}
completed: {ISO timestamp}
status: {completed|halted|failed}
steps_total: {N}
steps_completed: {N}
steps_skipped: {N}
steps_failed: {N}
duration: {total duration}
---

# Pipeline Run: {name}

**Run ID:** {run-id}
**Started:** {timestamp} | **Completed:** {timestamp}
**Status:** {status} | **Duration:** {duration}

## Steps

### Step 1: /autonexus:predict
- **Status:** {success|failed|skipped}
- **Duration:** {duration}
- **Condition:** {condition} → {met|not met}
- **Attempt:** {N}/{max}
- **Summary:** {brief summary}
- **Artifacts:** {links to Obsidian notes created}

### Step 2: /autonexus:agile
- **Status:** {success|failed|skipped}
- **Duration:** {duration}
- **Summary:** {brief summary}

{... repeat for each step}

## Pipeline Context
- **Last Metric:** {value}
- **Last Session:** {session-id}
- **Findings:** {count}
- **Stories Completed:** {count}

## Related
- Pipeline definition: [[Projects/{ProjectName}/Pipelines/{name}]]
```

### Save Pipeline Definition

If `--save <name>` was specified or user chose to save:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Pipelines/{name}.md",
  content: "{rendered pipeline definition template}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Update `last_run` and `run_count` frontmatter after each execution:

```
obsidian_manage_frontmatter({
  filePath: "Projects/{ProjectName}/Pipelines/{name}.md",
  operation: "set",
  key: "last_run",
  value: "{ISO timestamp}"
})
obsidian_manage_frontmatter({
  filePath: "Projects/{ProjectName}/Pipelines/{name}.md",
  operation: "set",
  key: "run_count",
  value: "{incremented count}"
})
```

All Obsidian writes use retry-then-queue protocol.

---

## Listing Saved Pipelines

When `--list` is invoked:

```
IF OBSIDIAN_AVAILABLE:
  obsidian_list_notes({
    dirPath: "Projects/{ProjectName}/Pipelines/",
    recursionDepth: 0
  })

  FOR each .md file (excluding runs/ directory):
    Read frontmatter for name, last_run, run_count
    Add to display table

  PRINT:
  ┌──────────────────┬──────────────────────┬──────┬────────────────────┐
  │ Pipeline         │ Description          │ Runs │ Last Run           │
  ├──────────────────┼──────────────────────┼──────┼────────────────────┤
  │ full-sprint      │ Predict → Agile → …  │ 5    │ 2026-03-20 14:30   │
  │ quality-audit    │ Security → Debug → …  │ 3    │ 2026-03-18 09:15   │
  └──────────────────┴──────────────────────┴──────┴────────────────────┘

  Also list built-in templates (always available):
  Built-in templates: full-sprint, quality-audit, release-prep, deep-analysis, continuous-quality
ELSE:
  PRINT "Obsidian unavailable. Built-in templates available:"
  List the 5 built-in templates with descriptions.
```

---

## Resuming an Interrupted Pipeline

When `--resume <run-id>` is invoked:

```
1. Load the run log from Obsidian
2. Parse each step's status
3. Find the first step that is NOT "success" or "skipped"
4. Set resume_from = that step's index
5. Reload the pipeline definition (from the run log's "pipeline" frontmatter)
6. Print: "Resuming pipeline '{name}' from step {resume_from + 1} ({command})"
7. Execute from that step onward using the normal execution loop
```

---

## Flags Summary

| Flag | Purpose |
|------|---------|
| `--pipeline <name>` | Load a saved pipeline from Obsidian |
| `--template <name>` | Use a built-in pipeline template |
| `--dry-run` | Validate and preview without executing |
| `--halt-on-error` | Stop pipeline on first step failure |
| `--resume <run-id>` | Resume an interrupted pipeline run |
| `--save <name>` | Save the pipeline definition to Obsidian |
| `--list` | List all saved pipelines |

## Error Handling

| Scenario | Response |
|----------|----------|
| Unknown command in step | Validation error — list valid commands, STOP |
| Recursive chain detected | "Recursive chain is not allowed. Flatten your pipeline." STOP |
| Pipeline not found in Obsidian | "Pipeline '{name}' not found. Run `/autonexus:chain --list` to see available pipelines." |
| Run ID not found for resume | "Run '{run-id}' not found in Obsidian." |
| Step fails with halt-on-error | Stop pipeline, mark status as "halted", write run log |
| Obsidian unavailable for --pipeline/--resume/--list | "This operation requires Obsidian. Run /autonexus:setup to configure." |
| Obsidian unavailable for inline/template execution | Proceed — run log written locally, pipeline functions without Obsidian |
| All steps skipped (conditions never met) | Print warning: "All steps were skipped — no conditions were satisfied." |
