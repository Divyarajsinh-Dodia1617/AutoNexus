# AutoNexus

**Autonomous goal-directed iteration engine with Obsidian as knowledge backbone — now with a full Agile agent team.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [Claude Autoresearch](https://github.com/uditgoenka/autoresearch). AutoNexus adds persistent cross-session knowledge via Obsidian MCP — every iteration reads from and writes to your Obsidian vault. The new **Agile workflow** orchestrates 28+ specialized agents (CEO to Jr. Engineer) through full sprint lifecycles with quality gates, conflict resolution, and Agile ceremonies.

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blue?logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-0.3.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

*"Set the GOAL → Claude runs the LOOP → Knowledge persists in Obsidian → You wake up to results AND understanding."*
*"Define the BACKLOG → Agents run the SPRINT → Design, build, review, test, ship — like a real MNC team."*
*"Chain WORKFLOWS → Hook EVENTS → Canvas VISUALIZES → The system drives itself."*

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

---

## Commands

### Core Loop

| Command | What it does |
|---------|--------------|
| `/autonexus` | Run the autonomous iteration loop with Obsidian integration |
| `/autonexus:plan` | Interactive wizard: Goal → Scope, Metric, Verify config |

### Agile Team

| Command | What it does |
|---------|--------------|
| `/autonexus:agile` | **Full Agile sprint lifecycle with autonomous agent team** — sprint planning, execution (10 stories), review, retrospective |
| `/autonexus:refine-backlog` | **Interactive backlog grooming** — create, refine, reprioritize, split, or archive stories with PM + Architect agent assistance |

### Analysis & Quality

| Command | What it does |
|---------|--------------|
| `/autonexus:predict` | Multi-persona swarm prediction with persona notes in Obsidian |
| `/autonexus:debug` | Autonomous bug-hunting loop with scientific method |
| `/autonexus:fix` | Autonomous error repair until zero errors remain |
| `/autonexus:security` | STRIDE + OWASP Top 10 security audit with red-team personas |

### Workflows

| Command | What it does |
|---------|--------------|
| `/autonexus:ship` | Universal shipping workflow (code, content, marketing, sales, research, design) |
| `/autonexus:scenario` | Scenario-driven use case generator across 12 dimensions |
| `/autonexus:learn` | Autonomous documentation engine with Obsidian Knowledge Center sync |
| `/autonexus:chain` | **Workflow composition** — chain multiple commands into sequential pipelines with conditions |
| `/autonexus:canvas` | **On-demand Obsidian Canvas** — architecture, sprint, dependency, timeline, knowledge, flow diagrams |
| `/autonexus:hook` | **Event-driven triggers** — define rules that fire actions when events occur in loops/sprints |

### Session Management

| Command | What it does |
|---------|--------------|
| `/autonexus:resume` | Resume a previous session — reload config from Obsidian, pick up where you left off |
| `/autonexus:status` | Read-only project dashboard — session history, knowledge freshness, strategy profile |
| `/autonexus:review` | Post-session analysis — pattern detection, lessons learned, session flow canvas |
| `/autonexus:setup` | One-time Obsidian MCP setup wizard — API key, platform detection, .mcp.json config |

---

## Agent Team Hierarchy

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
CPO ─── Product Manager
 │       Technical Program Manager
 │       Scrum Master (Orchestrator)
 │
 └────── UX Designer
         UI Designer
         QA Manager ── Sr. QA Engineer ── QA Engineer
         Technical Writer
         DBA
```

### Team Configurations

| Config | Agents | Best For | Sprint Capacity |
|--------|--------|----------|-----------------|
| **Auto** | Scales per story | Mixed complexity | Dynamic |
| **Lean** | 5 agents (PM, Architect, Sr. FS Eng, QA, EM) | Bug fixes, simple features | ~25 pts |
| **Standard** | 12 agents + CTO review | Typical feature development | ~50 pts |
| **Full** | 20+ agents + C-suite | Complex systems, new products | ~80 pts |

---

## Quick Start

### 1. Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [Obsidian](https://obsidian.md) desktop app
- [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) Obsidian community plugin enabled

### 2. Install AutoNexus

**Option A — Plugin marketplace (recommended):**

```
/plugin marketplace add Divyarajsinh-Dodia1617/AutoNexus
/plugin install autonexus@autonexus
/reload-plugins
```

That's it. All 18 commands are available immediately.

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

### 4. Run It

```
/autonexus
Goal: Increase test coverage from 72% to 90%
Scope: src/**/*.ts
Metric: coverage % (higher is better)
Verify: npm test -- --coverage | grep "All files"
```

### 5. Run an Agile Sprint

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

The orchestrator will:
1. Run **Sprint Planning** — PM selects stories, Architect assesses, team estimates, EM assigns
2. For each of **10 stories**: Design → Implement → Review → QA → Accept
3. Run **Sprint Review** — PM acceptance, CEO business assessment
4. Run **Retrospective** — metrics, agent reflections, action items
5. All artifacts persist in your Obsidian vault

### 6. Check Obsidian

After a session, your vault contains:
- Per-iteration notes with rationale
- Session summary with top wins and failed approaches
- Daily note entry with session overview
- Knowledge Center updated on push

After a sprint, your vault also contains:
- Sprint board with kanban state
- Design docs, code reviews, QA logs per story
- Sprint review + business assessment
- Retrospective with metrics + action items
- ADRs from any conflict resolutions

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
    └── Canvases/           ← Generated visual diagrams
        ├── architecture.canvas
        ├── sprint-{N}-board.canvas
        ├── dependencies.canvas
        ├── timeline.canvas
        ├── knowledge-map.canvas
        └── pipeline-{name}.canvas
Cross-Project/              ← Patterns across all projects
Daily Notes/                ← Enhanced session entries with backlinks
```

---

## Key Design Decisions

- **Obsidian is best-effort:** Every MCP call uses retry-then-queue. If Obsidian is down, the loop continues with git + TSV. No data lost.
- **Iteration notes are concise:** 5-8 lines each. What was tried, result, why, next suggestion.
- **Smart archival:** Relevance-scored instead of blanket 30-day. Notes linked from decisions or tagged "important" survive. Unlinked old notes archive first.
- **Pre-push rebuilds knowledge:** Full codebase scan before every push ensures Knowledge Center is current.
- **Predict personas persist:** Each persona's analysis becomes a permanent, searchable Obsidian note.
- **Strategy learning:** The loop learns from past sessions — which change types succeed vs fail in your project — and biases ideation accordingly.
- **Canvas visualizations:** Auto-generated `.canvas` files for predict debate maps and session flow diagrams.

---

## v0.3.0 Features — Automation & Visualization

### Workflow Composition (`/autonexus:chain`)
- **Pipeline engine** — Chain any autonexus commands into sequential workflows: `predict → agile → review → ship`
- **5 built-in templates** — full-sprint, quality-audit, release-prep, deep-analysis, continuous-quality
- **Conditional steps** — `?on-success`, `?on-failure`, `?always`, `?if-metric-above:N` conditions per step
- **Failure handling** — `!halt`, `!skip`, `!retry:N` per step
- **Saved pipelines** — Define once in Obsidian, reuse across sessions. `--save`, `--pipeline`, `--list`
- **Resumable** — Interrupted pipelines resume from last completed step with `--resume`
- **Pipeline run logs** — Every run produces a detailed log in Obsidian with per-step results and artifacts

### On-Demand Canvas (`/autonexus:canvas`)
- **7 canvas types** — Architecture, Sprint Board, Dependency Graph, Timeline, Knowledge Map, Data Flow, Pipeline
- **Architecture canvas** — Component relationships color-coded by risk level, hierarchical layout
- **Sprint board canvas** — Kanban visualization of story cards with blocking relationships
- **Dependency canvas** — Import/dependency graph from codebase analysis
- **Timeline canvas** — Session history over time with keep-rate color coding
- **Knowledge map canvas** — All Obsidian project notes with backlink connections, clustered by type
- **Data flow canvas** — Entry points → processors → data stores → outputs
- **Pipeline canvas** — Visual pipeline step flow with status colors

### Event-Driven Hooks (`/autonexus:hook`)
- **12 event types** — iteration-complete, consecutive-discards, consecutive-crashes, keep-rate-drop, metric-threshold, goal-achieved, session-end, finding-created, story-complete, gate-failed, sprint-end, pipeline-step-failed
- **7 action types** — run-command, create-note, create-story, tag-note, alert, pause, log
- **7 built-in templates** — auto-debug-on-crashes, backlog-from-findings, pause-on-low-keep-rate, celebrate-goal, auto-fix-on-guard-fail, auto-review-on-session-end, escalate-gate-failures
- **Variable substitution** — `$METRIC`, `$SEVERITY`, `$KEEP_RATE`, etc. in all action fields
- **Condition expressions** — `count >= 3`, `severity in ["CRITICAL", "HIGH"]`, `keep_rate < 20`, AND combinations
- **Priority ordering** — Lower priority number fires first; hooks evaluated in order
- **Local cache** — `autonexus-hooks.json` enables hook evaluation even when Obsidian is unavailable
- **Management CLI** — `--add`, `--list`, `--enable`, `--disable`, `--remove`, `--history`, `--dry-run`
- **Non-blocking** — Hook actions never block the loop; failures are logged and skipped
- **Circular protection** — Max 1 trigger per hook per iteration prevents infinite loops

---

## v0.2.0 Features — Agile Agent Team

### Autonomous Agent Team
- **28+ specialized agents** — CEO, CTO, CPO, Engineering Manager, Product Manager, Scrum Master, Architects, Sr./Mid/Jr. Engineers (Backend, Frontend, Full Stack), QA Manager, QA Engineers, DevOps, SRE, Security Engineer, UX/UI Designers, Technical Writer, DBA
- **Isolated context windows** — Every agent runs as a subagent with its own context. The orchestrator stays clean — it never reads source code or writes implementation
- **Flexible team sizing** — Auto (scales per story), Lean (5 agents), Standard (12), Full (20+)
- **Model-per-seniority** — Opus for C-suite/Principal, Sonnet for everyone else

### Full Agile Ceremonies
- **Sprint Planning** — PM prioritizes backlog, Architect assesses feasibility, Planning Poker with 3 parallel estimators, EM assigns, CTO strategic review
- **Daily Standups** — Scrum Master compiles status, burn metrics, blocker identification
- **Sprint Review** — PM acceptance verification, CEO business value assessment
- **Retrospective** — Metrics compilation, parallel agent reflections, EM action items
- **Backlog Refinement** — Interactive grooming via `/autonexus:refine-backlog`

### Quality Gates
- **Design Gate** — Architect design + CTO approval before implementation
- **Code Gate** — Sr. Engineer review + Security pass before QA
- **QA Gate** — All tests pass + coverage target met before acceptance
- **Acceptance Gate** — PM verifies all acceptance criteria before Done
- **No bypass** — Gates escalate to supervisors on repeated failure

### Conflict Resolution
- Automatic escalation to the appropriate supervisor in the chain
- Jr. → Sr. → Staff → Principal → CTO → CEO
- Decisions recorded as binding ADRs in Obsidian

### Sprint Execution
- **10 testable stories per sprint** — Each story has acceptance criteria, verify_command, test_file
- **Per-story lifecycle** — Design → Implement → Code Review → QA → Accept
- **Engineer agents run mini autoresearch loops** — Iterate until acceptance tests pass (max 20 iterations)
- **Resumable** — Sprint board in Obsidian is the checkpoint. `--resume` picks up where you left off

### Obsidian as Communication Channel
- Agents communicate **only through Obsidian notes** — never see each other's context
- Story notes are lifecycle documents — every agent appends to the same note
- Sprint board updated in place throughout execution
- All decisions, reviews, and test results persist as searchable Obsidian notes

---

## v0.1.1 Features

### Loop Intelligence
- **Strategy Learning** — Analyzes past keep rates by change category across sessions. Biases ideation toward historically successful strategies (>60% keep rate).
- **Predictive Budgeting** — Estimates iterations needed to reach your goal based on past session data (requires 2+ past sessions).
- **Parallel Branches** — When stuck (>8 consecutive discards), forks into 2-3 git worktrees, tries different strategies in parallel, merges the winner.

### Commands
- **`/autonexus:resume`** — Resume any previous session from Obsidian. Reloads goal, scope, metric, verify. No re-setup needed.
- **`/autonexus:status`** — Read-only project dashboard. Shows session history, knowledge freshness, keep rate, top strategies, unresolved findings.
- **`/autonexus:review`** — Post-session analysis. Identifies patterns, generates lessons learned notes, optionally writes cross-project insights.

### Obsidian Enhancements
- **Smart Archival** — Relevance-scored instead of blanket 30-day. Linked notes survive, unlinked decay. `autonexus/important` tag = never archived.
- **Knowledge Decay Alerts** — Warns at session start if Knowledge Center is stale (>20 commits behind or >14 days old).
- **Dashboard MOC** — Auto-generated Map of Content per project with stats, recent sessions, decisions, findings, strategies.
- **Bookmarks** — Auto-tags outstanding iterations, decisions, and critical findings with `autonexus/bookmarked`.
- **Enhanced Daily Notes** — Richer format with iteration backlinks, decision links, delta values, and project dashboard link.
- **Canvas Generation** — Visual `.canvas` files for predict persona debate maps and session iteration flow diagrams.

---

## FAQ

**Q: What if Obsidian isn't running?**
A: AutoNexus warns once and runs in local-only mode. Git + TSV work normally. Obsidian writes queue to `obsidian-sync-queue.md` and drain on next successful connection.

**Q: How many Obsidian MCP calls per iteration?**
A: 2 reads (failed approaches + predict warnings) + 1-2 writes (iteration note + optional tags). ~4-6 seconds overhead.

**Q: Does the pre-push rebuild block my push?**
A: It takes 2-5 minutes. It runs when you say "push" — Claude scans the codebase and updates 3 Knowledge Center notes in Obsidian before proceeding.

**Q: Can I use this without Obsidian?**
A: Yes — it degrades gracefully to a standard autoresearch-style loop with git + TSV. You just lose the persistent knowledge layer.

**Q: How many agent spawns does a sprint take?**
A: ~112 for a clean sprint (10 stories), ~125 with rework. Each spawn is focused with an isolated context — the orchestrator stays lightweight.

**Q: Do agents actually write code?**
A: Yes. Engineer agents read the design from Obsidian, read the codebase, then run a mini autoresearch loop — iterating until all acceptance tests pass. They commit real code to git.

**Q: How do agents communicate?**
A: Only through Obsidian notes. Agent A writes to a story note → Agent B reads from that same note. They never see each other's context window. The story note is the communication hub.

**Q: Can I resume an interrupted sprint?**
A: Yes — `--resume` reads the sprint board from Obsidian and picks up from the last completed step. All state lives in Obsidian, not in memory.

**Q: What happens when agents disagree?**
A: The orchestrator detects the conflict and automatically spawns the appropriate supervisor (e.g., Staff Engineer for Sr. vs Sr. disputes). The supervisor's decision becomes a binding ADR.

**Q: Can I run just the planning ceremony?**
A: Yes — `--phase planning` runs only sprint planning. Same for `--phase review` and `--phase retro`.

**Q: Can I chain commands without Obsidian?**
A: Yes — inline pipelines and built-in templates work without Obsidian. You just can't save/load pipelines or generate run logs. Use `--template full-sprint` to get started.

**Q: Do hooks work without Obsidian?**
A: Hook management (add/list/remove) requires Obsidian. But hook evaluation during loops works offline — hooks are cached locally in `autonexus-hooks.json` at session start.

**Q: Can hooks trigger other hooks?**
A: Yes, but with protection — each hook can fire at most once per iteration to prevent infinite loops. A `run-command` action that triggers the same event type won't re-fire the same hook.

**Q: What canvas types are available?**
A: Architecture (component relationships), Sprint (kanban board), Dependency (import graph), Timeline (session history), Knowledge (all notes with links), Flow (data flow), and Pipeline (workflow steps).

---

## Credits

- **[Andrej Karpathy](https://github.com/karpathy)** — for [autoresearch](https://github.com/karpathy/autoresearch)
- **[Udit Goenka](https://github.com/uditgoenka)** — for [Claude Autoresearch](https://github.com/uditgoenka/autoresearch)
- **[Anthropic](https://anthropic.com)** — for [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

---

## License

MIT — see [LICENSE](LICENSE).
