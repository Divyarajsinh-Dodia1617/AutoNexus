# AutoNexus

**Autonomous goal-directed iteration engine with Obsidian as knowledge backbone.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [Claude Autoresearch](https://github.com/uditgoenka/autoresearch). AutoNexus adds persistent cross-session knowledge via Obsidian MCP — every iteration reads from and writes to your Obsidian vault.

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blue?logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-0.1.1-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

*"Set the GOAL → Claude runs the LOOP → Knowledge persists in Obsidian → You wake up to results AND understanding."*

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

| Command | What it does |
|---------|--------------|
| `/autonexus` | Run the autonomous iteration loop with Obsidian integration |
| `/autonexus:plan` | Interactive wizard: Goal → Scope, Metric, Verify config |
| `/autonexus:predict` | Multi-persona swarm prediction with persona notes in Obsidian |
| `/autonexus:debug` | Autonomous bug-hunting loop with scientific method |
| `/autonexus:fix` | Autonomous error repair until zero errors remain |
| `/autonexus:security` | STRIDE + OWASP Top 10 security audit with red-team personas |
| `/autonexus:ship` | Universal shipping workflow (code, content, marketing, sales, research, design) |
| `/autonexus:scenario` | Scenario-driven use case generator across 12 dimensions |
| `/autonexus:learn` | Autonomous documentation engine with Obsidian Knowledge Center sync |
| `/autonexus:resume` | Resume a previous session — reload config from Obsidian, pick up where you left off |
| `/autonexus:status` | Read-only project dashboard — session history, knowledge freshness, strategy profile |
| `/autonexus:review` | Post-session analysis — pattern detection, lessons learned, session flow canvas |
| `/autonexus:setup` | One-time Obsidian MCP setup wizard — API key, platform detection, .mcp.json config |

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

That's it. All 13 commands are available immediately.

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

### 5. Check Obsidian

After the session, your vault contains:
- Per-iteration notes with rationale
- Session summary with top wins and failed approaches
- Daily note entry with session overview
- Knowledge Center updated on push

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
    ├── Decisions/          ← ADR-format decision records
    ├── Iterations/         ← Per-iteration notes (5-8 lines each)
    │   ├── 2026-03-22/
    │   │   └── *.canvas    ← Session flow visualizations
    │   └── Archive/        ← Smart archival (relevance-scored)
    └── Predictions/        ← Persona notes + debate transcripts
        └── *.canvas        ← Persona debate visualizations
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

## v0.1.1 Features

### Loop Intelligence
- **Strategy Learning** — Analyzes past keep rates by change category across sessions. Biases ideation toward historically successful strategies (>60% keep rate).
- **Predictive Budgeting** — Estimates iterations needed to reach your goal based on past session data (requires 2+ past sessions).
- **Parallel Branches** — When stuck (>8 consecutive discards), forks into 2-3 git worktrees, tries different strategies in parallel, merges the winner.

### New Commands
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

---

## Credits

- **[Andrej Karpathy](https://github.com/karpathy)** — for [autoresearch](https://github.com/karpathy/autoresearch)
- **[Udit Goenka](https://github.com/uditgoenka)** — for [Claude Autoresearch](https://github.com/uditgoenka/autoresearch)
- **[Anthropic](https://anthropic.com)** — for [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

---

## License

MIT — see [LICENSE](LICENSE).
