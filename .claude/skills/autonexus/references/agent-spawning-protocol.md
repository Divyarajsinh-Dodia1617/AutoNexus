# Agent Spawning Protocol — AutoNexus

Universal protocol for spawning agents in ANY AutoNexus workflow. Every agent spawn — whether in agile sprints, swarm coordination, SPARC phases, or stream chains — MUST follow this 7-step cycle.

## The 7-Step Spawning Cycle

### Step 1 — Read Current State from Obsidian

Before spawning, read current state so you know what context the agent needs.

### Step 2 — Load Agent Definition

Read the agent definition from `references/agents/{category}/{role}.md`. Extract: model (from frontmatter), identity, capabilities, constraints, prompt template.

### Step 3 — Construct the Agent Prompt

Build the full prompt. TEMPLATE:

```
{Full agent definition content from the .md file}

YOUR ASSIGNMENT:
{Specific task description with concrete deliverable}

CONTEXT — Read these from Obsidian:
- Projects/{project}/{path1} — {what to find there}
- Projects/{project}/{path2} — {what to find there}

OUTPUT — Write your results to:
- Projects/{project}/{output_path} — {what to write there}
- Use obsidian_update_note with mode: overwrite

RETURN TO ME: 2-3 sentence summary only.
Do NOT return full analysis — write that to Obsidian.
```

CRITICAL RULES:
- ALWAYS include explicit Obsidian READ paths (what context the agent needs)
- ALWAYS include explicit Obsidian WRITE path (where agent puts its work)
- ALWAYS request "2-3 sentence summary" as return (keep orchestrator context clean)
- NEVER dump large file contents into prompt — give Obsidian paths, let agent read them
- INCLUDE the agent's constraints section so it stays in lane

### Step 4 — Spawn the Agent

Use the Agent tool with these parameters:

```
Agent(
  prompt = {constructed prompt from Step 3},
  model = {model from agent definition frontmatter: "opus"|"sonnet"|"haiku"},
  description = "{Agent Name} — {brief task description}"
)
```

For parallel spawns (independent agents that do NOT depend on each other), launch multiple Agent calls in a SINGLE message.

### Step 5 — Receive Brief Return

The agent returns a 2-3 sentence summary. This is ALL that enters the orchestrator context. All detailed work products stay in Obsidian.

### Step 6 — Read Updated Obsidian State

After agent completes, read what it wrote to Obsidian. This is how the orchestrator sees the agent's full output without bloating context.

### Step 7 — Update State Machine and Transition

Update tracking (sprint board, swarm progress, SPARC phase). Determine next agent. Evaluate gates if needed.

---

## Obsidian as Agent Communication Bus

Agents communicate ONLY through Obsidian notes. They never talk directly to each other.

```
Orchestrator spawns Agent A with: read X, write Y
  Agent A reads X from Obsidian
  Agent A does work
  Agent A writes Y to Obsidian
  Agent A returns 2-3 sentence summary
Orchestrator reads Y from Obsidian
Orchestrator spawns Agent B with: read Y, write Z
  Agent B reads Y (this is Agent A's output)
  Agent B does work
  Agent B writes Z
  Agent B returns summary
```

### Path Conventions for Inter-Agent Communication

| Workflow | Agents Read From | Agents Write To |
|----------|-----------------|-----------------|
| Swarm | Projects/{project}/Swarm/strategy.md | Projects/{project}/Swarm/workers/{worker-id}.md |
| SPARC | Projects/{project}/SPARC/{feature}/{prev-phase}.md | Projects/{project}/SPARC/{feature}/{current-phase}.md |
| Stream | No Obsidian — direct return handoff | Projects/{project}/Streams/{chain-id}.md (final only) |
| Agile | Projects/{project}/Sprints/sprint-{N}/Board.md | Projects/{project}/Sprints/sprint-{N}/{artifact}.md |
| Security | Projects/{project}/Security/scope.md | Projects/{project}/Security/findings/{id}.md |
| Debug | Projects/{project}/Debug/{session}/hypotheses.md | Projects/{project}/Debug/{session}/findings/{id}.md |

---

## Spawning Pattern A: Sequential Chain (SPARC, Debug, Fix)

Each agent's Obsidian output feeds the next agent's input.

### Concrete SPARC Example

**Phase S — Specification:**
1. Load `references/agents/sparc/specification-agent.md`
2. Build prompt: agent definition + goal + scope + "Write to Projects/{project}/SPARC/{feature}/Specification.md"
3. `Agent(prompt=above, model="sonnet", description="SPARC Spec — {feature}")`
4. Receive: "Defined 8 requirements with 24 acceptance criteria."
5. Read Specification.md from Obsidian
6. Gate: All criteria measurable? If not, respawn with: "Issues: {list}. Please revise."

**Phase P — Pseudocode:**
1. Load `references/agents/sparc/pseudocode-agent.md`
2. Build prompt: agent definition + "CONTEXT: Read Projects/{project}/SPARC/{feature}/Specification.md" + "Write to Pseudocode.md"
3. `Agent(prompt=above, model="sonnet", description="SPARC Pseudo — {feature}")`
4. Gate: All requirements covered?

**Phase A — Architecture:**
1. Load `references/agents/sparc/architecture-agent.md`
2. Build prompt: agent definition + "CONTEXT: Read Specification.md AND Pseudocode.md" + "Write to Architecture.md"
3. `Agent(prompt=above, model="opus", description="SPARC Arch — {feature}")`
4. Gate: Spawn CTO agent to review Architecture.md. CTO approves or rejects.

**Phase R — Refinement:**
1. Load `references/agents/sparc/refinement-agent.md`
2. Build prompt: agent definition + "Read Architecture.md. Run mini autoresearch loop until tests pass. Write to Refinement-Log.md"
3. `Agent(prompt=above, model="sonnet", description="SPARC Refine — {feature}")`
4. Gate: All acceptance criteria from Specification.md pass?

**Phase C — Completion:**
1. Load `references/agents/sparc/completion-agent.md`
2. Build prompt: agent definition + "Read ALL SPARC/{feature}/ files. Verify coverage, docs, security. Write to Completion.md"
3. `Agent(prompt=above, model="sonnet", description="SPARC Complete — {feature}")`
4. Gate: Full sign-off?

---

## Spawning Pattern B: Hierarchical Decompose-and-Delegate (Swarm)

Queen decomposes, workers execute in parallel, consensus synthesizes.

### Concrete Swarm Example

**Phase 2 — Queen Decomposition:**
1. Load `references/agents/swarm/strategic-queen.md` (or tactical-queen.md for score <= 20)
2. Build prompt:
   - Agent definition
   - "YOUR ASSIGNMENT: Decompose this goal into max {maxAgents} parallelizable subtasks."
   - "Goal: {goal}. Scope: {scope}. Topology: {topology}."
   - "OUTPUT: Write subtask list to Projects/{project}/Swarm/strategy.md"
   - "Format each subtask with: title, scope (files), worker type, dependencies, acceptance criteria"
3. `Agent(prompt=above, model="opus", description="Strategic Queen — decompose")`
4. Read Projects/{project}/Swarm/strategy.md — parse subtask list

**Phase 3 — Worker Spawning (PARALLEL):**
For each subtask from strategy.md:
1. Map worker type to agent: builder->sr-backend-eng, scout->scout-explorer, reviewer->code-review-swarm
2. Load agent definition
3. Build prompt:
   - Agent definition
   - "YOUR ASSIGNMENT: {subtask description from strategy.md}"
   - "CONTEXT: Read Projects/{project}/Swarm/strategy.md for full context"
   - "OUTPUT: Write results to Projects/{project}/Swarm/workers/subtask-{N}.md"
4. Spawn ALL independent workers in a SINGLE message:
   - `Agent(prompt=worker1, model="sonnet", description="Builder — subtask 1")`
   - `Agent(prompt=worker2, model="sonnet", description="Builder — subtask 2")`
   - `Agent(prompt=worker3, model="sonnet", description="Scout — subtask 3")`

**Phase 3.5 — Anti-Drift Check (every 3 completions):**
1. Read original goal
2. Read all worker outputs from Projects/{project}/Swarm/workers/
3. Compare against goal and approved scope
4. If drift: pause, respawn queen to re-align

**Phase 4 — Consensus:**
1. Load `references/agents/swarm/collective-intelligence.md`
2. Build prompt:
   - Agent definition
   - "CONTEXT: Read ALL notes in Projects/{project}/Swarm/workers/"
   - "Algorithm: {raft|bft}"
   - "OUTPUT: Write to Projects/{project}/Swarm/consensus/result.md"
3. `Agent(prompt=above, model="sonnet", description="Consensus — synthesize")`

---

## Spawning Pattern C: Stream Handoff (Stream Chaining)

Agent A's full return injected directly into Agent B's prompt. No Obsidian round-trip between steps.

### Concrete Stream Example (Analysis Pipeline)

**Step 1 — Scout:**
1. Load `references/agents/swarm/scout-explorer.md`
2. Build prompt: agent definition + task + "RETURN structured findings (max 500 tokens): FILES, DEPENDENCIES, RISKS, RECOMMENDATIONS"
3. `Agent(prompt=above, model="sonnet", description="Scout — explore")`
4. Capture FULL return (not just summary) as stream_context_1

**Step 2 — Security Analyst (with stream context):**
1. Load `references/agents/security/security-analyst.md`
2. Build prompt:
   - Agent definition + task
   - "--- STREAM CONTEXT FROM PREVIOUS AGENT ---"
   - "Agent: Scout Explorer"
   - "{stream_context_1 verbatim}"
   - "--- END STREAM CONTEXT ---"
   - "RETURN structured findings: VULNERABILITIES, RISK_SCORE, RECOMMENDATIONS"
3. `Agent(prompt=above, model="sonnet", description="Security — scan")`
4. Capture return as stream_context_2

**Step 3 — Sr. QA (with accumulated context):**
1. Load `references/agents/quality/sr-qa-engineer.md`
2. Build prompt:
   - Agent definition + task
   - "--- STREAM CONTEXT FROM AGENT 1 ---"
   - "{stream_context_1}"
   - "--- STREAM CONTEXT FROM AGENT 2 ---"
   - "{stream_context_2}"
   - "OUTPUT: Write test plan to Projects/{project}/Streams/{chain-id}.md"
3. `Agent(prompt=above, model="sonnet", description="Sr. QA — test plan")`

KEY DIFFERENCE: In stream mode, agents return FULL structured output (up to 500 tokens) instead of 2-3 sentence summary. Only the FINAL agent writes to Obsidian.

---

## Gate Enforcement Protocol

Gates are evaluated by the ORCHESTRATOR, not by agents.

After agent completes:
1. Read agent output from Obsidian
2. Evaluate gate criteria
3. IF passed: proceed
4. IF failed AND attempt < max_retries: Respawn SAME agent with feedback
5. IF failed AND attempt >= max_retries: Escalate per escalation chain

| Gate | Criteria | Max Retries | Escalation |
|------|----------|-------------|------------|
| Design Gate | Architecture approved by CTO | 2 | Principal Engineer |
| Code Gate | Tests pass + review approved | 3 | Staff Engineer (binding) |
| QA Gate | 0 P0/P1 bugs, all ACs covered | 3 | EM decides fix/defer |
| Acceptance | PM confirms all ACs | 1 | CPO |
| SPARC Phase | Phase output complete and criteria met | 2 | Next tier per chain |

---

## Claims Integration

Before spawning any agent:
1. Agent tier (from frontmatter) determines security level
2. Tier 9=Minimal, 7-8=Standard, 5-6=Elevated, 1-3=Admin
3. Include in prompt: "You have permission to modify only: {scope}"
4. After return: claims auto-revoke

---

## Error Recovery

**Agent crashes:** Retry < 3 times. Then escalate. Then flag for human.
**Invalid output:** Respawn with feedback. Max 2 corrections. Then escalate.
**Obsidian write fails:** Agent still returns summary. Orchestrator queues write. Workflow continues.

---

## Integration Points

This protocol is referenced by ALL workflow protocols:
- agile-workflow.md (already follows this pattern)
- swarm-intelligence.md (Pattern B: Hierarchical)
- sparc-methodology.md (Pattern A: Sequential Chain)
- stream-chaining.md (Pattern C: Stream Handoff)
- chain-workflow.md (combines A + C)
- hook-workflow.md (run-command actions spawn via this protocol)
- background-workers.md (workers spawned with minimal claims)
