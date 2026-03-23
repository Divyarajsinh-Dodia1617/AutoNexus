# SPARC Methodology — AutoNexus

Structured 5-phase development workflow. Specification -> Pseudocode -> Architecture -> Refinement -> Completion with dedicated agents at each phase and Obsidian communication between them.

**IMPORTANT: All agent spawns follow the 7-step cycle in `agent-spawning-protocol.md`. Read that first.**

---

## Phase S — Specification

**Agent:** sparc-spec (sonnet) from `references/agents/sparc/specification-agent.md`

**Spawning:**
1. Load agent definition from `references/agents/sparc/specification-agent.md`
2. Build prompt:
   - Full agent definition
   - "YOUR ASSIGNMENT: Transform this goal into testable specifications."
   - "Goal: {goal}. Scope: {scope}."
   - "OUTPUT: Write to Projects/{project}/SPARC/{feature}/Specification.md"
   - "Include: requirements list, acceptance criteria (Given/When/Then), constraints, mechanical success metrics"
   - "RETURN: Requirements count, constraints identified, clarifications needed."
3. Spawn: `Agent(prompt=above, model="sonnet", description="SPARC Spec — {feature}")`
4. Read Specification.md from Obsidian

**Gate:** All criteria measurable?
- YES: Proceed to Phase P
- NO: Respawn sparc-spec with: "Issues: {unmeasurable criteria}. Please revise." (max 2 retries, then escalate to PM)

---

## Phase P — Pseudocode

**Agent:** sparc-pseudo (sonnet) from `references/agents/sparc/pseudocode-agent.md`

**Spawning:**
1. Load agent definition
2. Build prompt:
   - Full agent definition
   - "YOUR ASSIGNMENT: Design pseudocode from specification."
   - "CONTEXT: Read Projects/{project}/SPARC/{feature}/Specification.md"
   - "OUTPUT: Write to Projects/{project}/SPARC/{feature}/Pseudocode.md"
   - "Include: algorithm choices with justification, data structures, error paths, complexity analysis"
   - "RETURN: Algorithm count, coverage of requirements, complexity summary."
3. Spawn: `Agent(prompt=above, model="sonnet", description="SPARC Pseudo — {feature}")`
4. Read Pseudocode.md from Obsidian

**Gate:** All requirements from Specification.md covered?
- YES: Proceed to Phase A
- NO: Respawn with feedback (max 2 retries)

---

## Phase A — Architecture

**Agent:** sparc-arch (opus) from `references/agents/sparc/architecture-agent.md`

**Spawning:**
1. Load agent definition
2. Build prompt:
   - Full agent definition
   - "YOUR ASSIGNMENT: Design system architecture from spec and pseudocode."
   - "CONTEXT: Read these from Obsidian:"
   - "  - Projects/{project}/SPARC/{feature}/Specification.md"
   - "  - Projects/{project}/SPARC/{feature}/Pseudocode.md"
   - "  - Projects/{project}/Knowledge/Architecture.md (existing system)"
   - "OUTPUT: Write to Projects/{project}/SPARC/{feature}/Architecture.md"
   - "Also write ADRs to Projects/{project}/Decisions/ for each significant decision"
   - "RETURN: Component count, interface count, ADR count."
3. Spawn: `Agent(prompt=above, model="opus", description="SPARC Arch — {feature}")`
4. Read Architecture.md from Obsidian

**Gate:** CTO approval required.
1. Load `references/agents/executive/cto.md`
2. Build prompt: CTO definition + "Review architecture at Projects/{project}/SPARC/{feature}/Architecture.md. Approve or reject with rationale. Write decision to Architecture.md under 'CTO Review'."
3. Spawn: `Agent(prompt=above, model="opus", description="CTO — architecture review")`
4. Read CTO decision from Architecture.md
5. IF "Approved": Proceed to Phase R
6. IF "Rejected": Respawn sparc-arch with CTO feedback (max 2 cycles)

---

## Phase R — Refinement

**Agent:** sparc-refine (sonnet) from `references/agents/sparc/refinement-agent.md`

**Spawning:**
1. Load agent definition
2. Build prompt:
   - Full agent definition
   - "YOUR ASSIGNMENT: Implement the architecture. Run a mini autoresearch loop until all acceptance criteria pass."
   - "CONTEXT: Read these from Obsidian:"
   - "  - Projects/{project}/SPARC/{feature}/Architecture.md (what to build)"
   - "  - Projects/{project}/SPARC/{feature}/Specification.md (acceptance criteria)"
   - "Verify command: {verify_command}"
   - "Iterate: modify -> verify -> keep/discard. Max {budget} iterations."
   - "OUTPUT: Write iteration log to Projects/{project}/SPARC/{feature}/Refinement-Log.md"
   - "RETURN: Iterations completed, criteria passed, criteria remaining."
3. Spawn: `Agent(prompt=above, model="sonnet", description="SPARC Refine — {feature}")`

**For complex features:** Use `/autonexus:swarm` instead of single agent for Phase R.

**Gate:** All acceptance criteria from Specification.md pass?
- YES: Proceed to Phase C
- NO: Continue refinement or escalate

---

## Phase C — Completion

**Agent:** sparc-complete (sonnet) from `references/agents/sparc/completion-agent.md`

**Spawning:**
1. Load agent definition
2. Build prompt:
   - Full agent definition
   - "YOUR ASSIGNMENT: Verify feature is truly complete."
   - "CONTEXT: Read ALL files in Projects/{project}/SPARC/{feature}/"
   - "Verify: 1) All acceptance criteria pass with evidence, 2) Test coverage >= {threshold}%, 3) Docs updated, 4) Security review clean"
   - "OUTPUT: Write completion report to Projects/{project}/SPARC/{feature}/Completion.md"
   - "RETURN: PASS or FAIL with details."
3. Spawn: `Agent(prompt=above, model="sonnet", description="SPARC Complete — {feature}")`

**Gate:** PASS with full sign-off?
- YES: Feature complete. Fire sparc-phase-complete event.
- NO: Loop back to Phase R with unmet criteria.

---

## Phase Transitions and Obsidian Communication

```
Phase S writes → Specification.md
  ↓ (Phase P reads Specification.md)
Phase P writes → Pseudocode.md
  ↓ (Phase A reads Specification.md + Pseudocode.md)
Phase A writes → Architecture.md + ADRs
  ↓ (CTO reads Architecture.md, writes approval)
  ↓ (Phase R reads Architecture.md + Specification.md)
Phase R writes → Refinement-Log.md + actual code
  ↓ (Phase C reads ALL SPARC/{feature}/ files)
Phase C writes → Completion.md
```

Each agent reads the PREVIOUS phase's Obsidian output. This is how agents communicate without direct interaction.

---

## Parallel Tracks
S+P can overlap (start P when S stabilizes). P+A can overlap. R->C is strict. Within R: implementation + test writing run in parallel (two agents).

## SPARC in Agile Context
Design Gate = S+P+A. Code Gate = R. QA/Acceptance Gate = C.

## Integration Points
- Called from: /autonexus:sparc
- Each phase fires: sparc-phase-complete event (hookable)
- Chainable: sparc -> agile -> ship
- Agent spawning follows: agent-spawning-protocol.md
