# Results Logging Protocol — AutoNexus

Track every iteration in a structured log. Enables pattern recognition and prevents repeating failed experiments.

AutoNexus uses dual-write: local TSV (mechanical source of truth) + Obsidian iteration notes (persistent knowledge with rationale).

## Setup & Initialization

AutoNexus creates the log automatically at Phase 0 (baseline):

```bash
# 1. Create log file with metric direction and header
echo "# metric_direction: higher_is_better" > autonexus-results.tsv
echo -e "iteration\tcommit\tmetric\tdelta\tguard\tstatus\tdescription" >> autonexus-results.tsv

# 2. Add to .gitignore (log is local, not committed)
echo "autonexus-results.tsv" >> .gitignore

# 3. Run verify command to establish baseline metric
BASELINE=$(run_verify_command)

# 4. Record baseline as iteration 0
COMMIT=$(git rev-parse --short HEAD)
echo -e "0\t${COMMIT}\t${BASELINE}\t0.0\tpass\tbaseline\tinitial state" >> autonexus-results.tsv
```

## Log Format (TSV)

Create `autonexus-results.tsv` in the working directory (gitignored):

```tsv
iteration	commit	metric	delta	guard	status	description
```

### Columns

| Column | Type | Description |
|--------|------|-------------|
| iteration | int | Sequential counter starting at 0 (baseline) |
| commit | string | Short git hash (7 chars), "-" if reverted |
| metric | float | Measured value from verification |
| delta | float | Change from previous best (negative = improved for "lower is better") |
| guard | enum | `pass`, `fail`, or `-` (no guard configured) |
| status | enum | `baseline`, `keep`, `keep (reworked)`, `discard`, `crash`, `no-op`, `hook-blocked` |
| description | string | One-sentence description of what was tried |

### Example

```tsv
iteration	commit	metric	delta	guard	status	description
0	a1b2c3d	85.2	0.0	pass	baseline	initial state — test coverage 85.2%
1	b2c3d4e	87.1	+1.9	pass	keep	add tests for auth middleware edge cases
2	-	86.5	-0.6	-	discard	refactor test helpers (broke 2 tests)
3	-	0.0	0.0	-	crash	add integration tests (DB connection failed)
4	-	88.9	+1.8	fail	discard	inline hot-path functions (guard: 3 tests broke)
5	c3d4e5f	88.3	+1.2	pass	keep	add tests for error handling in API routes
```

## Logging Function

Called at Phase 7 of every iteration after the keep/discard/crash decision:

```bash
log_iteration() {
  local iteration=$1 commit=$2 metric=$3 delta=$4 guard=$5 status=$6 description=$7
  echo -e "${iteration}\t${commit}\t${metric}\t${delta}\t${guard}\t${status}\t${description}" \
    >> autonexus-results.tsv
}
```

## Dual-Write: TSV + Obsidian

After writing the TSV entry, write a concise Obsidian iteration note per the `obsidian-integration.md` protocol:

1. **TSV write** — ALWAYS happens, even if Obsidian is unavailable. This is the mechanical source of truth.
2. **Obsidian write** — Best-effort via retry-then-queue protocol. Adds rationale, file list, and "next suggestion" that the TSV doesn't capture.

The TSV and Obsidian note contain overlapping but complementary data:

| Data | TSV | Obsidian Note |
|------|-----|--------------|
| Iteration number | Yes | Yes (frontmatter) |
| Commit hash | Yes | Yes (frontmatter) |
| Metric value | Yes | Yes |
| Delta | Yes | Yes |
| Status | Yes | Yes (frontmatter) |
| Description | Yes (1 sentence) | Yes (1 sentence) |
| Rationale (WHY) | No | Yes |
| Files modified | No | Yes |
| What to try next | No | Yes |
| Tags | No | Yes (frontmatter) |

## Reading & Using the Log

```bash
# Phase 1 (Review): Read recent entries for pattern recognition
tail -20 autonexus-results.tsv

# Count outcomes for progress tracking
KEEPS=$(grep -c 'keep' autonexus-results.tsv || echo 0)
DISCARDS=$(grep -c 'discard' autonexus-results.tsv || echo 0)
CRASHES=$(grep -c 'crash' autonexus-results.tsv || echo 0)

# Detect stuck state: >5 consecutive discards triggers recovery
LAST_5=$(tail -5 autonexus-results.tsv | awk -F'\t' '{print $6}')

# Pattern recognition: which changes succeed?
grep 'keep' autonexus-results.tsv | awk -F'\t' '{print $7}'
```

## Integration with the AutoNexus Loop

```
Phase 0 (Setup):    → CREATE log file, record baseline (iteration 0)
                    → CREATE baseline Obsidian note (if available)
Phase 1 (Review):   → READ last 10-20 TSV entries for pattern recognition
                    → READ Obsidian for failed approaches + predict warnings
Phase 3-6 (Loop):   → Modify, Commit, Verify, Decide
Phase 7 (Log):      → APPEND TSV row (always)
                    → WRITE Obsidian iteration note (best-effort)
Phase 8 (Repeat):   → Back to Phase 1
Session End:        → WRITE Obsidian session summary + daily note entry
```

## Summary Reporting

Every 10 iterations (or at loop completion in bounded mode), print a brief summary:

```
=== AutoNexus Progress (iteration 20) ===
Baseline: 85.2% → Current best: 92.1% (+6.9%)
Keeps: 8 | Discards: 10 | Crashes: 2
Last 5: keep, discard, discard, keep, keep
```

## Metric Direction

Clarify at setup whether lower or higher is better:
- **Lower is better:** val_bpb, response time (ms), bundle size (KB), error count
- **Higher is better:** test coverage (%), lighthouse score, throughput (req/s)

Record direction in first line of results log:
```
# metric_direction: higher_is_better
```
