# Plan Workflow — /autonexus:plan

Convert a textual goal into a validated, ready-to-execute AutoNexus configuration.

**Output:** A complete `/autonexus` invocation with Scope, Metric, Direction, and Verify — all validated before launch. Plus a Decision note written to Obsidian documenting the plan configuration.

## Trigger

- User invokes `/autonexus:plan`
- User says "help me set up autonexus", "plan a run", "what should my metric be"

## Workflow

### Phase 1: Capture Goal

**CRITICAL — BLOCKING PREREQUISITE:** If no goal is provided inline, you MUST use `AskUserQuestion` to capture it.

```
AskUserQuestion:
  question: "What do you want to improve? Describe your goal in plain language."
  header: "Goal"
  options:
    - label: "Code quality"
      description: "Tests, coverage, type safety, linting, bundle size"
    - label: "Performance"
      description: "Response time, build speed, Lighthouse score, memory usage"
    - label: "Content"
      description: "SEO score, readability, word count, keyword density"
    - label: "Refactoring"
      description: "Reduce LOC, eliminate patterns, simplify architecture"
```

If user provides goal text directly, skip to Phase 2.

### Phase 2: Analyze Context

1. Read codebase structure (package.json, project files, test config)
2. Identify domain: backend, frontend, ML, content, DevOps, etc.
3. Detect existing tooling: test runner, linter, bundler, benchmark scripts
4. Infer likely metric candidates from goal + tooling

**Obsidian Context (if OBSIDIAN_AVAILABLE):**

5. Query Obsidian Knowledge Center for existing project context:
   ```
   obsidian_read_note({ filePath: "Projects/{ProjectName}/Knowledge/Architecture.md" })
   ```
   If the note exists and is recent (commit_hash matches or within 7 days), use it to:
   - Better understand project structure without re-scanning
   - Suggest more relevant scope/metric based on known components
   - Reference past decisions about this project

6. Query Obsidian for past plan decisions:
   ```
   obsidian_global_search({
     query: "autonexus/decision",
     searchInPath: "Projects/{ProjectName}/Decisions/",
     pageSize: 5
   })
   ```
   If past plans exist, use them to suggest previously successful configurations.

### Phase 3: Define Scope

Present scope options based on codebase analysis:

```
AskUserQuestion:
  question: "Which files should AutoNexus be allowed to modify?"
  header: "Scope"
  options:
    - label: "{inferred_scope_1}"
      description: "{file count} files — {rationale}"
    - label: "{inferred_scope_2}"
      description: "{file count} files — {rationale}"
    - label: "Entire project"
      description: "All source files (use with caution)"
```

**Scope validation rules:**
- Scope must resolve to at least 1 file (run glob, confirm matches)
- Warn if scope exceeds 50 files
- Warn if scope includes test files AND source files

### Phase 4: Define Metric

The metric must be **mechanical** — extractable from a command output as a single number.

```
AskUserQuestion:
  question: "What number tells you if things got better? Pick the mechanical metric."
  header: "Metric"
  options:
    - label: "{metric_1} (Recommended)"
      description: "{what it measures} — extracted via: {command snippet}"
    - label: "{metric_2}"
      description: "{what it measures} — extracted via: {command snippet}"
    - label: "{metric_3}"
      description: "{what it measures} — extracted via: {command snippet}"
```

**Metric validation rules:**

| Check | Pass | Fail |
|-------|------|------|
| Outputs a number | `87.3`, `0.95`, `42` | `PASS`, `looks good` |
| Extractable by command | `grep`, `awk`, `jq` | Requires human judgment |
| Deterministic | Same input → same output | Random, flaky |
| Fast | < 30 seconds | > 2 minutes |

### Phase 4.5: Define Guard (Optional)

```
AskUserQuestion:
  question: "Do you want a guard command? This prevents breaking existing behavior while optimizing."
  header: "Guard"
  options:
    - label: "Yes — run tests as guard (Recommended)"
      description: "{detected_test_command} must pass for every kept change"
    - label: "Yes — custom guard"
      description: "I'll provide my own guard command"
    - label: "No guard needed"
      description: "Skip — the metric is enough"
```

**Guard suggestion rules:**
- Performance/benchmark/bundle → suggest test command as guard
- Refactoring (LOC reduction) → suggest test + typecheck as guard
- Tests ARE the metric → suggest "No guard needed"

### Phase 5: Define Direction

```
AskUserQuestion:
  question: "Is a higher or lower number better for your metric?"
  header: "Direction"
  options:
    - label: "Higher is better"
      description: "Coverage %, score, count of passing tests, throughput"
    - label: "Lower is better"
      description: "Error count, response time, bundle size, LOC"
```

### Phase 6: Define Verify Command

Construct the verification command, then **dry run it.**

**Verify validation (MANDATORY):**

1. Dry run the verify command on current codebase
2. Confirm exit code 0
3. Confirm output contains a parseable number
4. Record baseline metric value
5. If fails → show error, ask user to fix, re-validate

```
Dry run result:
  Exit code: {0 or error}
  Output snippet: {relevant line}
  Extracted metric: {number}
  Baseline: {number}
  Status: VALID / INVALID — {reason}
```

**Do not proceed if verify command fails dry run.**

### Phase 7: Confirm & Launch

Present the complete configuration:

```markdown
## AutoNexus Configuration

**Goal:** {user's goal}
**Scope:** {glob pattern}
**Metric:** {metric name} ({direction})
**Verify:** `{command}`
**Guard:** `{guard_command}` *(or "none")*
**Baseline:** {value from dry run}

### Ready-to-use command:

/autonexus
Goal: {goal}
Scope: {scope}
Metric: {metric} ({direction})
Verify: {verify_command}
Guard: {guard_command}
```

Then ask:

```
AskUserQuestion:
  question: "Configuration validated. How do you want to run it?"
  header: "Launch"
  options:
    - label: "Launch now — unlimited (Recommended)"
      description: "Start /autonexus immediately, loop until interrupted"
    - label: "Launch now — bounded"
      description: "Run a fixed number of iterations"
    - label: "Copy config only"
      description: "Show me the command, I'll run it later"
```

**Obsidian Decision Note (if OBSIDIAN_AVAILABLE):**

After confirmation, write a Decision note to Obsidian documenting the plan:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-plan-{goal-slug}.md",
  content: "---\ntags: [autonexus/decision, autonexus/{project}]\ndate: {YYYY-MM-DD}\nstatus: accepted\niteration: plan\n---\n\n# Decision: AutoNexus Configuration for {goal}\n\n## Context\nUser wants to {goal}. Codebase uses {tech stack}.\n\n## Options Considered\n1. **Scope: {chosen_scope}** — {file count} files. Focused on {rationale}. ← CHOSEN\n2. **Scope: {alt_scope}** — Rejected: {reason}\n\n## Configuration\n- **Goal:** {goal}\n- **Scope:** {scope}\n- **Metric:** {metric} ({direction})\n- **Verify:** `{verify}`\n- **Guard:** `{guard}`\n- **Baseline:** {baseline}\n\n## Evidence\n- Dry run passed: exit code 0, metric = {baseline}\n",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Uses retry-then-queue protocol.

## Metric Suggestion Database

### Code Quality
| Goal Pattern | Metric | Verify Template |
|---|---|---|
| test coverage | Coverage % | `{test_runner} --coverage \| grep "All files"` |
| type safety | `any` count | `grep -r ":\s*any" {scope} --include="*.ts" \| wc -l` |
| lint errors | Error count | `{linter} {scope} 2>&1 \| grep -c "error"` |

### Performance
| Goal Pattern | Metric | Verify Template |
|---|---|---|
| bundle size | Size in KB | `{build_cmd} 2>&1 \| grep "First Load JS"` |
| response time | Time in ms | `{bench_cmd} \| grep "p95"` |
| lighthouse | Score 0-100 | `npx lighthouse {url} --output json --quiet \| jq '.categories.performance.score * 100'` |

### Content
| Goal Pattern | Metric | Verify Template |
|---|---|---|
| readability | Flesch score | `node scripts/readability.js {file}` |
| word count | Word count | `wc -w {scope}` |

### Refactoring
| Goal Pattern | Metric | Verify Template |
|---|---|---|
| reduce LOC | Line count | `{test_cmd} && find {scope} -name "*.ts" \| xargs wc -l \| tail -1` |
| eliminate pattern | Pattern count | `grep -r "{pattern}" {scope} \| wc -l` |

## Anti-Patterns

- **Do NOT accept subjective metrics** — "looks better" is not a metric
- **Do NOT skip the dry run** — always validate verify command works
- **Do NOT suggest verify commands you haven't tested**
- **Do NOT overwhelm with questions** — max 5-6 questions total
- **Do NOT auto-launch without explicit user consent**
