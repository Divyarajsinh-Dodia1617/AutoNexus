# /autonexus:predict — Multi-Persona Swarm Prediction

Before you optimize, debug, or ship — get 3-8 expert perspectives in minutes. Each persona's analysis becomes a permanent Obsidian note.

## Syntax

```
/autonexus:predict
Scope: src/**/*.ts
Goal: Find reliability issues

/autonexus:predict --depth shallow --chain debug
Scope: src/api/**

/autonexus:predict --depth deep --adversarial
Goal: Pre-deployment quality audit
```

## What It Does

8 phases:

1. **Setup** — validate config
2. **Reconnaissance** — scan codebase, build knowledge files (check Obsidian cache first)
3. **Persona Generation** — 3-8 expert personas from codebase context
4. **Independent Analysis** — each persona analyzes independently
5. **Debate** — 1-3 rounds of cross-examination
6. **Consensus** — voting + anti-herd detection
7. **Report** — local files + **Obsidian persona notes + debate transcript**
8. **Handoff** — optional chain to debug/security/fix/ship/scenario

## Default Personas (5)

| Persona | Focus | Bias |
|---------|-------|------|
| Architecture Reviewer | Scalability, coupling, design | Skeptical of god objects |
| Security Analyst | OWASP Top 10, injection, auth | Assumes hostile inputs |
| Performance Engineer | Complexity, N+1, memory | Prefers evidence |
| Reliability Engineer | Error handling, race conditions | Assumes failure |
| Devil's Advocate | Challenges consensus | MUST challenge >=50% of majority |

## Adversarial Personas (`--adversarial`)

| Persona | Focus |
|---------|-------|
| Red Team Attacker | Exploitation paths |
| Blue Team Defender | Detection gaps |
| Insider Threat | Data exfiltration |
| Supply Chain Analyst | Dependency risks |
| Judge | Evaluates claims |

## Depth Presets

| Depth | Personas | Rounds |
|-------|----------|--------|
| Shallow | 3 | 1 |
| Standard | 5 | 2 |
| Deep | 8 | 3 |

## Obsidian Integration

### What Gets Written

For a standard-depth run (5 personas):
- **5 persona notes** — one per persona with all findings, severity, evidence, recommendations
- **1 debate note** — full transcript with challenges, revised positions, consensus table, anti-herd status

Path: `Projects/{ProjectName}/Predictions/{YYYY-MM-DD}-{slug}/`

### Knowledge Center Cache

Before scanning the full codebase, predict checks Obsidian Knowledge Center:
- If `Architecture.md` exists and `commit_hash` matches current HEAD → skip full scan, use cached knowledge
- If stale → incremental update (only re-analyze changed files)
- If missing → full scan, then write knowledge files to Obsidian

### Searchable by Future Iterations

When `/autonexus` runs, it queries `Predictions/` for warnings about files being modified. If the Security Analyst flagged `auth.ts:33` and the loop is about to modify `auth.ts`, the iteration gets a warning: "Security Analyst flagged JWT handling — proceed with caution."

## Chain Patterns

```
# Predict then debug
/autonexus:predict --chain debug
Scope: src/auth/**

# Predict then security audit
/autonexus:predict --chain security --adversarial
Scope: src/api/**

# Full pipeline: predict → scenario → debug → fix
/autonexus:predict --chain scenario,debug,fix
Scope: src/**
Goal: Full quality pipeline
```

## Flags

| Flag | Purpose |
|------|---------|
| `--scope <glob>` | Files to analyze |
| `--goal <text>` | Focus area |
| `--depth <level>` | shallow / standard / deep |
| `--personas <N>` | Override count (3-8) |
| `--rounds <N>` | Override rounds (1-3) |
| `--adversarial` | Red-team persona set |
| `--chain <tools>` | Chain downstream (comma-separated) |
| `--budget <N>` | Max findings (default: 40) |
| `--fail-on <severity>` | Exit non-zero for CI/CD |
| `--incremental` | Reuse existing knowledge files |

## Anti-Herd Detection

If personas converge too quickly (flip_rate > 0.8 AND entropy < 0.3):
- All minority findings are preserved
- Warning flagged in report
- Suggestion to re-run with `--adversarial`

## Tips

- **Start with `--depth shallow`** for quick scans (~2 min)
- **Use `--adversarial`** for security-sensitive code
- **Chain with `--chain debug`** to empirically test predictions
- **Check Obsidian persona notes** — they're richer than the summary
- Empirical evidence from loops ALWAYS overrides predict consensus
