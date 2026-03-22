# /autonexus:debug — Autonomous Bug Hunting with Obsidian Memory

Scientific method applied to debugging, with every finding persisted to Obsidian. Doesn't stop at one bug — keeps investigating until the codebase is clean or you interrupt.

## Syntax

```
/autonexus:debug
Scope: src/**/*.ts
Symptom: API returns 500 on POST /users

/autonexus:debug --fix --scope src/auth/**
Symptom: JWT validation fails intermittently

/autonexus:debug --severity high --iterations 20
```

## What It Does

7 phases per iteration:

1. **Gather** — collect symptoms, error signals, stack traces
2. **Recon** — map error surface, check Obsidian for past sessions + predict warnings
3. **Hypothesize** — form specific, falsifiable hypothesis
4. **Test** — run minimal experiment to prove/disprove
5. **Classify** — bug confirmed / disproven / inconclusive / new lead
6. **Log** — record to local TSV + write Obsidian iteration note + bug Decision note
7. **Repeat** — next hypothesis, next vector

## Flags

| Flag | Purpose |
|------|---------|
| `--fix` | Auto-chain to `/autonexus:fix` after finding bugs |
| `--scope <glob>` | Limit investigation to specific files |
| `--symptom "<text>"` | Pre-fill symptom (skips interactive setup) |
| `--severity <level>` | Only report at or above: critical, high, medium, low |
| `--technique <name>` | Force technique: binary-search, differential, trace, etc. |
| `--iterations N` | Run exactly N iterations then stop |

## Examples

```bash
# Hunt all bugs in the entire codebase (interactive setup)
/autonexus:debug

# Investigate a specific error
/autonexus:debug --symptom "TypeError: Cannot read property 'id' of undefined"
Scope: src/api/**

# Deep investigation, bounded
/autonexus:debug --iterations 30
Scope: src/auth/**
Symptom: Intermittent 401 errors after token refresh

# Quick scan, then auto-fix what's found
/autonexus:debug --fix --iterations 10
Scope: src/**/*.ts

# Force a specific investigation technique
/autonexus:debug --technique binary-search
Scope: src/renderer/**
Symptom: Render output is wrong but no errors thrown
```

## Obsidian Integration

### What Gets Read (Phase 2 — Recon)

- **Past debug sessions:** Searches `Projects/{ProjectName}/Iterations/` for prior investigations on the same files. Avoids repeating failed hypotheses.
- **Predict warnings:** Searches `Projects/{ProjectName}/Predictions/` for persona warnings (e.g., Security Analyst flagged auth.ts). Prioritizes flagged areas.

### What Gets Written

| When | What | Obsidian Path |
|------|------|---------------|
| Every iteration | Concise iteration note (5-8 lines) | `Iterations/{date}/{session-id}-iter-{N}.md` |
| Each confirmed bug | Full bug Decision note with root cause | `Decisions/{date}-bug-{slug}.md` |
| Session end | Debug session summary | `Iterations/{date}/{session-id}-summary.md` |
| Session end | Daily note mini-summary | Daily Notes (append) |

### Why This Matters

Future debug sessions query Obsidian first. If you already investigated `auth.ts` and disproved "missing token validation," that hypothesis is skipped automatically. Bug Decision notes are searchable by future `/autonexus:fix` runs.

## Chain Patterns

```bash
# Debug then fix (most common workflow)
/autonexus:debug --fix
Scope: src/**

# Debug, review findings, then fix manually
/autonexus:debug --iterations 15
# (review findings.md)
/autonexus:fix --from-debug --guard "npm test"

# Predict then debug (informed investigation)
/autonexus:predict --chain debug
Scope: src/api/**

# Full pipeline: predict -> debug -> fix
/autonexus:predict --depth shallow
/autonexus:debug --fix --iterations 20
```

## Tips

- **Start without flags** to get the interactive setup — it scans your codebase and offers smart defaults.
- **Use `--severity high`** to focus on critical bugs and skip cosmetic issues.
- **Check Obsidian after sessions** — bug Decision notes link back to iteration details and are searchable across projects.
- **The debug -> fix chain** is the most powerful pattern: debug finds root causes, fix applies atomic repairs.
- If Obsidian is unavailable, the loop runs identically — just without knowledge persistence. Local TSV + git continue normally.
