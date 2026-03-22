# /autonexus:fix — Autonomous Error Repair with Obsidian Memory

Iteratively repairs errors until zero remain. One fix per iteration — atomic, committed, verified, auto-reverted on failure. Each fix is logged to Obsidian for cross-session learning.

## Syntax

```
/autonexus:fix
Target: npm run typecheck
Guard: npm test

/autonexus:fix --from-debug --guard "npm test"
Iterations: 30

/autonexus:fix --category test --scope src/auth/**
```

## What It Does

8 phases per iteration:

1. **Detect** — find all errors (tests, types, lint, build) + query Obsidian for recurring issues
2. **Prioritize** — fix order: build > critical bugs > recurring > types > tests > lint > warnings
3. **Fix ONE** — atomic change, check Obsidian for failed approaches first
4. **Commit** — before verification (enables clean rollback)
5. **Verify** — did error count decrease?
6. **Guard** — did anything else break?
7. **Decide** — keep / revert / rework (max 2 rework attempts, then skip)
8. **Log** — local TSV + Obsidian iteration note

Stops automatically when error count hits zero (writes a "clean state" Decision note).

## Flags

| Flag | Purpose |
|------|---------|
| `--target <cmd>` | Explicit verify command (overrides auto-detection) |
| `--guard <cmd>` | Safety command that must always pass |
| `--scope <glob>` | Limit fixes to specific files |
| `--category <type>` | Only fix: test, type, lint, build, or bug |
| `--skip-lint` | Skip lint fixes, focus on functional issues |
| `--from-debug` | Read findings from latest `/autonexus:debug` session |
| `--iterations N` | Run exactly N iterations then stop |

## Examples

```bash
# Fix everything (interactive setup — scans codebase first)
/autonexus:fix

# Fix findings from a debug session
/autonexus:fix --from-debug --guard "npm test"
Iterations: 30

# Fix only type errors, guard with tests
/autonexus:fix --category type --guard "npm test"
Scope: src/**/*.ts

# Fix build failures first, then types
/autonexus:fix --category build
/autonexus:fix --category type --guard "npm run build"

# Bounded sprint — fix as many as possible in 20 iterations
/autonexus:fix --iterations 20
Guard: npm test
```

## Obsidian Integration

### What Gets Read

| Phase | What | Why |
|-------|------|-----|
| Phase 1 (Detect) | Past fix sessions for same errors | Detects recurring issues — errors fixed before that came back |
| Phase 3 (Fix) | Failed approaches on target files | Avoids repeating strategies that were discarded |
| Phase 3 (Fix) | Predict persona warnings | Factors security/reliability concerns into fix approach |

### What Gets Written

| When | What | Obsidian Path |
|------|------|---------------|
| Every iteration | Concise iteration note (5-8 lines) | `Iterations/{date}/{session-id}-iter-{N}.md` |
| Error count = 0 | "Clean State" Decision note | `Decisions/{date}-fix-clean-{slug}.md` |
| Session end | Fix session summary | `Iterations/{date}/{session-id}-summary.md` |
| Session end | Daily note mini-summary | Daily Notes (append) |

### Recurring Issue Detection

When Obsidian shows the same error was fixed in a prior session but reappeared, it gets flagged as "RECURRING" with a priority boost. The fix loop then targets the root cause rather than applying the same surface-level patch.

## Chain Patterns

```bash
# Debug then fix (recommended workflow)
/autonexus:debug --fix
Scope: src/**

# Manual chain: debug -> fix -> ship
/autonexus:debug --iterations 15
/autonexus:fix --from-debug --guard "npm test"
/autonexus:ship

# Predict -> debug -> fix pipeline
/autonexus:predict --depth shallow
/autonexus:debug --fix --iterations 20

# Fix cascade after dependency upgrade
npm upgrade
/autonexus:fix --category type --category test

# Parallel scoped fixes
/autonexus:fix --scope "src/api/**" --category type
/autonexus:fix --scope "src/auth/**" --category test
```

## Tips

- **Start without flags** for interactive setup — it auto-detects test runners, type checkers, and linters, then asks what to fix.
- **Always use `--guard`** to prevent regressions. Fixing a type error should not break a test.
- **Use `--from-debug`** after a debug session to fix exactly what was found, with full root cause context.
- **Check Obsidian after sessions** — the "clean state" Decision note documents exactly what was fixed and how, searchable for future reference.
- **Recurring issues** are the highest-value targets. If Obsidian shows an error keeps coming back, the fix loop prioritizes a deeper architectural fix.
- If Obsidian is unavailable, the loop runs identically — just without cross-session memory. Local TSV + git continue normally.
