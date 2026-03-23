# /autonexus:swarm — Multi-Agent Swarm Intelligence

Decomposes a complex task across multiple agents working in parallel, with configurable topology and consensus mechanisms. A queen agent breaks the task down, worker agents execute sub-tasks concurrently, and a consensus step synthesizes results. Best for tasks that span multiple domains or are too complex for a single agent.

## Syntax

```
# Basic swarm with defaults
/autonexus:swarm Refactor payment system for multi-currency

# Hierarchical topology with agent cap
/autonexus:swarm Refactor payment system for multi-currency --topology hierarchical --max-agents 6

# Mesh topology for research tasks
/autonexus:swarm Research SOTA approaches for real-time fraud detection --topology mesh

# Specify the queen agent
/autonexus:swarm Modernize CI/CD pipeline --queen devops-lead

# Enable anti-drift checks
/autonexus:swarm Build microservice architecture --anti-drift 3

# Choose consensus strategy
/autonexus:swarm Optimize database performance --consensus voting --max-agents 4
```

## What It Does

1. **Queen decomposes** — The queen agent analyzes the task, identifies sub-domains, and creates a work breakdown structure with dependencies.
2. **Workers execute** — Worker agents are spawned for each sub-task. They execute concurrently within the chosen topology, sharing findings through the coordination layer.
3. **Anti-drift checks** — If enabled, every N completions the queen reviews worker outputs against the original goal to prevent scope drift.
4. **Consensus synthesizes** — All worker outputs are collected and the consensus mechanism produces a unified result: merged code, combined report, or voted recommendation.

## Topologies

| Topology | Structure | Best For |
|----------|-----------|----------|
| `hierarchical` | Queen → Workers → Queen (default) | Most development tasks, clear sub-task boundaries |
| `mesh` | All agents communicate peer-to-peer | Research, exploration, brainstorming |
| `pipeline` | Agent A → Agent B → Agent C | Sequential processing, data transformation |
| `star` | Central coordinator with specialist spokes | Tasks needing a single integration point |

## Consensus Strategies

| Strategy | How It Works |
|----------|--------------|
| `merge` | Queen merges all worker outputs into a unified result (default) |
| `voting` | Workers vote on competing approaches, majority wins |
| `rank` | Workers rank alternatives, best aggregate rank wins |
| `review` | Each worker reviews others' work, queen resolves conflicts |

## Flags

| Flag | Purpose |
|------|---------|
| `--topology <type>` | Set the swarm topology (hierarchical, mesh, pipeline, star) |
| `--consensus <strategy>` | Set the consensus mechanism (merge, voting, rank, review) |
| `--max-agents <n>` | Maximum number of worker agents to spawn (default: 4) |
| `--queen <agent>` | Override which agent serves as queen/coordinator |
| `--anti-drift <n>` | Run drift check every N worker completions (default: off) |

## Examples

```bash
# Hierarchical swarm for a multi-domain refactor
/autonexus:swarm Refactor payment system for multi-currency --topology hierarchical --max-agents 6

# Queen decomposition:
#   ✓ Queen (CTO): Identified 5 sub-tasks across 3 domains
#     1. Currency model + exchange rates (Backend Agent)
#     2. Payment gateway abstraction (Backend Agent)
#     3. Multi-currency UI components (Frontend Agent)
#     4. Database schema migration (Database Agent)
#     5. Integration tests (QA Agent)
#
# Worker execution:
#   ✓ Backend Agent 1: Currency model — 4 files, 180 lines
#   ✓ Backend Agent 2: Gateway abstraction — 3 files, 220 lines
#   ✓ Frontend Agent: UI components — 6 files, 310 lines
#   ✓ Database Agent: Migration scripts — 2 files, 85 lines
#   ✓ QA Agent: Integration tests — 3 files, 190 lines
#   ⟳ Anti-drift check at completion 3/5: on track
#
# Consensus (merge):
#   ✓ Queen merged all outputs — 18 files, 985 lines total
#   ✓ No conflicts detected
#   ✓ All 14 integration tests passing

# Mesh topology for research
/autonexus:swarm Research SOTA approaches for real-time fraud detection --topology mesh --max-agents 4

# Voting consensus when agents disagree
/autonexus:swarm Choose the best state management solution --consensus voting --max-agents 3
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Swarm strategy | `Projects/{ProjectName}/Swarm/strategy.md` | After queen decomposes |
| Worker reports | `Projects/{ProjectName}/Swarm/workers/{agent}-{task}.md` | After each worker completes |
| Drift check logs | `Projects/{ProjectName}/Swarm/drift-checks/{n}.md` | After each anti-drift check |
| Consensus result | `Projects/{ProjectName}/Swarm/consensus.md` | After consensus synthesizes |

### What Gets Read
- Project context from `Projects/{ProjectName}/Sessions/`
- Prior swarm results if re-running

## Without Obsidian

- Queen decomposition, worker execution, and consensus all work normally
- No strategy/worker/consensus documents are persisted
- Context passing between agents uses in-memory state
- Anti-drift checks still function

## Tips

- **Start with hierarchical topology** — it works well for most development tasks and gives the queen clear control over work distribution.
- **Use mesh for research** — when you need agents to cross-pollinate ideas, mesh lets them share findings directly.
- **Anti-drift checks every 3 completions** is a good default. Without them, workers on long tasks can diverge from the original goal.
- **Max 8 agents for most tasks** — beyond that, coordination overhead outweighs parallelism benefits. Start with 4 and increase if needed.
- **Pair with `/autonexus:sparc`** — use SPARC for the specification phase, then hand the spec to a swarm for parallel implementation.
- **Use `--consensus voting`** when there are genuinely competing approaches and you want the agents to evaluate each other's work.
