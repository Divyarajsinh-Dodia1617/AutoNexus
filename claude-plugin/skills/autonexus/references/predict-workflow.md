# Predict Workflow — /autonexus:predict

Multi-persona swarm prediction that pre-analyzes code from multiple expert perspectives. Simulates 3-8 personas that independently analyze, debate, and reach consensus — producing ranked findings and hypotheses. All within Claude's native context. Persona notes and debate transcripts are written to Obsidian for persistent cross-session knowledge.

**Core idea:** Read code → Build knowledge files → Generate personas → Independent analysis → Debate → Consensus → Report → Write to Obsidian → Optional chain handoff.

## Trigger

- User invokes `/autonexus:predict`
- User says "predict", "multi-perspective analysis", "swarm analysis", "analyze from different angles"

## PREREQUISITE: Interactive Setup

**CRITICAL — BLOCKING PREREQUISITE:** If scope, goal, and depth are not all provided, MUST use `AskUserQuestion`.

**Adaptive question selection:**
- No input → ask all 4 questions
- Scope provided but no goal → ask 2, 3, 4
- Scope + goal but no depth → ask 3, 4
- All three provided → skip setup

Batch ALL questions into a SINGLE `AskUserQuestion` call:

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Scope` | "Which files should I analyze?" | Suggested globs + "Entire codebase" |
| 2 | `Goal` | "What should the swarm focus on?" | "Code quality & reliability", "Security vulnerabilities", "Performance bottlenecks", "Architecture review" |
| 3 | `Depth` | "How deep should I analyze?" | "Shallow (3 personas, 1 round)", "Standard (5 personas, 2 rounds)", "Deep (8 personas, 3 rounds)" |
| 4 | `Chain` | "After analysis, chain to another tool?" | "Debug", "Security", "Fix", "Ship", "Scenario", "No chain — report only" |

## Architecture

```
/autonexus:predict
  ├── Phase 1: Setup — Config validation
  ├── Phase 2: Reconnaissance — Build knowledge files (check Obsidian cache first)
  ├── Phase 3: Persona Generation — Create expert personas
  ├── Phase 4: Independent Analysis — Each persona analyzes independently
  ├── Phase 5: Debate — Structured cross-examination (1-3 rounds)
  ├── Phase 6: Consensus — Aggregation + anti-herd check
  ├── Phase 7: Report — Generate output files + write to Obsidian
  └── Phase 8: Handoff — Write handoff.json, optional chain
```

## Phase 1: Setup

Parse and validate configuration:
- Resolve `--scope` globs to file list
- Map `--depth` to persona/round count:
  - `shallow` → 3 personas, 1 round
  - `standard` → 5 personas, 2 rounds (default)
  - `deep` → 8 personas, 3 rounds
- Validate `--chain` targets (debug, security, fix, ship, scenario)
- If `--adversarial`, swap persona set

**Output:** `Phase 1: Setup — [N] files in scope, [M] personas, [K] rounds planned`

## Phase 2: Reconnaissance — Build Knowledge Files

### Obsidian Knowledge Cache Check (if OBSIDIAN_AVAILABLE)

Before scanning the codebase, check if Obsidian Knowledge Center notes exist and are current:

```
TRY:
  arch = obsidian_read_note({ filePath: "Projects/{ProjectName}/Knowledge/Architecture.md", includeStat: true })
  IF arch exists AND arch.frontmatter.commit_hash == git rev-parse HEAD:
    LOG "Knowledge cache hit — using existing Obsidian Knowledge Center"
    Use cached knowledge as reconnaissance context
    SKIP full codebase scan (still read in-scope source files)
  ELIF arch exists AND arch.frontmatter.commit_hash != HEAD:
    LOG "Knowledge cache stale — incremental update"
    Run: git diff --name-only {cached_hash}..HEAD
    Re-analyze only changed files
    Update knowledge files
  ELSE:
    LOG "No knowledge cache — full scan"
    Proceed with full reconnaissance
```

### Full Reconnaissance

Read all in-scope source files and build three knowledge files:

#### codebase-analysis.md

```markdown
---
commit_hash: {git rev-parse HEAD}
analyzed_at: {ISO timestamp}
scope: {glob patterns}
files_analyzed: {count}
---

## Functions
| File | Function | Signature | Lines | Calls | Called By |

## Classes & Types
| File | Name | Kind | Key Properties | Methods |

## Routes / Endpoints
| Method | Path | File | Handler | Auth Required | Input |

## Models / Database
| Name | File | Fields | Indexes | Relations |
```

#### dependency-map.md

```markdown
---
commit_hash: {git rev-parse HEAD}
---

## Import Graph
| File | Imports From | Symbols |

## Call Graph
| Caller | Callee | File:Line | Type |

## Data Flows
| Source | Transform | Sink | Risk Areas |
```

#### component-clusters.md

```markdown
---
commit_hash: {git rev-parse HEAD}
---

## Clusters
| Cluster | Files | Key Entities | External Deps | Risk Areas |
```

### Write Knowledge Files to Obsidian (if OBSIDIAN_AVAILABLE)

After building knowledge files, write/update them in Obsidian Knowledge Center:

```
FOR each of [Architecture.md, Components.md, Dependencies.md]:
  obsidian_update_note({
    targetType: "filePath",
    targetIdentifier: "Projects/{ProjectName}/Knowledge/{file}",
    content: {generated from knowledge analysis},
    modificationType: "wholeFile",
    wholeFileMode: "overwrite",
    createIfNeeded: true
  })
```

Uses retry-then-queue protocol.

**Output:** `Phase 2: Reconnaissance — [N] files scanned, [M] entities, [K] clusters`

## Phase 3: Persona Generation

### Default Persona Set

| # | Persona | Focus Areas | Bias |
|---|---------|-------------|------|
| 1 | Architecture Reviewer | Scalability, coupling, design patterns, tech debt | Skeptical of god objects; prefers separation |
| 2 | Security Analyst | OWASP Top 10, injection, auth failures, data exposure | Assumes hostile inputs; trusts nothing external |
| 3 | Performance Engineer | Complexity, N+1 queries, memory, blocking I/O | Prefers evidence; skeptical of premature optimization |
| 4 | Reliability Engineer | Error handling, retry logic, race conditions, edge cases | Assumes failure; asks "what if X is nil?" |
| 5 | Devil's Advocate | Challenges consensus, blind spots, non-code hypotheses | MUST challenge >=50% of majority positions |

### Adversarial Persona Set (`--adversarial`)

| # | Persona | Focus |
|---|---------|-------|
| 1 | Red Team Attacker | Exploitation paths, attack chains, privilege escalation |
| 2 | Blue Team Defender | Detection gaps, missing monitoring, incident response |
| 3 | Insider Threat | Data exfiltration, audit trail gaps, privilege abuse |
| 4 | Supply Chain Analyst | Dependency risks, build pipeline, unsigned artifacts |
| 5 | Judge | Evaluates adversarial claims, assigns exploitability scores |

### Persona Prompt Template

```
You are {name}, a {role} with expertise in {expertise}.

Task: Analyze the provided codebase files and knowledge context. Produce findings independently.

Context:
- codebase-analysis.md: Functions, types, routes, models
- dependency-map.md: Import graph, call graph, data flows
- component-clusters.md: Logical groupings and risk areas
- In-scope source files: {file list}

Goal: {user-provided goal}

Constraints:
- Every finding MUST include file:line reference
- Maximum {finding_limit} findings (prioritize highest-severity)
- Do NOT hallucinate APIs or functions not in source files
- Confidence: HIGH (certain), MEDIUM (likely but runtime-dependent), LOW (theoretical)

Bias: {bias_direction}

Output format:
<{persona_tag}_findings>
  <finding id="{persona_abbr}-{n}">
    <title>{one-line title}</title>
    <location>{file}:{line}</location>
    <severity>CRITICAL|HIGH|MEDIUM|LOW</severity>
    <confidence>HIGH|MEDIUM|LOW</confidence>
    <evidence>{exact code or flow}</evidence>
    <recommendation>{concrete action}</recommendation>
  </finding>
</{persona_tag}_findings>
```

**Output:** `Phase 3: [N] personas generated — [names]`

## Phase 4: Independent Analysis

Each persona receives:
- Their persona prompt
- All three knowledge files
- All in-scope source files

**Isolation:** Personas do NOT see each other's outputs.
**Finding limit:** `ceil(total_budget / persona_count)` — default 8 per persona.

**Output:** `Phase 4: Independent analysis — [N] personas produced [M] total findings`

## Phase 5: Debate — Cross-Examination

Personas see ALL Phase 4 outputs. Must respond to peers, challenge disagreements, revise if evidence is compelling.

**Rounds:** 1-3 based on depth.

### Devil's Advocate Rules

- **MUST** challenge >=50% of majority positions
- **MUST** propose >=1 non-code hypothesis per round
- **MUST** question highest-consensus finding
- **MUST NOT** simply agree — concede with conditions if evidence overwhelming

**Output:** `Phase 5: Debate — [N] rounds, [M] challenges, [K] positions revised`

## Phase 6: Consensus

### Voting Protocol

| Vote | Meaning |
|------|---------|
| `confirm` | Finding is valid |
| `dispute` | Finding wrong or overstated |
| `abstain` | Outside domain |

**Consensus thresholds:**
- >=3/5 → **Confirmed**
- 2/5 → **Probable**
- 1/5 → **Minority**
- 0/5 → **Discarded**

### Anti-Herd Detection

| Signal | Threshold |
|--------|-----------|
| `flip_rate` (changed positions / total) | > 0.8 suspicious |
| `entropy` (position distribution) | < 0.3 suspicious |
| `convergence_speed` (rounds to 80% agreement) | 1 round suspicious |

**GROUPTHINK WARNING** if `flip_rate > 0.8` AND `entropy < 0.3`:
1. Preserve ALL minority findings
2. Flag in overview
3. Suggest `--adversarial` re-run

### Priority Ranking

```
priority_score = severity_weight * 0.4 + confidence_boost * 0.2 + consensus_ratio * 0.4

severity_weight = CRITICAL:4, HIGH:3, MEDIUM:2, LOW:1
confidence_boost = HIGH:1.0, MEDIUM:0.6, LOW:0.3
consensus_ratio = personas_confirmed / personas_total
```

**Output:** `Phase 6: Consensus — [N] confirmed, [M] probable, [K] minority`

## Phase 7: Report + Obsidian Notes

### Local Output Files

Write to `predict/{YYMMDD}-{HHMM}-{slug}/`:
- `overview.md` — summary, anti-herd status, top findings
- `findings.md` — all findings ranked by priority with votes
- `hypothesis-queue.md` — testable hypotheses for chain handoff
- `persona-debates.md` — full debate transcript
- `predict-results.tsv` — per-persona per-round log
- `handoff.json` — machine-readable schema for downstream tools
- `codebase-analysis.md`, `dependency-map.md`, `component-clusters.md` — knowledge files

### Obsidian Notes (if OBSIDIAN_AVAILABLE)

Write 6 notes (for standard depth with 5 personas):

**1-5. Persona Notes (1 per persona):**

```
FOR each persona:
  obsidian_update_note({
    targetType: "filePath",
    targetIdentifier: "Projects/{ProjectName}/Predictions/{YYYY-MM-DD}-{slug}/persona-{name}.md",
    content: "{rendered persona note template from obsidian-integration.md}",
    modificationType: "wholeFile",
    wholeFileMode: "overwrite",
    createIfNeeded: true
  })
```

**6. Debate Note:**

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Predictions/{YYYY-MM-DD}-{slug}/debate.md",
  content: "{rendered debate note template from obsidian-integration.md}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Each write uses retry-then-queue. If any write fails twice, queue it and continue with remaining notes. Never block the report generation on Obsidian.

**Output:** `Phase 7: Report — [N] local files + [M] Obsidian notes written`

## Phase 8: Handoff

### handoff.json Schema

```json
{
  "version": "1.0",
  "tool": "predict",
  "generated_at": "{ISO timestamp}",
  "commit_hash": "{hash}",
  "scope": ["{globs}"],
  "summary": {
    "personas": 5,
    "rounds": 2,
    "findings_confirmed": 8,
    "findings_probable": 3,
    "anti_herd_passed": true,
    "predict_score": 142
  },
  "findings": [{
    "id": "H-01",
    "type": "security",
    "severity": "HIGH",
    "confidence": "HIGH",
    "location": "src/auth/jwt.ts:33",
    "title": "{title}",
    "description": "{description}",
    "recommendation": "{action}",
    "personas_agreed": 4,
    "personas_total": 5
  }],
  "hypotheses": [{
    "rank": 1,
    "id": "H-01",
    "hypothesis": "{testable statement}",
    "confidence": "HIGH",
    "location": "{file:line}"
  }]
}
```

### Chain Conversion

**`--chain debug`:** Map findings to hypotheses for debug loop.
**`--chain security`:** Filter security findings → STRIDE categories.
**`--chain fix`:** Sort by severity * consensus → fix queue.
**`--chain ship`:** Convert to gate classifications (BLOCKER/WARNING/INFO).
**`--chain scenario`:** Each finding becomes a scenario seed.

**Multi-chain:** `--chain scenario,debug,fix` executes sequentially. Each stage feeds the next via handoff.

### Empirical Evidence Rule

**CRITICAL:** When chained, loop results ALWAYS override swarm consensus. Predictions are starting points, not conclusions.

## Safety

### Input Sanitization
Scan for injection patterns before including in persona prompts. Flag suspicious patterns in overview.md for human review.

### PII Scrubbing
Redact emails, phone numbers, secrets, and IP addresses in findings.

### Budget Enforcement

| Budget Tier | Token Limit | Action |
|-------------|-------------|--------|
| Standard | 200,000 | Proceed |
| Warning | 400,000 | Warn user |
| Hard limit | 600,000 | Halt, ask to narrow scope |

## Composite Metric

```
predict_score = findings_confirmed * 15
              + findings_probable * 8
              + minority_preserved * 3
              + (personas_active / personas_total) * 20
              + (rounds_completed / planned_rounds) * 10
              + anti_herd_passed * 5
```

## Flags

| Flag | Purpose |
|------|---------|
| `--scope <glob>` | Files to analyze |
| `--goal <text>` | Focus area |
| `--depth <level>` | shallow/standard/deep |
| `--personas <N>` | Override count (3-8) |
| `--rounds <N>` | Override rounds (1-3) |
| `--adversarial` | Red-team persona set |
| `--chain <tools>` | Chain to downstream tools |
| `--budget <N>` | Max total findings (default: 40) |
| `--fail-on <severity>` | Exit non-zero for CI/CD |
| `--incremental` | Reuse existing knowledge files |

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Skip Devil's Advocate | Removes diversity that makes swarm valuable |
| Trust swarm over empirical evidence | Loop experiments always win |
| Use >8 personas | Diminishing returns |
| Skip debate (--rounds 0) | Independent opinions, not swarm intelligence |
| Ignore minority findings | Minorities often right on non-obvious issues |
| Chain without reviewing findings | Garbage in → garbage out |
