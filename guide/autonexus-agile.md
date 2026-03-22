# /autonexus:agile — Agile Sprint Lifecycle

Run a full Agile sprint with an autonomous agent team. The orchestrator coordinates specialized agents — each in its own isolated context window — through Obsidian as the shared communication channel.

## What It Does

- **Sprint Planning:** PM prioritizes backlog, Architect assesses, team estimates (Planning Poker), EM assigns work
- **Sprint Execution:** For each of 10 stories: Design → Implement → Code Review → QA → Accept
- **Sprint Review:** PM verifies acceptance, CEO/CPO assesses business value
- **Retrospective:** Metrics, agent reflections, action items for next sprint
- **Full Agile Ceremonies:** Planning Poker, daily standups, sprint review, retrospective

## Quick Start

```bash
# Run a sprint with auto-configured team
/autonexus:agile
Sprint Goal: Deliver user authentication and preferences

# Run with specific team size
/autonexus:agile --team standard
Sprint Goal: Build notification system

# Run specific stories only
/autonexus:agile --stories "STORY-042,STORY-043,STORY-044"

# Resume an interrupted sprint
/autonexus:agile --resume

# Run just the planning ceremony
/autonexus:agile --phase planning

# Dry run (plan without executing)
/autonexus:agile --dry-run
```

## Prerequisites

1. **Obsidian MCP configured** — Run `/autonexus:setup` if not done
2. **Product backlog exists** — Run `/autonexus:refine-backlog` to create stories
3. **Stories are testable** — Each story needs acceptance criteria + verify_command + test_file

## How It Works

### The Orchestrator

The main Claude instance acts as a **pure scheduler** (Scrum Master). It:
- Reads state from Obsidian (sprint board, story notes)
- Decides which agent to spawn next
- Spawns the agent as a **subagent** (separate context window)
- Reads the agent's brief return summary
- Updates the sprint board
- Moves to the next step

The orchestrator NEVER reads source code, writes code, or reviews code. All work is delegated. This keeps the main context window clean throughout the entire sprint.

### Agent Isolation

Every agent runs in its own context window. Agents communicate **only through Obsidian notes** — they never see each other's reasoning or context. This prevents context pollution and keeps each agent focused on its specialty.

### Team Configurations

| Config | Agents | Best For | Capacity |
|--------|--------|----------|----------|
| **Auto** (default) | Scales per story | Mixed complexity | Dynamic |
| **Lean** | 5 agents | Simple features, bug fixes | ~25 pts |
| **Standard** | 12 agents | Typical features | ~50 pts |
| **Full** | 20+ agents | Complex systems, new features | ~80 pts |

### Sprint Flow

```
Sprint Planning → Sprint Execution (x10 stories) → Sprint Review → Retrospective
                  |- Design → Implement → Review → QA → Accept
                  |- Standups every 2-3 stories
```

## Quality Gates

Every story must pass 4 sequential gates:

| Gate | Required | Blocks |
|------|----------|--------|
| **Design Gate** | Architect design + CTO approval | Implementation |
| **Code Gate** | Sr. Engineer review + Security pass | QA testing |
| **QA Gate** | All tests pass + coverage target met | Acceptance |
| **Acceptance Gate** | PM verifies all acceptance criteria | Done |

## Conflict Resolution

When agents disagree, the orchestrator automatically escalates to the appropriate supervisor. The supervisor's decision is recorded as an ADR and is binding.

## Obsidian Structure

```
Projects/{ProjectName}/
|- Backlog/           <- Product backlog (stories + epics)
|- Sprints/sprint-N/  <- Sprint artifacts
|   |- Goal.md        <- Sprint goal + committed stories
|   |- Board.md       <- Kanban board (updated in place)
|   |- Standups/      <- Daily standup notes
|   |- Review.md      <- Sprint review
|   |- Retro.md       <- Retrospective
|- Designs/           <- Technical design docs
|- Reviews/           <- Code review records
|- Team/              <- Team roster + norms
|- Decisions/         <- ADRs from conflict resolution
```

## Flags

| Flag | Purpose |
|------|---------|
| `--team <config>` | Team size: auto, lean, standard, full |
| `--stories "S1,S2"` | Specific stories for this sprint |
| `--sprint N` | Override sprint number |
| `--phase <name>` | Run specific phase: planning, review, retro |
| `--resume` | Resume interrupted sprint |
| `--parallel` | Parallel story execution (worktrees) |
| `--dry-run` | Plan without executing |

## Tips

- **Start with `/autonexus:refine-backlog`** to build your backlog before running a sprint
- **Use `--team lean`** for your first sprint to see how it works with fewer agents
- **Use `--dry-run`** to preview the sprint plan before committing
- **The sprint is resumable** — if interrupted, use `--resume` to pick up where you left off
- **Review sprint board** in Obsidian during execution to see real-time progress

## Integration with Other Commands

The agile workflow composes existing AutoNexus commands:
- Engineer agents internally use the **core autoresearch loop** for implementation
- Code review can optionally invoke **`/autonexus:predict`** for multi-persona analysis
- Bug investigation uses **`/autonexus:debug`** when QA finds unclear bugs
- Documentation updates use **`/autonexus:learn`** during retrospective
- Deployment uses **`/autonexus:ship`** after sprint review

## Example Session

```bash
# Step 1: Create your backlog
/autonexus:refine-backlog --new "User authentication system"
/autonexus:refine-backlog --new "User preferences API"
/autonexus:refine-backlog --new "Settings dashboard"

# Step 2: Run a sprint
/autonexus:agile
Sprint Goal: Core user management features
Team: standard

# Step 3: After sprint, review in Obsidian
# See: Board.md, Review.md, Retro.md
```
