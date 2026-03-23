---
name: autonexus:swarm
description: Swarm intelligence orchestration — multi-agent coordination with queen hierarchy, configurable topologies, and consensus algorithms.
argument-hint: "[task description] [--topology <hierarchical|mesh|ring|star|adaptive>] [--consensus <raft|bft|gossip|crdt|quorum>] [--max-agents <N>] [--queen <strategic|tactical|adaptive>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--topology <type>` or `Topology:` — swarm topology (hierarchical|mesh|ring|star|adaptive). Default: hierarchical
- `--consensus <algo>` or `Consensus:` — consensus algorithm (raft|bft|gossip|crdt|quorum). Default: raft
- `--max-agents <N>` or `MaxAgents:` — maximum concurrent agents. Default: 6
- `--queen <type>` or `Queen:` — queen type (strategic|tactical|adaptive). Default: auto-selected based on complexity
- `--anti-drift <strict|standard|relaxed>` or `Drift:` — drift checking frequency. Default: standard
- `--scope <glob>` or `Scope:` — file scope for the swarm task

Remaining unmatched text = task description for the swarm.

## Execution

1. Read the swarm intelligence protocol: `.claude/skills/autonexus/references/swarm-intelligence.md`
2. Read the anti-drift protocol: `.claude/skills/autonexus/references/anti-drift-protocol.md`
3. Read the agent roster: `.claude/skills/autonexus/references/agents/index.md`
4. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
5. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
6. If no task provided → use `AskUserQuestion`:
   - Question 1 (Task): "What task should the swarm tackle?" with custom input
   - Question 2 (Scope): "Which files/modules are in scope?" with detected options
   - Question 3 (Topology): "Which swarm topology?" with [Hierarchical (Recommended), Mesh, Ring, Star, Adaptive]
   - Question 4 (Size): "How many max concurrent agents?" with [4, 6 (Recommended), 8]
7. Assess task complexity → select queen type if not specified:
   - Score ≤ 20 → Tactical Queen (sonnet)
   - Score > 20 → Strategic Queen (opus)
8. Spawn queen agent with task decomposition mandate
9. Queen returns subtask list → validate subtask count ≤ maxAgents
10. Spawn workers per subtask (using configured topology)
11. Workers execute with anti-drift checkpoints every 3 completions
12. Collect results via configured consensus algorithm
13. Synthesize final output
14. Write swarm summary to Obsidian: `Projects/{ProjectName}/Swarm/`
15. Fire `swarm-complete` event

Stream all output live — never run in background.
