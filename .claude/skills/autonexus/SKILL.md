---
name: autonexus
description: Autonomous Goal-directed Iteration with Obsidian Knowledge Backbone. Loops autonomously — modify, verify, keep/discard, repeat — with Obsidian reads/writes every iteration for persistent cross-session knowledge.
version: 0.4.0
---

# AutoNexus — Autonomous Goal-directed Iteration with Obsidian Knowledge Backbone

Inspired by Karpathy's autoresearch. Applies constraint-driven autonomous iteration to ANY work — with Obsidian as the primary knowledge store for decisions, rationale, and codebase understanding.

**Core idea:** You are an autonomous agent. Modify → Verify → Keep/Discard → Write to Obsidian → Repeat.

## MANDATORY: Obsidian MCP Prerequisite

At session start (Phase 0), perform a TWO-STEP check:

**Step A — Tool Existence Check:**
Before calling any Obsidian MCP tool, check whether Obsidian MCP tools are available in your current tool set. Look for tools named `obsidian_read_note`, `obsidian_update_note`, `obsidian_global_search`, `obsidian_list_notes`.

```
IF obsidian MCP tools are NOT in your available tool set:
  SET OBSIDIAN_AVAILABLE = false
  SET OBSIDIAN_NOT_CONFIGURED = true
  WARN once: "Obsidian MCP is not configured. AutoNexus is running in local-only mode.
              The autonomous loop works fully without Obsidian — you get git + TSV tracking.
              To enable persistent knowledge in Obsidian, run /autonexus:setup
              (it will walk you through everything automatically)."
  SKIP all remaining Obsidian Phase 0 steps
  DO NOT attempt any Obsidian MCP calls for the entire session
```

**Step B — Connectivity Check (only if tools exist):**
```
TRY: obsidian_list_notes({ dirPath: "Projects/", recursionDepth: 0 })
SUCCESS → OBSIDIAN_AVAILABLE = true
FAILURE → OBSIDIAN_AVAILABLE = false
  WARN once: "Obsidian MCP tools found but connection failed. Running in local-only mode.
              Check that Obsidian is open and Local REST API plugin is enabled."
```

If unavailable (either step), ALL Obsidian operations become no-ops. The loop runs normally with git + local TSV. AutoNexus is a **fully functional** iteration engine without Obsidian — Obsidian adds persistent knowledge but is never required. Do NOT prompt the user to fix it mid-session.

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
| `/autonexus:debug` | Issue/Symptom, Scope | 4 batched questions per `references/debug-workflow.md` |
| `/autonexus:fix` | Target, Scope | 4 batched questions per `references/fix-workflow.md` |
| `/autonexus:security` | Scope, Depth | 3 batched questions per `references/security-workflow.md` |
| `/autonexus:ship` | What/Type, Mode | 3 batched questions per `references/ship-workflow.md` |
| `/autonexus:scenario` | Scenario, Domain | 4-8 adaptive questions per `references/scenario-workflow.md` |
| `/autonexus:predict` | Scope, Goal, Depth | 3-4 batched questions per `references/predict-workflow.md` |
| `/autonexus:learn` | Mode, Scope | 4 batched questions per `references/learn-workflow.md` |
| `/autonexus:setup` | (none — self-guided wizard) | Walks user through Obsidian MCP config automatically |
| `/autonexus:resume` | (none — session picker) | Presents recent sessions from Obsidian, user picks one |
| `/autonexus:status` | (none — read-only) | No setup needed — queries Obsidian and prints dashboard |
| `/autonexus:review` | (none — session picker) | Presents recent sessions, user picks one for analysis |
| `/autonexus:agile` | Project, Backlog, Team, Sprint Goal | 5 batched questions per `references/agile-workflow.md` |
| `/autonexus:refine-backlog` | Action, Scope | 2 batched questions per `references/refine-backlog-workflow.md` |
| `/autonexus:chain` | Steps or Pipeline name | 3 batched questions per `references/chain-workflow.md` |
| `/autonexus:canvas` | Canvas Type | 2 batched questions per `references/canvas-workflow.md` |
| `/autonexus:hook` | (none for --list; varies for --add) | 3+2 batched questions per `references/hook-workflow.md` |
| `/autonexus:sparc` | Feature/Goal, Scope | 3 batched questions per `references/sparc-methodology.md` |
| `/autonexus:swarm` | Task, Scope, Topology | 4 batched questions per `references/swarm-intelligence.md` |
| `/autonexus:reason` | (none for --stats; varies for --search) | Context-dependent per `references/reasoning-bank.md` |
| `/autonexus:govern` | (none for --list; varies for --add) | 3 batched questions per `references/governance-control.md` |
| `/autonexus:auto-agent` | Task, Scope | 2 batched questions per `references/three-tier-routing.md` |
| `/autonexus:optimize` | (none — read-only) | No setup needed — queries Obsidian optimization data |
| `/autonexus:stream` | Agent chain or Pipeline name | 2 batched questions per `references/stream-chaining.md` |
| `/autonexus:mle-star` | ML Goal, Dataset, Metric | 4 batched questions per `references/reasoning-bank.md` |

**YOU MUST NOT start any loop, phase, or execution without completing interactive setup when context is missing.**

## Subcommands

| Subcommand | Purpose |
|------------|---------|
| `/autonexus` | Run the autonomous loop with Obsidian integration (default) |
| `/autonexus:plan` | Interactive wizard: Goal → Scope, Metric, Direction, Verify config |
| `/autonexus:predict` | Multi-persona swarm prediction with persona notes in Obsidian |
| `/autonexus:debug` | Autonomous bug-hunting loop: scientific method + iterative investigation |
| `/autonexus:fix` | Autonomous fix loop: iteratively repair errors until zero remain |
| `/autonexus:security` | Autonomous security audit: STRIDE + OWASP Top 10 + red-team personas |
| `/autonexus:ship` | Universal shipping workflow: code, content, marketing, sales, research, design |
| `/autonexus:scenario` | Scenario-driven use case generator: explore situations, edge cases, derivatives |
| `/autonexus:learn` | Autonomous documentation engine: scout, learn, generate/update docs, validate |
| `/autonexus:setup` | One-time setup wizard: configure Obsidian MCP (API key, platform detection, .mcp.json) |
| `/autonexus:resume` | Resume a previous session from Obsidian — reload config, pick up from last metric |
| `/autonexus:status` | Read-only project dashboard — session history, knowledge freshness, strategy profile |
| `/autonexus:review` | Post-session analysis — pattern detection, lessons learned, session flow canvas |
| `/autonexus:agile` | Full Agile sprint lifecycle with autonomous agent team |
| `/autonexus:refine-backlog` | Interactive backlog grooming with agent assistance |
| `/autonexus:chain` | Workflow composition — chain multiple commands into a pipeline |
| `/autonexus:canvas` | On-demand Obsidian Canvas generation (architecture, sprint, dependency, etc.) |
| `/autonexus:hook` | Event-driven triggers — define rules that fire actions on events |
| `/autonexus:sparc` | SPARC 5-phase development: Specification → Pseudocode → Architecture → Refinement → Completion |
| `/autonexus:swarm` | Swarm intelligence: multi-agent coordination with queen hierarchy and consensus |
| `/autonexus:reason` | ReasoningBank: view, search, prune learned patterns from self-learning system |
| `/autonexus:govern` | Governance control plane: policies, audit trail, agent permissions |
| `/autonexus:auto-agent` | Auto-scaling agent spawning based on task complexity assessment |
| `/autonexus:optimize` | Token optimization dashboard: cost tracking, routing efficiency, recommendations |
| `/autonexus:stream` | Stream chaining: agent-to-agent output piping without intermediate storage |
| `/autonexus:mle-star` | ML Engineering workflow: Search, Train, Ablate, Refine structured experiment lifecycle |

## When to Activate

- User invokes `/autonexus` → run the loop
- User invokes `/autonexus:plan` → run the planning wizard
- User invokes `/autonexus:predict` → run the predict workflow
- User invokes `/autonexus:debug` → run the debug loop
- User invokes `/autonexus:fix` → run the fix loop
- User invokes `/autonexus:security` → run the security audit
- User invokes `/autonexus:ship` → run the ship workflow
- User invokes `/autonexus:scenario` → run the scenario loop
- User invokes `/autonexus:learn` → run the learn workflow
- User says "help me set up autonexus", "plan a run" → run the planning wizard
- User says "predict", "multi-perspective", "swarm analysis", "what do experts think" → run the predict workflow
- User says "find all bugs", "hunt bugs", "debug this", "why is this failing" → run the debug loop
- User says "fix all errors", "make tests pass", "fix the build", "clean up errors" → run the fix loop
- User says "security audit", "threat model", "OWASP", "STRIDE", "find vulnerabilities" → run the security audit
- User says "ship it", "deploy this", "publish this", "launch this" → run the ship workflow
- User says "explore scenarios", "generate use cases", "what could go wrong", "edge cases" → run the scenario loop
- User says "learn this codebase", "generate docs", "document this project", "update docs" → run the learn workflow
- User invokes `/autonexus:resume` → run the resume workflow
- User invokes `/autonexus:status` → run the status dashboard
- User invokes `/autonexus:review` → run the review workflow
- User invokes `/autonexus:setup` → run the setup wizard
- User says "resume", "continue where I left off", "pick up from last session" → run the resume workflow
- User says "status", "dashboard", "how's the project", "show progress" → run the status dashboard
- User says "review session", "what happened", "analyze last run", "lessons learned" → run the review workflow
- User says "setup obsidian", "configure obsidian", "connect obsidian", "setup autonexus" → run the setup wizard
- User invokes `/autonexus:agile` → run the agile sprint workflow
- User invokes `/autonexus:refine-backlog` → run the backlog refinement workflow
- User says "run a sprint", "agile sprint", "start sprint", "team sprint", "run the team" → run the agile workflow
- User says "refine backlog", "groom stories", "add stories", "update backlog", "create user stories" → run the backlog refinement workflow
- User invokes `/autonexus:chain` → run the chain workflow
- User invokes `/autonexus:canvas` → run the canvas workflow
- User invokes `/autonexus:hook` → run the hook workflow
- User says "chain commands", "run a pipeline", "compose workflows", "run predict then agile" → run the chain workflow
- User says "generate a canvas", "visualize architecture", "draw a diagram", "dependency graph", "sprint board canvas", "knowledge map" → run the canvas workflow
- User says "add a hook", "create a trigger", "list hooks", "when crashes happen", "auto-fix on errors" → run the hook workflow
- User invokes `/autonexus:sparc` → run the SPARC workflow
- User invokes `/autonexus:swarm` → run the swarm workflow
- User invokes `/autonexus:reason` → run the reasoning bank manager
- User invokes `/autonexus:govern` → run the governance control plane
- User invokes `/autonexus:auto-agent` → run auto-scaling agent spawning
- User invokes `/autonexus:optimize` → run the optimization dashboard
- User invokes `/autonexus:stream` → run stream chaining
- User invokes `/autonexus:mle-star` → run ML engineering workflow
- User says "SPARC", "specification to completion", "5-phase development", "build this feature step by step" → run SPARC
- User says "swarm", "multi-agent", "queen coordination", "parallel agents" → run swarm
- User says "reasoning bank", "what patterns", "learned patterns", "self-learning" → run reasoning bank
- User says "governance", "policies", "permissions", "audit trail", "compliance" → run governance
- User says "auto-scale", "how many agents", "auto-assign agents" → run auto-agent
- User says "token usage", "cost optimization", "routing efficiency" → run optimize
- User says "stream agents", "pipe output", "chain agents directly" → run stream
- User says "ML experiment", "train model", "ablation study", "machine learning workflow" → run MLE-STAR
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

See `references/core-principles.md` for the 14 principles (7 from autoresearch + 2 AutoNexus additions + 5 ruflo-inspired additions).

## Adapting to Different Domains

| Domain | Metric | Scope | Verify Command | Guard |
|--------|--------|-------|----------------|-------|
| Backend code | Tests pass + coverage % | `src/**/*.ts` | `npm test` | — |
| Frontend UI | Lighthouse score | `src/components/**` | `npx lighthouse` | `npm test` |
| ML training | val_bpb / loss | `train.py` | `uv run train.py` | — |
| Blog/content | Word count + readability | `content/*.md` | Custom script | — |
| Performance | Benchmark time (ms) | Target files | `npm run bench` | `npm test` |
| Refactoring | Tests pass + LOC reduced | Target module | `npm test && wc -l` | `npm run typecheck` |
| Security | OWASP + STRIDE coverage | API/auth/middleware | `/autonexus:security` | — |
| Shipping | Checklist pass rate (%) | Any artifact | `/autonexus:ship` | Domain-specific |
| Debugging | Bugs found + coverage | Target files | `/autonexus:debug` | — |
| Fixing | Error count (lower) | Target files | `/autonexus:fix` | `npm test` |
| Scenarios | Use cases + edge cases | Feature/domain files | `/autonexus:scenario` | — |
| Prediction | Findings + hypotheses | Target files | `/autonexus:predict` | — |
| Documentation | Validation pass rate | `docs/*.md` | `/autonexus:learn` | `npm test` |
| Agile sprint | Stories completed (%) | Backlog stories | `/autonexus:agile` | Quality gates |

Adapt the loop to your domain. The PRINCIPLES are universal; the METRICS are domain-specific.
