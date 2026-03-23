# /autonexus:stream — Fast Agent-to-Agent Chaining

Pipes agent outputs directly from one agent to the next without Obsidian round-trips. Intermediate results stay in-memory as stream contexts, and only the final agent writes to Obsidian. This is 40-60% faster than standard mode for short chains. Best for quick multi-agent workflows where speed matters more than persistence.

## Syntax

```
# Use a built-in pipeline
/autonexus:stream --pipeline debug

# Run a custom pipeline with a task
/autonexus:stream --pipeline analysis --task "Investigate slow API response times"

# Dry-run to see the pipeline plan
/autonexus:stream --pipeline debug --dry-run

# Verbose mode to see inter-agent context
/autonexus:stream --pipeline analysis --verbose

# Scope to specific files
/autonexus:stream --pipeline debug --scope src/api/**
```

## What It Does

1. **Resolves pipeline** — Loads the named pipeline definition (built-in or custom).
2. **Creates stream context** — Initializes an in-memory context buffer shared across agents.
3. **Executes agents sequentially** — Each agent reads the stream context, performs its work, and appends its output to the stream. No Obsidian reads/writes between steps.
4. **Final agent persists** — Only the last agent in the pipeline writes results to Obsidian and/or git.
5. **Reports** — Prints a summary with per-agent timing and the final result.

## Built-in Pipelines

| Pipeline | Agents | Purpose |
|----------|--------|---------|
| `debug` | Debug → Fix → QA | Find bugs, fix them, verify fixes |
| `analysis` | Learn → Predict → Scenario | Understand codebase, predict risks, explore scenarios |
| `security` | Security → Fix → Security | Audit, fix issues, re-audit to verify |
| `review` | Learn → Review → Fix | Document, analyze, apply improvements |
| `quick-ship` | Fix → QA → Ship | Fix issues, test, ship |

## Flags

| Flag | Purpose |
|------|---------|
| `--pipeline <name>` | Select the pipeline to run (built-in name or custom) |
| `--dry-run` | Show the pipeline plan without executing |
| `--verbose` | Print inter-agent stream contexts during execution |
| `--task <description>` | Provide a task description to guide all agents in the pipeline |
| `--scope <glob>` | Limit all agents in the pipeline to files matching the glob |

## Examples

```bash
# Debug pipeline — find and fix bugs fast
/autonexus:stream --pipeline debug

# Output:
# Stream pipeline: debug (3 agents)
# ─────────────────────────────────
# [1/3] Debug Agent (stream context: 0 → 2.1KB)
#   Found 3 bugs in src/api/handlers.ts
#   Stream: bug reports + stack traces
#   ⏱ 12s
#
# [2/3] Fix Agent (stream context: 2.1KB → 4.8KB)
#   Fixed 3/3 bugs, modified 2 files
#   Stream: diffs + fix explanations
#   ⏱ 18s
#
# [3/3] QA Agent (stream context: 4.8KB → 6.2KB)
#   Wrote 5 regression tests, all passing
#   ✓ Written to Obsidian + committed to git
#   ⏱ 15s
#
# Total: 45s (vs ~78s standard mode — 42% faster)

# Analysis pipeline with a specific task
/autonexus:stream --pipeline analysis --task "Investigate slow API response times"

# Output:
# Stream pipeline: analysis (3 agents)
# ─────────────────────────────────────
# [1/3] Learn Agent (stream context: 0 → 3.4KB)
#   Analyzed API layer: 12 endpoints, 4 middleware, 2 DB connections
#   ⏱ 14s
#
# [2/3] Predict Agent (stream context: 3.4KB → 5.9KB)
#   Identified 3 bottleneck risks: N+1 queries, missing indexes, sync middleware
#   ⏱ 11s
#
# [3/3] Scenario Agent (stream context: 5.9KB → 8.1KB)
#   Explored 5 scenarios: load spike, cold start, concurrent writes, cache miss, timeout cascade
#   ✓ Written to Obsidian
#   ⏱ 16s
#
# Total: 41s (vs ~72s standard mode — 43% faster)

# Dry-run to preview
/autonexus:stream --pipeline debug --dry-run

# Output:
# [DRY RUN] Pipeline: debug
#   Step 1: Debug Agent — find bugs and issues
#   Step 2: Fix Agent — apply fixes from debug output
#   Step 3: QA Agent — write tests and verify (writes to Obsidian)
# Est. time: ~45s | Est. tokens: ~18,000
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Final agent output | Depends on the final agent's normal write behavior | After pipeline completes |
| Pipeline run summary | `Projects/{ProjectName}/Streams/{pipeline}-{timestamp}.md` | After pipeline completes |

### Key Difference from Standard Mode
- **Standard mode:** Each agent reads from and writes to Obsidian. N agents = N read/write cycles.
- **Stream mode:** Only the final agent writes to Obsidian. N agents = 1 write cycle. Intermediate results stay in-memory.

### What Gets Read
- Only the initial context is read from Obsidian (project context, prior sessions)
- Inter-agent context is passed via in-memory stream, not Obsidian

## Without Obsidian

- Pipeline execution works fully without Obsidian — stream contexts are in-memory by design
- Final agent writes to git instead of Obsidian
- No pipeline run summary is persisted
- This is the mode where Obsidian absence matters least

## Tips

- **Use for short chains (2-5 agents).** For longer chains, use `/autonexus:chain` with Obsidian persistence — the overhead of Obsidian round-trips is worth it for debuggability.
- **40-60% faster than standard mode** because there are no Obsidian read/write cycles between agents.
- **`--verbose` is useful for debugging** — it shows exactly what each agent passed to the next, helping you understand unexpected results.
- **`--dry-run` before complex tasks** to verify the pipeline matches your intent.
- **`--scope` improves both speed and accuracy** — smaller scope means less context for each agent to process.
- **The `debug` pipeline is the most common use case** — finding and fixing bugs is a natural fit for fast streaming since you want quick turnaround.
