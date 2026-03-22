# AutoNexus

**Autonomous goal-directed iteration engine with Obsidian as knowledge backbone.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [Claude Autoresearch](https://github.com/uditgoenka/autoresearch). AutoNexus adds persistent cross-session knowledge via Obsidian MCP — every iteration reads from and writes to your Obsidian vault.

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blue?logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)]()
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

---

## Quick Start

### 1. Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [Obsidian](https://obsidian.md) desktop app
- [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) Obsidian community plugin enabled

### 2. Configure Obsidian MCP

Add to your Claude Code MCP config (`.mcp.json`):

```json
{
  "mcpServers": {
    "obsidian-mcp-server": {
      "command": "npx",
      "args": ["obsidian-mcp-server"],
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

### 3. Install AutoNexus

```bash
git clone https://github.com/Divyarajsinh-Dodia1617/AutoNexus.git

# Copy to your project
cp -r autonexus/claude-plugin/skills/autonexus .claude/skills/autonexus
cp -r autonexus/claude-plugin/commands/autonexus .claude/commands/autonexus
cp autonexus/claude-plugin/commands/autonexus.md .claude/commands/autonexus.md
```

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
    ├── Knowledge/          ← Codebase model (updated pre-push)
    │   ├── Architecture.md
    │   ├── Components.md
    │   └── Dependencies.md
    ├── Decisions/          ← ADR-format decision records
    ├── Iterations/         ← Per-iteration notes (5-8 lines each)
    │   ├── 2026-03-22/
    │   └── Archive/        ← Auto-archived after 30 days
    └── Predictions/        ← Persona notes + debate transcripts
Cross-Project/              ← Patterns across all projects
```

---

## Key Design Decisions

- **Obsidian is best-effort:** Every MCP call uses retry-then-queue. If Obsidian is down, the loop continues with git + TSV. No data lost.
- **Iteration notes are concise:** 5-8 lines each. What was tried, result, why, next suggestion.
- **Archive is searchable:** 30-day auto-archive moves old notes but Obsidian search still indexes them.
- **Pre-push rebuilds knowledge:** Full codebase scan before every push ensures Knowledge Center is current.
- **Predict personas persist:** Each persona's analysis becomes a permanent, searchable Obsidian note.

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
