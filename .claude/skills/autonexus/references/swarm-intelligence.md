# Swarm Intelligence Protocol — AutoNexus

Ruflo-inspired multi-agent swarm coordination. When tasks exceed single-agent capacity, swarm mode coordinates multiple agents through a queen hierarchy with configurable topologies and consensus algorithms.

## Queen Hierarchy

| Queen | Tier | Model | Activated When | Responsibility |
|-------|------|-------|----------------|----------------|
| Strategic Queen | 2 | opus | Complexity > 20 | Goal decomposition, resource allocation, topology selection |
| Tactical Queen | 3 | sonnet | Standard swarms | Task routing, progress monitoring, bottleneck detection |
| Adaptive Queen | 3 | sonnet | Throughput drops >40% | Dynamic scaling, topology switching, load rebalancing |

## Worker Types

| Type | Agent | Role |
|------|-------|------|
| Builder | sr-backend-eng, sr-frontend-eng | Implement code changes |
| Scout | scout-explorer | Explore codebase, gather context (read-only) |
| Reviewer | code-review-swarm, sr-qa-eng | Review and validate |
| Memory Manager | swarm-memory-mgr | Obsidian reads/writes |
| Optimizer | perf-monitor, benchmark-runner | Performance analysis |
| Specialist | dba, security-analyst, ai-defense | Domain-specific tasks |

## Topologies (5)

| Topology | Structure | Best For | Anti-Drift Risk |
|----------|-----------|----------|-----------------|
| Hierarchical (default) | Queen -> Workers | Coding tasks | LOW |
| Mesh | All connected | Research/exploration | MEDIUM |
| Hierarchical-Mesh | Queen -> Clusters (mesh within) | Complex multi-domain | LOW |
| Ring | Sequential handoff | Pipeline processing | VERY LOW |
| Star | Central hub + spokes | Parallel independent | LOW |

## Consensus Algorithms (5)

| Algorithm | Mechanism | Fault Tolerance | Use When |
|-----------|-----------|-----------------|----------|
| Raft (default) | Leader election, majority vote | f < n/2 | Authoritative coding decisions |
| Byzantine (BFT) | Multi-round verification | n/3 faulty | Conflicting outputs |
| Gossip | Random exchange, eventual convergence | High | Large swarm knowledge sharing |
| CRDT | Commutative merge operations | Full | Concurrent shared state edits |
| Quorum | Tunable R+W>N | Configurable | Specific consistency needs |

## Swarm Activation Protocol

**IMPORTANT: All agent spawns follow the 7-step cycle in `agent-spawning-protocol.md`. Read that first.**

### Phase 0 — Assessment
Score task complexity (5 dimensions from three-tier-routing.md). Score > 20 or multi-domain activates swarm.

### Phase 1 — Configuration
Select topology, queen, maxAgents (default 6), consensus (default raft), anti-drift level.

### Phase 2 — Decomposition (Spawn Queen)
1. Load agent definition: `references/agents/swarm/strategic-queen.md` (score > 20) or `tactical-queen.md` (score <= 20)
2. Build prompt per agent-spawning-protocol.md Step 3:
   - Include full queen definition
   - "YOUR ASSIGNMENT: Decompose this goal into max {maxAgents} parallelizable subtasks."
   - "Goal: {goal}. Scope: {scope}. Topology: {topology}."
   - "OUTPUT: Write subtask list to Projects/{project}/Swarm/strategy.md"
   - "Format each subtask: title, scope (files), worker type (builder|scout|reviewer|specialist), dependencies, acceptance criteria"
   - "RETURN: Subtask count and critical path."
3. Spawn: `Agent(prompt=above, model="opus"|"sonnet", description="Queen — decompose")`
4. Read Projects/{project}/Swarm/strategy.md — parse subtask list

### Phase 3 — Execution (Spawn Workers in PARALLEL)
For each subtask from strategy.md:
1. Map worker type to agent from roster:
   - builder -> sr-backend-eng, sr-frontend-eng, or fullstack-eng (match domain)
   - scout -> scout-explorer
   - reviewer -> code-review-swarm
   - specialist -> match domain (dba, security-analyst, etc.)
2. Load agent definition from `references/agents/{category}/{role}.md`
3. Build prompt per agent-spawning-protocol.md:
   - Include agent definition
   - "YOUR ASSIGNMENT: {subtask description from strategy.md}"
   - "CONTEXT: Read Projects/{project}/Swarm/strategy.md for full context"
   - "OUTPUT: Write results to Projects/{project}/Swarm/workers/subtask-{N}.md"
   - "Include: what you changed, files modified, test results, issues found"
   - "RETURN: 2-3 sentence summary."
4. Spawn ALL independent workers in a SINGLE message (parallel):
   `Agent(prompt=worker1, model="sonnet", description="Builder — subtask 1")`
   `Agent(prompt=worker2, model="sonnet", description="Builder — subtask 2")`
   `Agent(prompt=worker3, model="sonnet", description="Scout — subtask 3")`

**Anti-drift checkpoint every 3 completions:**
1. Read original goal statement
2. Read all completed worker outputs from Projects/{project}/Swarm/workers/
3. Compare outputs against goal and approved file scope
4. Check git diff for out-of-scope modifications
5. IF drift detected: pause, respawn queen with "Workers drifted. Re-align: {issues}"

### Phase 4 — Consensus (Spawn Collective Intelligence)
1. Load `references/agents/swarm/collective-intelligence.md`
2. Build prompt:
   - "CONTEXT: Read ALL notes in Projects/{project}/Swarm/workers/"
   - "Consensus algorithm: {raft|bft}"
   - "OUTPUT: Write unified result to Projects/{project}/Swarm/consensus/result.md"
   - "RETURN: Unified recommendation with confidence level."
3. Spawn: `Agent(prompt=above, model="sonnet", description="Consensus — synthesize")`
4. If deadlock (no consensus after 2 rounds): escalate to queen, then CTO if queen fails.

### Phase 5 — Finalization
Write swarm summary to Obsidian. Fire swarm-complete event. Report to user.

## Dynamic Topology Switching

Adaptive Queen monitors throughput. May switch: hierarchical->mesh (need sharing), mesh->hierarchical (need coordination), any->star (independent tasks), any->ring (sequential). Max 1 switch per 5 completions.

## Obsidian Layout

Projects/{ProjectName}/Swarm/ — swarm state, strategy, progress, consensus/, workers/, scouts/

## Integration Points

- Called from: /autonexus:swarm, /autonexus:agile (complex stories), main loop (>8 discards)
- Fires events: swarm-activated, swarm-consensus-reached, swarm-drift-detected, swarm-complete
- Uses: anti-drift-protocol.md, claims-authorization.md, three-tier-routing.md
