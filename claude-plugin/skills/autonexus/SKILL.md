---
name: autonexus
description: Autonomous Goal-directed Iteration with Obsidian Knowledge Backbone. Loops autonomously — modify, verify, keep/discard, repeat — with Obsidian reads/writes every iteration for persistent cross-session knowledge.
version: 0.1.0
---

# AutoNexus — Autonomous Goal-directed Iteration with Obsidian Knowledge Backbone

Inspired by Karpathy's autoresearch. Applies constraint-driven autonomous iteration to ANY work — with Obsidian as the primary knowledge store for decisions, rationale, and codebase understanding.

**Core idea:** You are an autonomous agent. Modify → Verify → Keep/Discard → Write to Obsidian → Repeat.

## MANDATORY: Obsidian MCP Prerequisite

At session start (Phase 0), check Obsidian MCP availability:

```
TRY: obsidian_list_notes({ dirPath: "Projects/", recursionDepth: 0 })
SUCCESS → OBSIDIAN_AVAILABLE = true
FAILURE → OBSIDIAN_AVAILABLE = false
  WARN once: "Obsidian MCP unavailable. Running in local-only mode."
```

If unavailable, ALL Obsidian operations become no-ops. The loop runs normally with git + local TSV. Do NOT prompt the user to fix it mid-session.

## MANDATORY: Interactive Setup Gate

**CRITICAL — READ THIS FIRST BEFORE ANY ACTION:**

For ALL commands:

1. **Check if the user provided ALL required context inline**
2. **If ANY required context is missing → you MUST use `AskUserQuestion` to collect it BEFORE any execution phase.**
3. Each subcommand's reference file has an "Interactive Setup" section — follow it exactly.

| Command | Required Context | If Missing → Ask |
|---------|-----------------|-----------------|
| `/autonexus` | Goal, Scope, Metric, Direction, Verify | Batch 1 (4 questions) + Batch 2 (3 questions) from Setup Phase below |
| `/autonexus:plan` | Goal | Ask via `AskUserQuestion` per `references/plan-workflow.md` |
| `/autonexus:predict` | Scope, Goal, Depth | 3-4 batched questions per `references/predict-workflow.md` |

**YOU MUST NOT start any loop, phase, or execution without completing interactive setup when context is missing.**

## Subcommands

| Subcommand | Purpose |
|------------|---------|
| `/autonexus` | Run the autonomous loop with Obsidian integration (default) |
| `/autonexus:plan` | Interactive wizard: Goal → Scope, Metric, Direction, Verify config |
| `/autonexus:predict` | Multi-persona swarm prediction with persona notes in Obsidian |

### Phase 2 (Future)

| Subcommand | Purpose | Status |
|------------|---------|--------|
| `/autonexus:debug` | Autonomous bug-hunting loop | Planned |
| `/autonexus:fix` | Autonomous error repair loop | Planned |
| `/autonexus:security` | STRIDE + OWASP security audit | Planned |
| `/autonexus:ship` | Universal shipping workflow | Planned |
| `/autonexus:scenario` | Scenario-driven use case generator | Planned |
| `/autonexus:learn` | Autonomous documentation engine | Planned |

## When to Activate

- User invokes `/autonexus` → run the loop
- User invokes `/autonexus:plan` → run the planning wizard
- User invokes `/autonexus:predict` → run the predict workflow
- User says "help me set up autonexus", "plan a run" → run the planning wizard
- User says "predict", "multi-perspective", "swarm analysis", "what do experts think" → run the predict workflow
- User says "work autonomously", "iterate until done", "keep improving", "run overnight" → run the loop
- Any task requiring repeated iteration cycles with measurable outcomes → run the loop

## Bounded Iterations

By default, AutoNexus loops **forever** until manually interrupted. To run exactly N iterations, add `Iterations: N`.

**Unlimited (default):**
```
/autonexus
Goal: Increase test coverage to 90%
```

**Bounded (N iterations):**
```
/autonexus
Goal: Increase test coverage to 90%
Iterations: 25
```

After N iterations, print final summary with baseline → current best.

## Setup Phase (Do Once)

**If the user provides Goal, Scope, Metric, and Verify inline** → extract them and proceed to step 5.

**If ANY critical field is missing → MUST use `AskUserQuestion` to collect them interactively.**

### Interactive Setup (when invoked without full config)

Scan the codebase for smart defaults, then ask ALL questions in batched calls.

**Batch 1 — Core config (4 questions):**

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Goal` | "What do you want to improve?" | "Test coverage (higher)", "Bundle size (lower)", "Performance (faster)", "Code quality (fewer errors)" |
| 2 | `Scope` | "Which files can AutoNexus modify?" | Suggested globs from project structure |
| 3 | `Metric` | "What number tells you if it got better?" | Detected options from tooling |
| 4 | `Direction` | "Higher or lower is better?" | "Higher is better", "Lower is better" |

**Batch 2 — Verify + Guard + Launch (3 questions):**

| # | Header | Question | Options |
|---|--------|----------|---------|
| 5 | `Verify` | "What command produces the metric?" | Suggested commands from detected tooling |
| 6 | `Guard` | "Any command that must ALWAYS pass?" | "npm test", "tsc --noEmit", "Skip — no guard" |
| 7 | `Launch` | "Ready to go?" | "Launch (unlimited)", "Launch with iteration limit", "Edit config", "Cancel" |

**After Batch 2:** Dry-run the verify command. If fails, ask user to fix.

### Setup Steps (after config is complete)

1. **Read all in-scope files** for full context
2. **Define the goal** — from user input or inline config
3. **Define scope constraints** — validated file globs
4. **Define guard (optional)** — regression prevention command
5. **Create results log** — `autonexus-results.tsv` (see `references/results-logging.md`)
6. **Obsidian initialization** — connectivity check, archive old sessions, create session note (see `references/obsidian-integration.md`)
7. **Establish baseline** — run verification + guard. Record as iteration #0
8. **Confirm and go** — show setup, begin the loop

## The Loop

Read `references/autonomous-loop-protocol.md` for full protocol details.
Read `references/obsidian-integration.md` for Obsidian MCP integration details.

```
LOOP (FOREVER or N times):
  1. Review: Read state + git history + TSV + Obsidian (failed approaches + predict warnings)
  2. Ideate: Pick next change (informed by git, TSV, AND Obsidian context)
  3. Modify: Make ONE focused change to in-scope files
  4. Commit: Git commit (before verification)
  5. Verify: Run mechanical metric
  6. Guard: If guard set, run guard command
  7. Decide:
     - IMPROVED + guard passed → Keep commit
     - IMPROVED + guard FAILED → Revert, rework (max 2 attempts)
     - SAME/WORSE → Git revert
     - CRASHED → Fix (max 3 attempts) or move on
  8. Log: Write TSV (always) + Obsidian iteration note (best-effort)
  9. Repeat:
     - Unbounded: NEVER STOP. NEVER ASK "should I continue?"
     - Bounded: Stop after N iterations, run session end protocol

SESSION END:
  - Write session summary to Obsidian
  - Append daily note mini-summary
  - Drain obsidian-sync-queue.md
  - If user pushes: run pre-push knowledge rebuild
```

## Critical Rules

1. **Loop until done** — Unbounded: forever. Bounded: N times then summarize.
2. **Read before write** — Always understand full context before modifying
3. **One change per iteration** — Atomic changes. If it breaks, you know why
4. **Mechanical verification only** — No subjective "looks good." Use metrics
5. **Automatic rollback** — Failed changes revert instantly
6. **Simplicity wins** — Equal results + less code = KEEP
7. **Git is memory** — Every experiment committed with `experiment:` prefix. Use `git revert` for rollbacks. Read `git log` before each iteration
8. **When stuck, think harder** — Re-read files, re-read goal, combine near-misses, query Obsidian for cross-session patterns
9. **Obsidian is best-effort** — NEVER block the loop on an Obsidian failure. Queue and continue. Local TSV is always the mechanical truth.
10. **Iteration notes are concise** — 5-8 lines. Not a novel. Capture what, result, why, and next.

## Principles Reference

See `references/core-principles.md` for the 9 principles (7 from autoresearch + 2 AutoNexus additions).

## Adapting to Different Domains

| Domain | Metric | Scope | Verify Command | Guard |
|--------|--------|-------|----------------|-------|
| Backend code | Tests pass + coverage % | `src/**/*.ts` | `npm test` | — |
| Frontend UI | Lighthouse score | `src/components/**` | `npx lighthouse` | `npm test` |
| ML training | val_bpb / loss | `train.py` | `uv run train.py` | — |
| Blog/content | Word count + readability | `content/*.md` | Custom script | — |
| Performance | Benchmark time (ms) | Target files | `npm run bench` | `npm test` |
| Refactoring | Tests pass + LOC reduced | Target module | `npm test && wc -l` | `npm run typecheck` |
| Prediction | Findings + hypotheses | Target files | `/autonexus:predict` | — |

Adapt the loop to your domain. The PRINCIPLES are universal; the METRICS are domain-specific.
