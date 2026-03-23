# Stream Chaining Protocol — AutoNexus

Agent-to-agent output piping. One agent's return becomes the next agent's input directly — no Obsidian round-trip. 40-60% latency reduction vs standard Obsidian handoff.

**IMPORTANT: All agent spawns follow the 7-step cycle in `agent-spawning-protocol.md` (Pattern C: Stream Handoff).**

---

## Handoff Protocol

### Step-by-Step Execution

**Step 1 — Spawn First Agent:**
1. Load agent definition from `references/agents/{category}/{role}.md`
2. Build prompt: agent definition + task + scope
3. CRITICAL DIFFERENCE: Request FULL structured output (max 500 tokens), NOT 2-3 sentence summary
4. Spawn: `Agent(prompt=above, model={model}, description="{role} — {task}")`
5. Capture FULL return as `stream_context_1`

**Step 2 — Spawn Second Agent (with stream context):**
1. Load next agent definition
2. Build prompt:
   - Agent definition + task
   - Insert stream context block:
     ```
     --- STREAM CONTEXT FROM PREVIOUS AGENT ---
     Agent: {agent_1_name} ({agent_1_role})
     Task: {original task}
     {stream_context_1 verbatim}
     --- END STREAM CONTEXT ---
     ```
   - "Use the above context from the previous agent as your starting point."
   - Request FULL structured output (max 500 tokens)
3. Spawn: `Agent(prompt=above, model={model}, description="{role} — {task}")`
4. Capture return as `stream_context_2`

**Step 3+ — Continue Chain (accumulate all contexts):**
1. Load next agent definition
2. Build prompt with ALL prior stream contexts:
   ```
   --- STREAM CONTEXT FROM AGENT 1 ---
   Agent: {name}
   {stream_context_1}
   --- END STREAM CONTEXT ---

   --- STREAM CONTEXT FROM AGENT 2 ---
   Agent: {name}
   {stream_context_2}
   --- END STREAM CONTEXT ---
   ```
3. FINAL agent in chain: Also write to Obsidian for persistence

---

## Key Difference from Obsidian Communication

| Aspect | Obsidian Mode | Stream Mode |
|--------|--------------|-------------|
| Agent return | 2-3 sentence summary | Full structured output (max 500 tokens) |
| Context passing | Via Obsidian read/write | Via prompt injection |
| Persistence | Every step persisted | Only final step persisted |
| Latency | +2-5s per step (Obsidian round-trip) | Near-zero between steps |
| Resumability | Can resume from any step | Must restart chain if interrupted |
| Context growth | Constant (agents read from Obsidian) | Grows with each step (accumulated contexts) |

**Use Stream when:** Speed matters, chain is short (2-5 agents), no need to resume mid-chain.
**Use Obsidian when:** Persistence matters, chain is long, need cross-session access to intermediate results.

---

## Predefined Pipelines

### Analysis Pipeline
```
scout-explorer → security-analyst → sr-qa-eng
```
1. Scout explores codebase, returns file map + risks
2. Security analyst receives scout context, returns vulnerabilities
3. Sr. QA receives both contexts, writes test plan to Obsidian

### Refactor Pipeline
```
architect → sr-fullstack-eng → code-review-swarm → sr-qa-eng
```
1. Architect designs refactoring approach, returns architecture decisions
2. Sr. Full Stack implements based on architect context, returns changes summary
3. Code Review Swarm reviews with architect + implementer context, returns findings
4. Sr. QA creates regression tests from all prior context, writes to Obsidian

### Debug Pipeline
```
scout-explorer → sr-backend-eng → sr-qa-eng → qa-manager
```
1. Scout explores failure area, returns context package
2. Sr. Backend diagnoses with scout context, returns root cause + fix
3. Sr. QA verifies fix with diagnosis context, returns test results
4. QA Manager triages with all context, writes report to Obsidian

### Optimize Pipeline
```
perf-monitor → topology-optimizer → sr-backend-eng → benchmark-runner
```
1. Performance Monitor profiles, returns bottleneck analysis
2. Topology Optimizer recommends approach from perf context
3. Sr. Backend implements optimization with both contexts
4. Benchmark Runner measures improvement, writes results to Obsidian

### SPARC Pipeline (stream variant)
```
sparc-spec → sparc-pseudo → sparc-arch → sparc-refine → sparc-complete
```
Same as SPARC methodology but with stream handoffs instead of Obsidian between phases. Faster but no intermediate persistence. Final completion report writes to Obsidian.

---

## Structured Return Format

Agents in stream mode return structured output for easy parsing by next agent:

```
FINDINGS:
- {finding 1}
- {finding 2}

FILES_ANALYZED: {list}
FILES_MODIFIED: {list}
RISKS: {list}
RECOMMENDATIONS: {list}
STATUS: {complete|partial|failed}
CONFIDENCE: {0-100}
```

Max 500 tokens per agent return. If an agent's output would exceed 500 tokens, it must summarize and write the full output to Obsidian.

---

## Integration Points
- Called from: `/autonexus:stream`, chain workflow with `--stream` flag
- Compatible with: swarm worker chains, SPARC phases, agile story lifecycle
- Agent spawning follows: `agent-spawning-protocol.md` Pattern C
- Final results persist to: `Projects/{project}/Streams/{chain-id}.md`
