# AutoNexus

**Autonomous goal-directed iteration engine with Obsidian knowledge backbone — swarm intelligence, self-learning, SPARC methodology, and 60+ autonomous agents.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [Claude Autoresearch](https://github.com/uditgoenka/autoresearch). AutoNexus adds persistent cross-session knowledge via Obsidian MCP — every iteration reads from and writes to your Obsidian vault. Inspired by [ruflo](https://github.com/ruvnet/ruflo)'s swarm intelligence and self-learning patterns, v0.4.0 orchestrates **60+ specialized agents** through swarm coordination, SPARC development, and full Agile sprints — with a self-improving ReasoningBank that makes every session smarter than the last.

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blue?logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-0.4.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

*"Set the GOAL → Claude runs the LOOP → Knowledge persists in Obsidian → You wake up to results AND understanding."*
*"Deploy the SWARM → Queen decomposes → Workers execute in parallel → Consensus synthesizes."*
*"SPARC builds FEATURES → ReasoningBank LEARNS → Governance GUARDS → The system evolves itself."*

---

## Table of Contents

- [Start Here — Pick Your Command](#start-here--pick-your-command)
- [Quick Start](#quick-start)
- [Command Reference](#command-reference)
- [Best Strategies for Maximum Value](#best-strategies-for-maximum-value)
- [How It Works](#how-it-works)
- [Agent Team (60+ Agents)](#agent-team-60-agents)
- [Obsidian Vault Structure](#obsidian-vault-structure)
- [Feature Reference by Version](#feature-reference-by-version)
- [FAQ](#faq)
- [Credits](#credits)

---

## Start Here — Pick Your Command

**Don't know which command to use?** Find your situation below.

### "I want to improve something measurable"

| Situation | Command | Example |
|-----------|---------|---------|
| Improve a metric (coverage, perf, size) | [`/autonexus`](#autonexus--core-autonomous-loop) | `Goal: Increase test coverage from 72% to 90%` |
| Don't know what metric to use | [`/autonexus:plan`](#autonexusplan--goal-configuration-wizard) | Walks you through Goal → Scope → Metric → Verify |
| Resume where you left off yesterday | [`/autonexus:resume`](#autonexusresume--resume-previous-session) | Reloads everything from Obsidian |

### "I want to build features"

| Situation | Command | Example |
|-----------|---------|---------|
| Build a feature with design-first rigor | [`/autonexus:sparc`](#autonexussparc--sparc-5-phase-development) | Spec → Pseudocode → Architecture → Refine → Complete |
| Run a full Agile sprint (10 stories) | [`/autonexus:agile`](#autonexusagile--full-agile-sprint) | Sprint planning → execution → review → retro |
| Ship anything (code, content, release) | [`/autonexus:ship`](#autonexusship--universal-shipping-workflow) | 8-phase workflow from draft to delivery |
| Complex multi-domain task | [`/autonexus:swarm`](#autonexusswarm--multi-agent-swarm) | Queen decomposes → workers execute in parallel |

### "I want to find and fix problems"

| Situation | Command | Example |
|-----------|---------|---------|
| Hunt ALL bugs in a codebase | [`/autonexus:debug`](#autonexusdebug--autonomous-bug-hunting) | Scientific method, finds all bugs not just one |
| Fix all errors (tests, types, lint) | [`/autonexus:fix`](#autonexusfix--autonomous-error-repair) | One fix per iteration until zero errors remain |
| Security audit (STRIDE + OWASP) | [`/autonexus:security`](#autonexussecurity--security-audit) | Threat model + red-team with 4 adversarial personas |

### "I want to analyze, learn, or plan"

| Situation | Command | Example |
|-----------|---------|---------|
| Get expert opinions before starting | [`/autonexus:predict`](#autonexuspredict--multi-persona-prediction) | 5+ expert personas debate your plan |
| Explore edge cases for a feature | [`/autonexus:scenario`](#autonexusscenario--scenario-generator) | Generates situations across 12 dimensions |
| Generate docs for a codebase | [`/autonexus:learn`](#autonexuslearn--documentation-engine) | Scout → learn → generate with validation loop |
| Check project health at a glance | [`/autonexus:status`](#autonexusstatus--project-dashboard) | Read-only dashboard, no modifications |
| Analyze what happened in a session | [`/autonexus:review`](#autonexusreview--post-session-analysis) | Patterns, lessons learned, turning points |
| Run an ML experiment lifecycle | [`/autonexus:mle-star`](#autonexusmle-star--ml-engineering) | Search → Train → Ablate → Refine → Deploy |

### "I want to automate and orchestrate"

| Situation | Command | Example |
|-----------|---------|---------|
| Chain commands into a pipeline | [`/autonexus:chain`](#autonexuschain--workflow-composition) | `predict → agile → review → ship` |
| Fast agent-to-agent piping | [`/autonexus:stream`](#autonexusstream--stream-chaining) | 40-60% faster than Obsidian handoffs |
| Auto-react when events happen | [`/autonexus:hook`](#autonexushook--event-driven-triggers) | "When 3 crashes, auto-run debug" |
| Auto-pick optimal team size | [`/autonexus:auto-agent`](#autonexusauto-agent--auto-scaling-teams) | Complexity assessment → team composition |
| Visualize architecture or sprint | [`/autonexus:canvas`](#autonexuscanvas--visual-diagrams) | 7 canvas types for Obsidian |

### "I want to manage and optimize"

| Situation | Command | Example |
|-----------|---------|---------|
| Set safety guardrails for agents | [`/autonexus:govern`](#autonexusgovern--governance-control-plane) | Policies, permissions, audit trail |
| See what patterns the system learned | [`/autonexus:reason`](#autonexusreason--reasoningbank-management) | View, search, prune learned strategies |
| Check token spending | [`/autonexus:optimize`](#autonexusoptimize--token-optimization) | Cost tracking, routing efficiency |
| Groom the product backlog | [`/autonexus:refine-backlog`](#autonexusrefine-backlog--backlog-grooming) | Create, split, reprioritize stories |
| First-time Obsidian setup | [`/autonexus:setup`](#autonexussetup--obsidian-setup-wizard) | One-time MCP configuration wizard |

---

## Quick Start

### 1. Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [Obsidian](https://obsidian.md) desktop app (optional — works without it)
- [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) Obsidian community plugin enabled

### 2. Install AutoNexus

**Option A — Plugin marketplace (recommended):**

```
/plugin marketplace add Divyarajsinh-Dodia1617/AutoNexus
/plugin install autonexus@autonexus
/reload-plugins
```

That's it. All 26 commands and 60+ agents are available immediately.

**Updating:**
```
/plugin update autonexus@autonexus
/reload-plugins
```

**Option B — Manual copy:**

```bash
git clone https://github.com/Divyarajsinh-Dodia1617/AutoNexus.git

# Copy to your project
cp -r AutoNexus/claude-plugin/skills/autonexus .claude/skills/autonexus
cp -r AutoNexus/claude-plugin/commands/autonexus .claude/commands/autonexus
cp AutoNexus/claude-plugin/commands/autonexus.md .claude/commands/autonexus.md
```

### 3. Configure Obsidian MCP

Run the setup wizard (easiest):
```
/autonexus:setup
```

Or manually add to your `.mcp.json`:

```json
{
  "mcpServers": {
    "obsidian-mcp-server": {
      "command": "npx",
      "args": ["-y", "obsidian-mcp-server"],
      "env": {
        "OBSIDIAN_API_KEY": "<your API key from Local REST API plugin>",
        "OBSIDIAN_BASE_URL": "https://127.0.0.1:27124",
        "OBSIDIAN_VERIFY_SSL": "false",
        "OBSIDIAN_ENABLE_CACHE": "true"
      }
    }
  }
}
```

> **Obsidian is optional.** AutoNexus works fully without it — you just don't get persistent knowledge. Run `/autonexus:setup` anytime to enable it.

### 4. Your First Run

```
/autonexus
Goal: Increase test coverage from 72% to 90%
Scope: src/**/*.ts
Metric: coverage % (higher is better)
Verify: npm test -- --coverage | grep "All files"
```

### 5. Your First Sprint

```bash
# Step 1: Build your backlog
/autonexus:refine-backlog --new "User authentication with JWT"
/autonexus:refine-backlog --new "User preferences API"
/autonexus:refine-backlog --new "Settings dashboard"

# Step 2: Run a sprint
/autonexus:agile
Sprint Goal: Core user management features
Team: standard
```

---

## Command Reference

Every command explained with purpose, syntax, key flags, and real examples.

---

### `/autonexus` — Core Autonomous Loop

**Purpose:** Runs an autonomous modify → verify → keep/discard loop. The foundational command. Every iteration reads/writes Obsidian for persistent cross-session knowledge.

**When to use:** Any time you have a measurable goal — test coverage, performance, bundle size, code quality score, error count.

**Syntax:**
```
/autonexus
Goal: <what to improve>
Scope: <file globs>
Metric: <what to measure> (higher/lower is better)
Verify: <command that outputs the metric>
Guard: <command that must always pass> (optional)
Iterations: N (optional, default: unlimited)
```

**Examples:**
```
# Performance optimization
/autonexus
Goal: Reduce API response time from 450ms to under 200ms
Scope: src/api/**/*.ts
Metric: p95 latency ms (lower is better)
Verify: npm run bench | grep "p95"
Guard: npm test

# Bundle size reduction
/autonexus
Goal: Reduce bundle size from 2.1MB to under 1MB
Scope: src/**/*.{ts,tsx}
Metric: bundle size KB (lower is better)
Verify: npm run build 2>&1 | grep "Total size"
Guard: npm run typecheck
Iterations: 30
```

**What happens each iteration:**
1. **Review** — reads git log, TSV, and queries Obsidian for failed approaches and predict warnings
2. **Ideate** — picks a strategy informed by ReasoningBank patterns and past failures
3. **Modify** — makes ONE atomic change
4. **Commit** — `experiment:` prefix commit
5. **Verify** — runs your metric command
6. **Decide** — keep if metric improved (and guard passes), otherwise revert
7. **Log** — writes to TSV (always) + Obsidian note (best-effort)

**Tips:**
- Start with bounded iterations (e.g., `Iterations: 15`) on your first run to understand behavior
- Use `Guard` for performance work to prevent regressions (e.g., tests must still pass)
- If you don't know your metric, start with `/autonexus:plan` instead
- After 5+ sessions, the ReasoningBank significantly improves keep rates

---

### `/autonexus:plan` — Goal Configuration Wizard

**Purpose:** Converts a plain-language goal into a validated, executable configuration. Outputs the exact Scope, Metric, Direction, and Verify command.

**When to use:** When you know WHAT you want but not HOW to measure it.

**Syntax:**
```
/autonexus:plan
Goal: <plain-language description>
```

**Example:**
```
/autonexus:plan
Goal: Make the app faster
```

Outputs something like:
```
Scope: src/api/**/*.ts, src/middleware/**/*.ts
Metric: p95 response time ms (lower is better)
Verify: npm run bench -- --reporter json | jq '.p95'
Guard: npm test
```

You can then copy-paste this directly into `/autonexus`.

---

### `/autonexus:agile` — Full Agile Sprint

**Purpose:** Runs a complete Agile sprint lifecycle with autonomous agents — planning, execution of 10 stories, sprint review, and retrospective.

**When to use:** When you have a backlog of stories and want structured team-based delivery with design → implement → review → QA → accept per story.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `Team:` | Team size configuration | `lean`, `standard`, `full`, `auto` |
| `--phase` | Run only one ceremony | `--phase planning`, `--phase review`, `--phase retro` |
| `--resume` | Resume interrupted sprint | Picks up from last completed step |

**Example:**
```
/autonexus:agile
Sprint Goal: Core user management features
Team: standard
```

**What happens:**
1. **Sprint Planning** — PM selects stories, Architect assesses, 3 parallel estimators do Planning Poker, EM assigns, CTO reviews
2. **Execution (10 stories)** — For each: Architect designs → Engineer implements (mini autoresearch loop) → Sr. Engineer reviews → QA tests → PM accepts
3. **Sprint Review** — PM acceptance, CEO business value assessment
4. **Retrospective** — Metrics, agent reflections, action items
5. Everything persists in Obsidian (sprint board, designs, reviews, QA logs, retro)

**Agent spawn count:** ~112 for a clean sprint, ~125 with rework.

---

### `/autonexus:refine-backlog` — Backlog Grooming

**Purpose:** Interactive backlog management with PM + Architect agent assistance. Create, refine, reprioritize, split, or archive stories.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--new` | Create a new story | `--new "JWT authentication"` |
| `--refine` | Refine existing story | `--refine STORY-42` |
| `--split` | Split large story | `--split STORY-42` |
| `--prioritize` | Reprioritize backlog | `--prioritize` |
| `--archive` | Archive completed stories | `--archive` |

**Example:**
```
/autonexus:refine-backlog --new "User preferences API"
/autonexus:refine-backlog --split STORY-15
/autonexus:refine-backlog --prioritize
```

---

### `/autonexus:predict` — Multi-Persona Prediction

**Purpose:** Assembles 5+ expert personas who independently analyze your plan, then debate. Each persona's analysis becomes a permanent, searchable Obsidian note.

**When to use:** Before starting major work. Get diverse expert perspectives to surface risks and opportunities you might miss.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--chain` | Continue into another command after | `--chain debug`, `--chain agile` |
| `--personas` | Specify custom personas | `--personas "security,perf,ux"` |

**Example:**
```
/autonexus:predict
What: We're migrating from REST to GraphQL
Context: 200+ endpoints, 50k daily users, 3 frontend apps
```

**What happens:**
1. Assembles personas (e.g., Security Architect, Performance Engineer, API Designer, Migration Specialist, End-User Advocate)
2. Each independently analyzes risks, benefits, and recommendations
3. Debate phase — personas challenge each other
4. Consensus report with scored risks
5. Everything saved to Obsidian (persona notes + debate canvas)

---

### `/autonexus:debug` — Autonomous Bug Hunting

**Purpose:** Scientific-method bug hunting loop. Unlike typical debugging, it finds ALL bugs — not just one. Each finding becomes a searchable Obsidian note.

**When to use:** When you suspect multiple bugs, or want a systematic audit of a codebase area.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--fix` | Auto-fix found bugs after | Chains into `/autonexus:fix` |
| `--scope` | Limit search area | `--scope src/auth/**` |

**Example:**
```
/autonexus:debug
Target: Authentication module
Symptoms: Intermittent 401 errors on valid tokens
Scope: src/auth/**, src/middleware/auth.*
```

**What happens per iteration:**
1. **Hypothesize** — form a testable theory based on symptoms and past findings
2. **Instrument** — add targeted logging/assertions
3. **Test** — run against reproduction case
4. **Analyze** — confirm or reject hypothesis
5. **Document** — write finding to Obsidian with severity, root cause, suggested fix
6. **Continue** — move to next potential bug

---

### `/autonexus:fix` — Autonomous Error Repair

**Purpose:** Fix ALL errors until zero remain. One fix per iteration, atomic, auto-reverted on failure.

**When to use:** When you have known errors (test failures, type errors, lint violations) and want them all fixed automatically.

**Syntax:**
```
/autonexus:fix
Errors: <command that shows errors>
Scope: <file globs>
```

**Example:**
```
/autonexus:fix
Errors: npm test 2>&1 | grep "FAIL"
Scope: src/**/*.ts

# Or for type errors
/autonexus:fix
Errors: npx tsc --noEmit 2>&1
Scope: src/**/*.ts
```

**How it works:** Runs the error command, picks one error, fixes it atomically, verifies the fix didn't break anything else, commits. Repeats until zero errors.

---

### `/autonexus:security` — Security Audit

**Purpose:** Comprehensive security audit using STRIDE threat modeling + OWASP Top 10 checklist + red-team testing with 4 adversarial personas.

**When to use:** Before releases, after major changes, or as part of a regular security cadence.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--scope` | Limit audit area | `--scope src/api/**` |
| `--fix` | Auto-remediate findings | Chains into fix loop |

**Example:**
```
/autonexus:security
Target: Payment processing module
Scope: src/payments/**, src/api/billing/**
```

**What happens:**
1. **STRIDE analysis** — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
2. **OWASP Top 10 check** — Injection, Broken Auth, Sensitive Data Exposure, etc.
3. **Red-team phase** — 4 adversarial personas attempt to find exploits
4. **Finding report** — Severity-scored findings with remediation suggestions
5. Everything saved to Obsidian Security folder

---

### `/autonexus:ship` — Universal Shipping Workflow

**Purpose:** 8-phase structured shipping workflow. Works for code, content, marketing materials, research papers, sales materials — anything that goes from draft to delivery.

**When to use:** When you need a structured workflow to go from "done coding" to "actually shipped."

**Example:**
```
/autonexus:ship
What: User authentication feature
Type: code
Target: Production via PR
```

**8 phases:** Prepare → Draft → Review → Test → Stage → Approve → Ship → Verify

---

### `/autonexus:scenario` — Scenario Generator

**Purpose:** Explores situations, edge cases, and derivative scenarios from a seed scenario across 12 dimensions. Uses autonomous iteration with Obsidian persistence.

**When to use:** When designing a feature and you need to think through all the ways users might interact with it.

**Example:**
```
/autonexus:scenario
Seed: User uploads a profile picture
Context: Mobile app, 10M users, CDN delivery
```

**Dimensions explored:** Happy path, error states, edge cases, concurrency, security, performance, accessibility, internationalization, data integrity, degraded mode, admin actions, integration points.

---

### `/autonexus:learn` — Documentation Engine

**Purpose:** Autonomous documentation generation. Scouts the codebase, learns patterns, generates/updates docs with a validation-fix loop, then syncs to Obsidian Knowledge Center.

**When to use:** When documentation is stale or nonexistent and you want comprehensive, accurate docs generated automatically.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--scope` | Limit doc scope | `--scope src/api/**` |
| `--update` | Update existing docs | Only regenerate stale sections |

**Example:**
```
/autonexus:learn
Target: Full API documentation
Scope: src/api/**, src/middleware/**
```

---

### `/autonexus:sparc` — SPARC 5-Phase Development

**Purpose:** Structured feature development through 5 rigorous phases with dedicated agents and quality gates between each phase.

**When to use:** For medium-to-large features where you'd normally write a design doc. More structured than the basic loop, less overhead than a full Agile sprint.

**Syntax:**
```
/autonexus:sparc
Feature: <what to build>
Scope: <file globs>
```

**Example:**
```
/autonexus:sparc
Feature: Real-time collaborative editing with conflict resolution
Scope: src/collab/**, src/api/ws/**
```

**5 Phases:**

| Phase | Agent | What happens | Gate |
|-------|-------|-------------|------|
| **1. Specification** | Specification Agent | Requirements, acceptance criteria, constraints | Completeness check |
| **2. Pseudocode** | Pseudocode Agent | Algorithm design, data flow, API contracts | Logic review |
| **3. Architecture** | Architecture Agent (opus) + CTO Review | System design, ADRs, component diagram | CTO approval |
| **4. Refinement** | Refinement Agent | Mini autoresearch loop until all acceptance criteria pass | All tests pass |
| **5. Completion** | Completion Agent | Documentation, cleanup, integration verification | Full verification |

**Tips:**
- SPARC is best when the feature has clear acceptance criteria
- For quick fixes, use `/autonexus:fix` instead
- For simple metric improvement, use `/autonexus` instead
- The CTO review at Phase 3 catches architectural issues early

---

### `/autonexus:swarm` — Multi-Agent Swarm

**Purpose:** Deploy a queen-led multi-agent swarm for complex tasks that exceed single-agent capacity. Queen decomposes the goal, workers execute in parallel, consensus synthesizes results.

**When to use:**
- Task spans multiple domains (e.g., refactor auth across API + frontend + tests)
- Single-agent loop stuck after 8+ consecutive discards
- Need fault tolerance through redundancy

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--topology` | Set swarm topology | `hierarchical`, `mesh`, `ring`, `star`, `adaptive` |
| `--consensus` | Consensus algorithm | `raft`, `byzantine`, `gossip`, `crdt`, `quorum` |
| `--workers` | Max worker count | `--workers 8` |

**Example:**
```
/autonexus:swarm
Goal: Migrate authentication from session-based to JWT across all services
Scope: src/**
Topology: hierarchical
Consensus: raft
```

**5 Topologies:**

| Topology | Best For | Drift Risk |
|----------|----------|------------|
| **Hierarchical** (default) | Coding tasks, clear decomposition | Lowest |
| **Mesh** | Research, exploration, brainstorming | Medium |
| **Hierarchical-Mesh** | Complex multi-domain (queen → mesh clusters) | Low |
| **Ring** | Pipeline processing, sequential transforms | Very low |
| **Star** | Independent parallel tasks | Low |

**5 Consensus Algorithms:**

| Algorithm | Best For |
|-----------|----------|
| **Raft** (default) | Authoritative coding decisions |
| **Byzantine** | When agents produce conflicting outputs |
| **Gossip** | Large swarms (10+ workers), knowledge sharing |
| **CRDT** | Concurrent edits to shared state |
| **Quorum** | Tunable consistency requirements |

**Anti-drift:** Goal alignment checks every 3 worker completions. Out-of-scope changes auto-revert. If alignment drops below 70%, swarm pauses and re-aligns.

---

### `/autonexus:stream` — Stream Chaining

**Purpose:** Pipe one agent's output directly into the next agent's input — no Obsidian round-trip. 40-60% faster than standard handoffs.

**When to use:** When speed matters more than persistence. Each agent returns a structured summary (max 500 tokens) that feeds the next.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--pipeline` | Use predefined pipeline | `analysis`, `refactor`, `debug`, `optimize`, `sparc` |
| `--agents` | Custom agent chain | `--agents "security→debug→fix"` |

**Example:**
```
# Use a predefined pipeline
/autonexus:stream --pipeline analysis

# Custom chain
/autonexus:stream --agents "security→debug→fix"
Target: src/api/**
```

**Predefined pipelines:**

| Pipeline | Agents | Purpose |
|----------|--------|---------|
| `analysis` | Security → Debug → Review | Full codebase analysis |
| `refactor` | Architect → Implement → Review | Structured refactoring |
| `debug` | Debug → Fix → Verify | Find and fix bugs fast |
| `optimize` | Profile → Optimize → Benchmark | Performance tuning |
| `sparc` | Spec → Pseudo → Arch → Refine → Complete | Full SPARC flow |

**Trade-off:** Faster but less resumable. Only the final agent persists to Obsidian. Use `/autonexus:chain` when you need full persistence and resumability.

---

### `/autonexus:chain` — Workflow Composition

**Purpose:** Chain multiple autonexus commands into sequential pipelines with conditions, failure handling, and Obsidian-persisted run logs.

**When to use:** When your workflow requires multiple commands in sequence (e.g., predict → sprint → review → ship) and you want persistence, resumability, and conditional logic.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--template` | Use built-in template | `full-sprint`, `quality-audit`, `release-prep` |
| `--save` | Save pipeline to Obsidian | `--save my-pipeline` |
| `--pipeline` | Load saved pipeline | `--pipeline my-pipeline` |
| `--list` | List saved pipelines | Shows all defined pipelines |
| `--resume` | Resume interrupted run | From last completed step |

**Inline syntax:**
```
/autonexus:chain predict → agile → review → ship
```

**With conditions:**
```
/autonexus:chain security ?on-success→ ship ?on-failure→ fix !retry:2 → security
```

**Condition types:**
- `?on-success` — run only if previous step succeeded
- `?on-failure` — run only if previous step failed
- `?always` — run regardless
- `?if-metric-above:N` — conditional on metric value

**Failure handling:**
- `!halt` — stop pipeline (default)
- `!skip` — skip this step, continue
- `!retry:N` — retry N times before halting

**5 Built-in Templates:**

| Template | Pipeline | Purpose |
|----------|----------|---------|
| `full-sprint` | predict → agile → review → ship | Complete sprint with prediction |
| `quality-audit` | security → debug → fix → learn | Full quality pass |
| `release-prep` | fix → security → learn → ship | Pre-release checklist |
| `deep-analysis` | predict → scenario → debug → review | Thorough analysis |
| `continuous-quality` | debug → fix → security → review | Ongoing quality loop |

**Example:**
```
# Use a template
/autonexus:chain --template quality-audit

# Save a custom pipeline
/autonexus:chain predict → sparc → security → ship --save feature-pipeline

# Reuse it later
/autonexus:chain --pipeline feature-pipeline
```

---

### `/autonexus:hook` — Event-Driven Triggers

**Purpose:** Define "when X happens, do Y" rules that fire automatically during loops and sprints. Set them up once, they work across sessions.

**When to use:** When you want automatic reactions to events (e.g., auto-debug on crashes, auto-fix on guard failures, pause when keep rate drops).

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--add` | Create a new hook | Define event → condition → action |
| `--template` | Use built-in template | 7 templates available |
| `--list` | List all hooks | Shows enabled/disabled status |
| `--enable` / `--disable` | Toggle a hook | `--enable auto-debug` |
| `--remove` | Delete a hook | `--remove hook-name` |
| `--history` | View trigger history | When hooks fired and results |
| `--dry-run` | Test without executing | See what would fire |

**12 Event Types:**
`iteration-complete`, `consecutive-discards`, `consecutive-crashes`, `keep-rate-drop`, `metric-threshold`, `goal-achieved`, `session-end`, `finding-created`, `story-complete`, `gate-failed`, `sprint-end`, `pipeline-step-failed`

**7 Built-in Templates:**

| Template | Event → Action |
|----------|---------------|
| `auto-debug-on-crashes` | 3 consecutive crashes → run debug |
| `backlog-from-findings` | New finding → create backlog story |
| `pause-on-low-keep-rate` | Keep rate < 20% → pause loop |
| `celebrate-goal` | Goal achieved → create celebration note |
| `auto-fix-on-guard-fail` | Guard failure → run fix |
| `auto-review-on-session-end` | Session ends → run review |
| `escalate-gate-failures` | Gate fails 2x → escalate to supervisor |

**Example:**
```
# Apply a template
/autonexus:hook --template auto-debug-on-crashes

# Custom hook
/autonexus:hook --add
Event: consecutive-discards
Condition: count >= 5
Action: run-command /autonexus:swarm
Priority: 1
```

---

### `/autonexus:auto-agent` — Auto-Scaling Teams

**Purpose:** Automatically determines optimal team size and composition based on task complexity. Scores your task across 5 dimensions and routes to the right tier.

**When to use:** When you're not sure whether to use the basic loop, SPARC, swarm, or a full sprint.

**Example:**
```
/autonexus:auto-agent
Task: Add real-time notifications across web and mobile
Scope: src/**
```

**Complexity scoring (0-10 each):**
- **File breadth** — how many files affected?
- **Domain depth** — how deep is the domain knowledge needed?
- **Novelty** — has this been done before in this codebase?
- **Risk** — what's the blast radius of failure?
- **Reasoning depth** — how complex are the decisions?

**Routing:**
| Score | Tier | Action |
|-------|------|--------|
| 0-5 | Tier 1 (Direct) | Simple edit, zero tokens |
| 6-20 | Tier 2 (Sonnet) | Standard loop or SPARC |
| 21-50 | Tier 3 (Opus/Swarm) | Full swarm or sprint |

---

### `/autonexus:canvas` — Visual Diagrams

**Purpose:** Generate Obsidian `.canvas` files for visual project understanding. 7 canvas types available.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--type` | Canvas type | `architecture`, `sprint`, `dependency`, `timeline`, `knowledge`, `flow`, `pipeline` |

**7 Canvas Types:**

| Type | Shows | Best For |
|------|-------|----------|
| `architecture` | Component relationships, color-coded by risk | Understanding system structure |
| `sprint` | Kanban board with story cards | Sprint visibility |
| `dependency` | Import/dependency graph | Finding coupling |
| `timeline` | Session history with keep-rate colors | Tracking progress over time |
| `knowledge` | All Obsidian notes with backlinks | Navigating knowledge base |
| `flow` | Entry points → processors → stores → outputs | Understanding data flow |
| `pipeline` | Pipeline steps with status colors | Visualizing workflows |

**Example:**
```
/autonexus:canvas --type architecture
/autonexus:canvas --type sprint
```

---

### `/autonexus:reason` — ReasoningBank Management

**Purpose:** View, search, prune, and analyze the learned patterns that make AutoNexus smarter over time. The ReasoningBank is the self-learning system — it stores what worked and what didn't.

**When to use:** To understand what strategies the system has learned, prune stale patterns, or check confidence scores.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--stats` | Show learning statistics | Keep rates, top strategies, confidence |
| `--search` | Search patterns | `--search "test coverage"` |
| `--prune` | Remove low-confidence patterns | Auto-removes below threshold |
| `--export` | Export for transfer learning | Share patterns across projects |
| `--import` | Import from another project | At 50% initial confidence |

**Example:**
```
/autonexus:reason --stats
/autonexus:reason --search "performance optimization"
/autonexus:reason --prune --threshold 0.3
```

**How it works:**
- **RETRIEVE** — Before ideating, search past patterns matching current context
- **JUDGE** — If confidence > 0.5 and context similarity > 70%, recommend the strategy
- **DISTILL** — After each iteration, store new patterns (keeps) or negative evidence (discards)
- **Decay** — Patterns lose 5% confidence/week without re-validation. Below 0.3 = flagged. Below 0.1 = auto-archived
- **Meta-learning** — Every 10 iterations, opus-tier analyst reviews what's working

---

### `/autonexus:govern` — Governance Control Plane

**Purpose:** Manage policies, agent permissions, audit trail, and compliance. Ensures agents only access what they should.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--template` | Apply built-in policy | `secret-protection`, `branch-protection`, `scope-guard` |
| `--audit` | View audit trail | All policy evaluations |
| `--violations` | View violations | Blocked actions with details |
| `--policies` | List active policies | Shows all rules |

**7 Built-in Policies (always active):**
1. Never modify `.env`, `*.key`, `*.pem`, `credentials.*`
2. Never `git push --force` to main/master
3. Never `rm -rf` outside project
4. Max 10 files per atomic change
5. Agent tasks timeout after 5 minutes
6. Scope enforcement (approved file globs only)
7. Cost limits (token budget per session/sprint)

**4 Security Levels:**

| Level | Agents | Permissions |
|-------|--------|-------------|
| **Minimal** | Junior (Tier 9) | Read in-scope only, verify commands only |
| **Standard** | Mid/Senior (Tier 7-8) | Project read, in-scope write, standard commands |
| **Elevated** | Leads/Architects (Tier 5-6) | All read/write, all commands, spawn agents |
| **Admin** | Executive (Tier 1-3) | Unrestricted, time-limited (30 min expiry) |

**Example:**
```
/autonexus:govern --template secret-protection
/autonexus:govern --audit
/autonexus:govern --violations
```

---

### `/autonexus:optimize` — Token Optimization

**Purpose:** Dashboard showing token spending, routing efficiency, cache hit rates, and optimization recommendations.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--dashboard` | Full cost dashboard | Spending, routing, savings |
| `--recommendations` | Optimization tips | Based on your usage patterns |

**Example:**
```
/autonexus:optimize --dashboard
```

**What it tracks:**
- Token usage per session/sprint
- Three-tier routing decisions (how many tasks went to Tier 1/2/3)
- Cache hit rates
- Estimated savings vs. all-Opus routing
- Recommendations for improving efficiency

---

### `/autonexus:mle-star` — ML Engineering

**Purpose:** Structured ML experiment lifecycle — Search, Train, Ablate, Refine, Ensemble, Validate, Deploy.

**When to use:** When running ML experiments that benefit from systematic methodology.

**Example:**
```
/autonexus:mle-star
Task: Improve sentiment classifier accuracy from 82% to 92%
Dataset: data/reviews.csv
Metric: F1 score (higher is better)
```

**7 Phases:**

| Phase | What happens |
|-------|-------------|
| **Search** | Literature search, SOTA discovery |
| **Train** | Baseline training with tracked experiments |
| **Ablate** | Systematic component removal to identify what matters |
| **Refine** | Core autoresearch loop on best configuration |
| **Ensemble** | Combine top variants (averaging, stacking, voting) |
| **Validate** | Hold-out validation, statistical significance tests |
| **Deploy** | Model packaging, serving config, monitoring setup |

---

### `/autonexus:resume` — Resume Previous Session

**Purpose:** Reload goal, scope, metric config from a past session and continue iterating from current codebase state.

**Example:**
```
/autonexus:resume
```
Lists recent sessions and lets you pick which to resume. Links new session to the resumed one in Obsidian.

---

### `/autonexus:status` — Project Dashboard

**Purpose:** Read-only dashboard. Makes no modifications. Shows session history, iteration stats, keep rate, strategy effectiveness, Knowledge Center freshness, and unresolved findings.

**Example:**
```
/autonexus:status
```

---

### `/autonexus:review` — Post-Session Analysis

**Purpose:** Analyzes what worked and what failed in past sessions. Finds turning points, diminishing returns, and pattern insights. Writes Lessons Learned to Obsidian.

**Key flags:**
| Flag | Purpose | Example |
|------|---------|---------|
| `--cross-project` | Generate cross-project insights | Patterns that apply everywhere |
| `--canvas` | Generate session flow canvas | Visual timeline of iterations |

**Example:**
```
/autonexus:review
/autonexus:review --cross-project --canvas
```

---

### `/autonexus:setup` — Obsidian Setup Wizard

**Purpose:** One-time setup. Gets your Obsidian API key, detects your platform, writes `.mcp.json`, verifies the connection.

**Example:**
```
/autonexus:setup
```

---

## Best Strategies for Maximum Value

### Level 1: Getting Started (First Day)

**Goal:** Learn the basics and see results.

```
# 1. Set up Obsidian (one-time, optional but recommended)
/autonexus:setup

# 2. Plan your first goal
/autonexus:plan
Goal: Improve test coverage

# 3. Run with bounded iterations
/autonexus
Goal: Increase test coverage from 72% to 85%
Scope: src/**/*.ts
Metric: coverage % (higher is better)
Verify: npm test -- --coverage | grep "All files"
Iterations: 15

# 4. Review what happened
/autonexus:review
```

**Key insight:** Start bounded (15-25 iterations). Watch the TSV and Obsidian notes to understand the loop's behavior before going unbounded.

---

### Level 2: Building Confidence (First Week)

**Goal:** Use the right command for each situation and build up your knowledge base.

**The "predict-first" habit:**
```
# Before any major work, get expert opinions
/autonexus:predict
What: We're adding WebSocket support for real-time features
Context: Express.js API, React frontend, 10k concurrent users

# Then act on the recommendations
/autonexus:sparc
Feature: WebSocket-based real-time notifications
```

**The "fix-first" habit:**
```
# Start each session by cleaning up
/autonexus:fix
Errors: npm test 2>&1 | grep "FAIL"
Scope: src/**/*.ts

# Then work on new features from a clean slate
/autonexus:sparc
Feature: ...
```

**Use debug + fix together:**
```
/autonexus:debug --fix
Target: Payment processing module
Scope: src/payments/**
```

---

### Level 3: Automation (First Month)

**Goal:** Set up hooks and chains so the system does more with less intervention.

**Set up defensive hooks:**
```
# Auto-debug when things crash repeatedly
/autonexus:hook --template auto-debug-on-crashes

# Pause if keep rate drops (you're going in circles)
/autonexus:hook --template pause-on-low-keep-rate

# Auto-fix guard failures
/autonexus:hook --template auto-fix-on-guard-fail

# Create backlog stories from findings
/autonexus:hook --template backlog-from-findings
```

**Create reusable pipelines:**
```
# Save a feature development pipeline
/autonexus:chain predict → sparc → security → review → ship --save feature-pipeline

# Save a quality audit pipeline
/autonexus:chain security → debug → fix → learn → review --save quality-check

# Reuse across sessions
/autonexus:chain --pipeline feature-pipeline
/autonexus:chain --pipeline quality-check
```

**Use templates for common workflows:**
```
/autonexus:chain --template full-sprint     # predict → agile → review → ship
/autonexus:chain --template quality-audit    # security → debug → fix → learn
/autonexus:chain --template release-prep     # fix → security → learn → ship
```

---

### Level 4: Swarm Mastery (Month 2+)

**Goal:** Use multi-agent orchestration for complex tasks.

**When to escalate from loop to swarm:**
- Regular loop stuck after 8+ consecutive discards? → Swarm
- Task spans 3+ domains (API + frontend + infra)? → Swarm
- Need parallel exploration of different approaches? → Swarm

**Choose the right topology:**
```
# Standard coding task — hierarchical (default, safest)
/autonexus:swarm
Goal: Refactor database layer to use connection pooling
Topology: hierarchical

# Research/exploration — mesh (more creative)
/autonexus:swarm
Goal: Find the best caching strategy for our API
Topology: mesh

# Pipeline processing — ring (sequential, lowest drift)
/autonexus:swarm
Goal: Migrate data from MongoDB to PostgreSQL
Topology: ring

# Independent parallel tasks — star (fastest)
/autonexus:swarm
Goal: Add unit tests for all API endpoints
Topology: star
```

**Use auto-agent when unsure:**
```
# Let the system decide the right approach
/autonexus:auto-agent
Task: Add OAuth2 with Google and GitHub providers
Scope: src/**
```

---

### Level 5: Self-Learning Optimization (Month 3+)

**Goal:** The system learns YOUR codebase and gets better every session.

**Check what the system has learned:**
```
/autonexus:reason --stats
```

**After 5+ sessions, you should see:**
- Keep rates improving 15-25%
- Top strategies emerging for your codebase
- Negative patterns being avoided automatically

**Transfer learning across projects:**
```
# Export patterns from a mature project
/autonexus:reason --export project-a-patterns

# Import into a new project (starts at 50% confidence)
/autonexus:reason --import project-a-patterns
```

**Prune stale patterns:**
```
# Remove patterns below 0.3 confidence
/autonexus:reason --prune --threshold 0.3
```

**Monitor costs:**
```
/autonexus:optimize --dashboard
```

Typical savings with three-tier routing: 30-50% vs. all-Opus execution.

---

### Recommended Command Combinations

These are the most effective multi-command workflows:

| Workflow | Commands | When |
|----------|----------|------|
| **Feature Development** | `plan → predict → sparc → security → ship` | New feature from scratch |
| **Bug Bash** | `debug → fix → review` | Known quality issues |
| **Sprint Delivery** | `refine-backlog → predict → agile → review` | Delivering a batch of stories |
| **Release Prep** | `fix → security → learn → ship` | Shipping to production |
| **Deep Investigation** | `predict → scenario → debug → review` | Understanding a complex problem |
| **Performance Tuning** | `predict → autonexus (with perf metric) → review` | Optimizing speed/size |
| **Security Hardening** | `security → fix → security → govern` | Hardening a module |
| **Codebase Onboarding** | `learn → status → canvas --type architecture` | New team member |
| **ML Experiment** | `predict → mle-star → review` | Machine learning workflow |
| **Swarm for Complex Tasks** | `predict → auto-agent → swarm → review` | Multi-domain changes |

---

### Anti-Patterns to Avoid

| Anti-Pattern | Why It Fails | Do This Instead |
|-------------|-------------|-----------------|
| Unbounded iterations on first run | You won't understand the loop's behavior | Start with `Iterations: 15` |
| Skipping `/autonexus:plan` | Bad metrics = bad results | Plan first if you're unsure about your metric |
| Using swarm for simple tasks | Overhead outweighs benefit | Use the basic loop for single-domain work |
| No guard command | Performance work can break tests | Always set `Guard: npm test` |
| Ignoring predict warnings | Personas surface risks you'd miss | Run predict before major changes |
| Never reviewing sessions | You miss patterns and diminishing returns | Run `/autonexus:review` after every major session |
| Not setting up hooks | You manually intervene on every crash | Set up `auto-debug-on-crashes` at minimum |
| Running SPARC for tiny fixes | 5-phase overhead for a one-line fix | Use `/autonexus:fix` for small errors |

---

## Why AutoNexus?

Autoresearch loops capture WHAT changed in git and metrics in TSV. But they lose **WHY** — the rationale, the alternatives considered, the failed approaches, the cross-project patterns.

AutoNexus adds an Obsidian knowledge backbone:

| Without Obsidian | With AutoNexus |
|-----------------|----------------|
| New session starts from scratch | Session reads past decisions + failed approaches from Obsidian |
| Same failed approach retried | Agent queries: "this was tried and discarded in iteration 12" |
| Lessons siloed per project | Cross-project patterns inform new work |
| Rationale lost after squash/rebase | Decisions survive all git operations |
| Predict persona insights are ephemeral | Persona notes persist, searchable via backlinks |

---

## How It Works

```
EVERY ITERATION:
  1. Review: git log + TSV + QUERY OBSIDIAN (failed approaches, predict warnings)
  2. Ideate: Informed by persistent knowledge — avoid past failures
  3. Modify: ONE atomic change
  4. Commit: experiment: prefix
  5. Verify: Mechanical metric
  6. Decide: Keep/revert
  7. Log: TSV (always) + Obsidian note (best-effort, retry-then-queue)
  8. Repeat

SESSION END: Summary note + daily note entry + sync queue drain
PRE-PUSH: Full knowledge center rebuild (~2-5 min)
```

### Core Principles

1. **Constraint = Enabler** — Bounded scope, fixed iteration cost, single success metric
2. **Separate Strategy from Tactics** — Humans set direction; agents execute
3. **Metrics Must Be Mechanical** — No subjective "looks good"
4. **Verification Must Be Fast** — Enables rapid iteration
5. **Git as Memory** — Every change committed with causality tracking
6. **Knowledge Persists Beyond Code** — Obsidian captures WHY
7. **Obsidian Is Best-Effort** — Never blocks the loop
8. **Swarm Intelligence Over Solo** — Multiple perspectives > sequential bottleneck
9. **Least Privilege by Default** — Every agent gets minimum permissions
10. **Route to Cheapest Capable Handler** — Three-tier routing saves 30-50%
11. **Self-Learning Through RETRIEVE-JUDGE-DISTILL** — Gets smarter every session
12. **Anti-Drift by Design** — Structural prevention, not reactive monitoring

---

## Agent Team (60+ Agents)

### Core Engineering Team (28 agents)
```
CEO ─── CTO ─── Principal Engineer ─── Sr. Staff Engineer ─── Staff Engineer
 │       │              │
 │       │       Solutions Architect ─── Security Architect
 │       │
 │       ├── Engineering Manager
 │       │       ├── Sr. Backend Engineer ── Backend Engineer ── Jr. Backend Engineer
 │       │       ├── Sr. Frontend Engineer ── Frontend Engineer ── Jr. Frontend Engineer
 │       │       ├── Sr. Full Stack Engineer ── Full Stack Engineer
 │       │       ├── Sr. DevOps Engineer
 │       │       └── Sr. SRE
 │       │
 │       └── Sr. Security Engineer ── Security Architect
 │
CPO ─── Product Manager ── Technical Program Manager ── Scrum Master
 │
 └────── UX Designer, UI Designer, QA Manager, Sr. QA Engineer, QA Engineer, Jr. QA Engineer
         Technical Writer, DBA
```

### Swarm Agents (6)
```
Strategic Queen (opus) ─── Tactical Queen ─── Adaptive Queen
         │
         ├── Scout Explorer (reconnaissance, read-only)
         ├── Swarm Memory Manager (Obsidian coordination)
         └── Collective Intelligence Coordinator (consensus)
```

### Consensus Agents (2)
Raft Consensus Manager, Byzantine Fault Tolerance Coordinator

### SPARC Agents (5)
Specification Agent → Pseudocode Agent → Architecture Agent (opus) → Refinement Agent → Completion Agent

### GitHub Agents (5)
PR Manager, Issue Tracker, Release Manager, Code Review Swarm, Multi-Repo Coordinator

### Learning Agents (3)
ReasoningBank Manager, Pattern Distiller, Meta-Learning Analyst (opus)

### Governance Agents (3)
Governance Controller, Claims Manager, Compliance Auditor

### Optimization Agents (4)
Topology Optimizer, Performance Monitor (haiku), Resource Allocator, Benchmark Runner (haiku)

### Security Agents (3)
Security Analyst, CVE Remediation Specialist, AI Defense Specialist (opus)

### Team Configurations

| Config | Agents | Best For | Sprint Capacity |
|--------|--------|----------|-----------------|
| **Auto** | Scales per story | Mixed complexity | Dynamic |
| **Lean** | 5 agents | Bug fixes, simple features | ~25 pts |
| **Standard** | 12 agents + CTO | Typical feature development | ~50 pts |
| **Full** | 20+ agents + C-suite | Complex systems, new products | ~80 pts |
| **Swarm** | 8-15 agents + queen | Complex multi-domain tasks | Task-dependent |
| **SPARC** | 7-12 agents | Structured feature development | 1 feature at a time |
| **Governance** | 5 agents | Policy & compliance audits | On-demand |

---

## Obsidian Vault Structure

```
Projects/
└── {YourProject}/
    ├── Dashboard.md        ← Auto-generated Map of Content (MOC)
    ├── Knowledge/          ← Codebase model (updated pre-push)
    │   ├── Architecture.md
    │   ├── Components.md
    │   └── Dependencies.md
    ├── Decisions/          ← ADR-format decision records + conflict resolutions
    ├── Iterations/         ← Per-iteration notes (5-8 lines each)
    │   ├── 2026-03-22/
    │   │   └── *.canvas    ← Session flow visualizations
    │   └── Archive/        ← Smart archival (relevance-scored)
    ├── Predictions/        ← Persona notes + debate transcripts
    │   └── *.canvas        ← Persona debate visualizations
    │
    │   ── Agile Team ──────────────────────────────
    │
    ├── Backlog/            ← Product backlog (stories + epics)
    │   ├── _index.md       ← Priority-ordered story list
    │   ├── epics/          ← Epic definitions
    │   └── stories/        ← Individual story lifecycle documents
    ├── Sprints/            ← Sprint artifacts
    │   └── sprint-{N}/
    │       ├── Goal.md     ← Sprint goal + committed stories
    │       ├── Board.md    ← Kanban board (updated in place)
    │       ├── Standups/   ← Daily standup notes
    │       ├── Review.md   ← Sprint review + business assessment
    │       └── Retro.md    ← Retrospective + action items
    ├── Designs/            ← Technical design docs per story
    ├── Reviews/            ← Code review records per story
    ├── Team/               ← Team roster + working agreements
    │   ├── Roster.md
    │   └── Norms.md
    │
    │   ── Automation ──────────────────────────────
    │
    ├── Pipelines/          ← Chain pipeline definitions
    │   ├── {name}.md       ← Pipeline definition (steps, conditions)
    │   └── runs/           ← Pipeline run logs
    ├── Hooks/              ← Event-driven trigger definitions
    │   └── {name}.md       ← Hook definition (event, condition, action)
    ├── Canvases/           ← Generated visual diagrams
    │   ├── architecture.canvas
    │   ├── sprint-{N}-board.canvas
    │   ├── dependencies.canvas
    │   ├── timeline.canvas
    │   ├── knowledge-map.canvas
    │   └── pipeline-{name}.canvas
    │
    │   ── v0.4.0 ─────────────────────────────────
    │
    ├── Swarm/              ← Swarm state, strategy, worker outputs, consensus records
    ├── SPARC/              ← Per-feature SPARC phase outputs
    ├── ReasoningBank/      ← Learned patterns, strategy scores, meta-learning
    ├── Security/           ← Findings, CVE tracking, AI safety, claims log
    ├── Governance/         ← Policies, audit log, violations, approvals
    ├── Optimization/       ← Token tracking, routing log, cache stats
    ├── Streams/            ← Stream chain results
    └── Workers/            ← Background worker reports
Cross-Project/              ← Patterns across all projects
Daily Notes/                ← Enhanced session entries with backlinks
```

---

## Feature Reference by Version

<details>
<summary><b>v0.4.0 — Swarm Intelligence & Self-Learning</b></summary>

### Swarm Intelligence (`/autonexus:swarm`)
- Queen-led hierarchy with 3 queen types (Strategic, Tactical, Adaptive)
- 5 topologies (Hierarchical, Mesh, Hierarchical-Mesh, Ring, Star)
- 5 consensus algorithms (Raft, Byzantine, Gossip, CRDT, Quorum)
- Anti-drift enforcement every 3 worker completions
- Dynamic topology switching when throughput drops >40%

### SPARC Methodology (`/autonexus:sparc`)
- 5-phase structured development with dedicated agents per phase
- CTO architecture review at Phase 3 (opus-tier)
- Mini autoresearch loop in refinement phase
- ADR generation for architecture decisions

### ReasoningBank Self-Learning (`/autonexus:reason`)
- RETRIEVE-JUDGE-DISTILL cycle for pattern learning
- Strategy scoring (keep rate, avg delta, confidence)
- Confidence decay (5%/week without re-validation)
- Meta-learning analysis every 10 iterations
- Transfer learning across projects (50% initial confidence)

### Claims-Based Governance (`/autonexus:govern`)
- 7 claim types with glob-scoped permissions
- 4 security levels (Minimal → Admin)
- Auto-revocation on task completion
- 7 built-in policies (always active)
- Full audit trail

### Stream Chaining (`/autonexus:stream`)
- Agent-to-agent output piping (40-60% faster)
- 5 predefined pipelines
- Structured 500-token return format

### Three-Tier Task Routing (`/autonexus:auto-agent`, `/autonexus:optimize`)
- Tier 1: Direct edit (zero tokens)
- Tier 2: Sonnet (standard tasks)
- Tier 3: Opus/Swarm (complex tasks)
- 5-dimension complexity scoring

### ML Engineering (`/autonexus:mle-star`)
- 7-phase experiment lifecycle
- Literature search, ablation studies, ensembling

### Agent Spawning Protocol
- Universal 7-step cycle
- 3 spawning patterns (Sequential, Hierarchical, Stream)
- Gate enforcement with retry → escalate → human

</details>

<details>
<summary><b>v0.3.0 — Automation & Visualization</b></summary>

### Workflow Composition (`/autonexus:chain`)
- Pipeline engine with 5 built-in templates
- Conditional steps (`?on-success`, `?on-failure`, `?always`, `?if-metric-above:N`)
- Failure handling (`!halt`, `!skip`, `!retry:N`)
- Saved/resumable pipelines with Obsidian run logs

### On-Demand Canvas (`/autonexus:canvas`)
- 7 canvas types for Obsidian

### Event-Driven Hooks (`/autonexus:hook`)
- 12 event types, 7 action types, 7 built-in templates
- Variable substitution, condition expressions, priority ordering
- Local cache for offline operation
- Non-blocking with circular protection

</details>

<details>
<summary><b>v0.2.0 — Agile Agent Team</b></summary>

### Autonomous Agent Team
- 28+ specialized agents with isolated context windows
- Flexible team sizing (Auto, Lean, Standard, Full)
- Model-per-seniority (Opus for C-suite, Sonnet for others)

### Full Agile Ceremonies
- Sprint Planning, Daily Standups, Sprint Review, Retrospective
- Backlog Refinement

### Quality Gates
- Design → Code → QA → Acceptance (no bypass)
- Conflict resolution with escalation chain (Jr. → Sr. → Staff → Principal → CTO → CEO)

### Sprint Execution
- 10 testable stories per sprint
- Engineer agents run mini autoresearch loops
- Resumable via Obsidian sprint board

</details>

<details>
<summary><b>v0.1.1 — Loop Intelligence</b></summary>

### Loop Intelligence
- Strategy Learning (biases toward >60% keep rate strategies)
- Predictive Budgeting (estimates iterations needed)
- Parallel Branches (forks when stuck >8 consecutive discards)

### Session Management
- `/autonexus:resume`, `/autonexus:status`, `/autonexus:review`

### Obsidian Enhancements
- Smart Archival, Knowledge Decay Alerts, Dashboard MOC, Bookmarks
- Enhanced Daily Notes, Canvas Generation

</details>

---

## Key Design Decisions

- **Obsidian is best-effort:** Every MCP call uses retry-then-queue. If Obsidian is down, the loop continues with git + TSV. No data lost.
- **Iteration notes are concise:** 5-8 lines each. What was tried, result, why, next suggestion.
- **Smart archival:** Relevance-scored instead of blanket 30-day. Notes linked from decisions or tagged "important" survive.
- **Pre-push rebuilds knowledge:** Full codebase scan before every push ensures Knowledge Center is current.
- **Strategy learning:** The ReasoningBank learns which changes succeed vs fail in YOUR project and biases ideation accordingly.
- **Agents communicate through Obsidian:** Never see each other's context. The story/task note is the communication hub.
- **Anti-drift by design:** Structural topology + checkpoint gates + goal alignment checks prevent multi-agent drift.

---

## FAQ

<details>
<summary><b>General</b></summary>

**Q: Can I use this without Obsidian?**
A: Yes — it degrades gracefully to a standard autoresearch-style loop with git + TSV. You lose the persistent knowledge layer but everything else works.

**Q: What if Obsidian isn't running?**
A: AutoNexus warns once and runs in local-only mode. Obsidian writes queue to `obsidian-sync-queue.md` and drain on next successful connection.

**Q: How many Obsidian MCP calls per iteration?**
A: 2 reads + 1-2 writes. ~4-6 seconds overhead.

**Q: Does the pre-push rebuild block my push?**
A: It takes 2-5 minutes. Claude scans the codebase and updates Knowledge Center notes before proceeding.

</details>

<details>
<summary><b>Choosing the Right Command</b></summary>

**Q: When should I use Swarm vs the regular loop?**
A: Regular loop for single-domain tasks (improve tests, optimize one module). Swarm when the task spans multiple domains or when >8 consecutive discards suggest single-agent exploration is exhausted.

**Q: When should I use SPARC vs the regular loop?**
A: SPARC for medium-to-large features where you'd normally write a design doc. Regular loop for metric-driven improvements. `/autonexus:fix` for simple error repair.

**Q: When should I use chain vs stream?**
A: Chain for persistence, resumability, and conditional logic. Stream when speed matters (40-60% faster) and you don't need to resume mid-pipeline.

**Q: Can I use SPARC for every feature?**
A: SPARC adds 5-phase overhead. For quick fixes or small changes, the regular loop or `/autonexus:fix` is faster. Use SPARC when you'd normally write a design doc.

</details>

<details>
<summary><b>Agents & Sprints</b></summary>

**Q: Do agents actually write code?**
A: Yes. Engineer agents read the design from Obsidian, read the codebase, then run a mini autoresearch loop until acceptance tests pass. They commit real code to git.

**Q: How do agents communicate?**
A: Only through Obsidian notes. Agent A writes to a story note → Agent B reads from that same note. They never see each other's context window.

**Q: How many agent spawns does a sprint take?**
A: ~112 for a clean sprint (10 stories), ~125 with rework.

**Q: Can I resume an interrupted sprint?**
A: Yes — `--resume` reads the sprint board from Obsidian and picks up from the last completed step.

**Q: What happens when agents disagree?**
A: Automatic escalation: Jr. → Sr. → Staff → Principal → CTO → CEO. The supervisor's decision becomes a binding ADR.

**Q: Can I run just the planning ceremony?**
A: Yes — `--phase planning` runs only sprint planning. Same for `--phase review` and `--phase retro`.

</details>

<details>
<summary><b>Automation</b></summary>

**Q: Can I chain commands without Obsidian?**
A: Yes — inline pipelines and built-in templates work without Obsidian. You can't save/load pipelines or generate run logs.

**Q: Do hooks work without Obsidian?**
A: Hook management (add/list/remove) requires Obsidian. But hook evaluation during loops works offline via local cache.

**Q: Can hooks trigger other hooks?**
A: Yes, with protection — max 1 trigger per hook per iteration prevents infinite loops.

</details>

<details>
<summary><b>Self-Learning & Optimization</b></summary>

**Q: How does the ReasoningBank make sessions smarter?**
A: Every "keep" stores a pattern. Every "discard" records negative evidence. Before ideating, the system retrieves matching patterns and biases toward proven strategies. After 5+ sessions, keep rates typically improve 15-25%.

**Q: Does three-tier routing really save tokens?**
A: Yes — 30-50% in typical sessions. Simple renames (Tier 1) cost zero tokens. Standard tasks route to Sonnet instead of Opus. Run `/autonexus:optimize --dashboard` to see actual savings.

**Q: What does claims-based authorization actually do?**
A: Prevents agents from accessing files/commands outside their scope. A Jr. QA agent can't modify production configs. Every action is checked against the agent's claim set and violations are logged.

</details>

---

## Credits

- **[Andrej Karpathy](https://github.com/karpathy)** — for [autoresearch](https://github.com/karpathy/autoresearch)
- **[Udit Goenka](https://github.com/uditgoenka)** — for [Claude Autoresearch](https://github.com/uditgoenka/autoresearch)
- **[rUv / ruflo](https://github.com/ruvnet/ruflo)** — for swarm intelligence, SPARC methodology, and self-learning patterns that inspired v0.4.0
- **[Anthropic](https://anthropic.com)** — for [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

---

## License

MIT — see [LICENSE](LICENSE).
