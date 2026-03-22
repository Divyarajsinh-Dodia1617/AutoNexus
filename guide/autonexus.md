# /autonexus — Core Autonomous Loop

The main command. Runs an autonomous modify → verify → keep/discard loop with Obsidian reads and writes every iteration.

## Syntax

```
/autonexus
Goal: <what to improve>
Scope: <file globs>
Metric: <what number to track> (<direction>)
Verify: <command that outputs the metric>
Guard: <command that must always pass> (optional)
Iterations: N (optional — default: unlimited)
```

## What Happens

1. **Setup:** Git checks → Obsidian init (connectivity, archive, session) → baseline
2. **Loop:** Review (git + TSV + Obsidian) → Ideate → Modify → Commit → Verify → Guard → Decide → Log (TSV + Obsidian) → Repeat
3. **Session end:** Summary → daily note → queue drain → pre-push rebuild (if pushing)

## Examples

### Test Coverage

```
/autonexus
Goal: Increase test coverage from 72% to 90%
Scope: src/**/*.ts, src/**/*.test.ts
Metric: coverage % (higher is better)
Verify: npx jest --coverage 2>&1 | grep 'All files' | awk '{print $4}'
Guard: npm run typecheck
Iterations: 25
```

### Bundle Size

```
/autonexus
Goal: Reduce bundle size below 200KB
Scope: src/**/*.ts
Metric: bundle size KB (lower is better)
Verify: npx esbuild src/index.ts --bundle --minify | wc -c | awk '{print $1/1024}'
Guard: npm test
```

### API Latency

```
/autonexus
Goal: API p95 latency under 100ms
Scope: src/api/**/*.ts
Metric: p95 latency ms (lower is better)
Verify: npm run bench:api | grep "p95" | awk '{print $2}'
Guard: npm test
Iterations: 15
```

### Python Coverage

```
/autonexus
Goal: Increase pytest coverage to 95%
Scope: src/**/*.py, tests/**/*.py
Metric: coverage % (higher is better)
Verify: pytest --cov=src --cov-report=term 2>&1 | grep TOTAL | awk '{print $4}'
```

## Obsidian Integration per Iteration

| Phase | Obsidian Action | MCP Tool |
|-------|----------------|----------|
| Review (Phase 1) | Search failed approaches for target files | `obsidian_global_search` |
| Review (Phase 1) | Search predict warnings for target files | `obsidian_global_search` |
| Log (Phase 7) | Write 5-8 line iteration note | `obsidian_update_note` |
| Session End | Write session summary | `obsidian_update_note` |
| Session End | Append daily note entry | `obsidian_update_note` (periodicNote) |
| Pre-Push | Rebuild Knowledge Center (3 notes) | `obsidian_update_note` x3 |

All calls use retry-then-queue. If Obsidian is down, loop continues with TSV only.

## Guard Patterns

```
# Tests as guard
Guard: npm test

# Typecheck as guard
Guard: tsc --noEmit

# Combined guard
Guard: npm test && tsc --noEmit

# No guard (metric IS the test)
# Just omit the Guard: line
```

## Tips

- **Start with `/autonexus:plan`** if you don't know your metric
- **Use bounded iterations** for your first run to see how it works
- **Check Obsidian** after the session — the iteration notes tell the story
- **Run `/autonexus:predict` first** for unfamiliar codebases — persona warnings feed into the loop
- **Guard is optional** but recommended for performance optimization (prevents breaking tests)
