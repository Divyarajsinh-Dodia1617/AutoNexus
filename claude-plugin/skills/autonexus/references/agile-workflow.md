# Agile Workflow — /autonexus:agile

Full Agile sprint lifecycle with autonomous agent team. The orchestrator (Scrum Master) coordinates specialized agents — each in its own isolated context window — through Obsidian as the shared communication channel. 10 user stories per sprint, full Agile ceremonies, quality gates, and automatic conflict resolution.

**Core idea:** Orchestrator reads Obsidian → Spawns agent (isolated context) → Agent does work + writes to Obsidian → Orchestrator reads summary → Updates sprint board → Next agent.

## Trigger

- User invokes `/autonexus:agile`
- User says "run a sprint", "agile sprint", "start sprint", "run the team"

## PREREQUISITE: Interactive Setup

**CRITICAL — BLOCKING PREREQUISITE:** If project context is not provided, MUST use `AskUserQuestion`.

**Adaptive question selection:**
- No input → ask all questions
- Project + backlog provided → ask team config only
- All provided → skip setup

Batch ALL questions into a SINGLE `AskUserQuestion` call:

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Project` | "What project is this sprint for?" | Auto-detected from git remote/package.json/dirname |
| 2 | `Backlog` | "Where are the stories?" | "Obsidian backlog (existing)", "Create new backlog", "Import from GitHub Issues" |
| 3 | `Team` | "What team configuration?" | "Auto (scale per story complexity)", "Lean (5 agents)", "Standard (12 agents)", "Full (20+ agents)" |
| 4 | `Sprint Goal` | "What's the focus for this sprint?" | Free text — sprint theme/objective |
| 5 | `Launch` | "Ready to start sprint planning?" | "Start planning", "Review backlog first (/autonexus:refine-backlog)", "Cancel" |

## Architecture

```
/autonexus:agile
  ├── Phase 0: Sprint Setup — Obsidian check, load roster, load backlog
  ├── Phase 1: Sprint Planning — PM prioritizes, Architect assesses, Planning Poker, EM assigns
  ├── Phase 2: Sprint Execution — Per-story: Design → Implement → Review → QA → Accept
  ├── Phase 3: Sprint Review — PM acceptance, CEO/CPO business assessment
  ├── Phase 4: Retrospective — Metrics, agent reflections, action items
  └── Phase 5: Next Sprint — Backlog update, loop or end
```

## The Orchestrator Pattern

The orchestrator (main Claude context) is a PURE SCHEDULER. It contains ONLY:
- Sprint state machine (current phase, current story, current step)
- Agent roster index (loaded from `references/agents/index.md`)
- Quality gate rules (loaded from `references/agile-gates.md`)
- Escalation chain (who supervises whom)
- Workflow transitions (what happens next)

It does NOT contain: source code, test output, design documents, review feedback, agent reasoning.

### Agent Spawning Protocol

Every agent activation follows this exact protocol:

```
1. READ from Obsidian:
   - Sprint board (current state: Sprints/sprint-{N}/Board.md)
   - Story note (if applicable: Backlog/stories/STORY-{id}.md)
   - Any relevant artifact refs (design doc path, review note path)
   → Read ONLY metadata/status fields, not full content

2. LOAD agent definition:
   - Read from references/agents/{category}/{role}.md
   - Extract: identity, capabilities, constraints, model, escalation

3. CONSTRUCT agent prompt:
   ┌─────────────────────────────────────────────┐
   │ {Agent definition content}                  │
   │                                             │
   │ YOUR ASSIGNMENT:                            │
   │ {Specific task description}                 │
   │                                             │
   │ CONTEXT (read these from Obsidian):         │
   │ - {List of Obsidian note paths to read}     │
   │                                             │
   │ WHEN DONE, write results to:                │
   │ - {Obsidian note path + section}            │
   │                                             │
   │ RETURN TO ME: 2-3 sentence summary only.    │
   └─────────────────────────────────────────────┘

4. SPAWN via Agent tool:
   Agent(prompt={above}, model={from agent def}, description="{role} {task}")

5. RECEIVE brief return summary (this is ALL that enters main context)

6. UPDATE sprint board in Obsidian (status transition)

7. TRANSITION state machine to next step
```

## Phase 0: Sprint Setup

1. **Obsidian connectivity check** — Two-step check per SKILL.md protocol
2. **Load team roster** — Read `references/agents/index.md` for available agents and team configs
3. **Load/create backlog** — Read `Projects/{ProjectName}/Backlog/_index.md` from Obsidian
   - IF no backlog exists: suggest running `/autonexus:refine-backlog --new` first
4. **Create sprint folder** — Create `Sprints/sprint-{N}/` in Obsidian:
   - Goal.md (sprint goal + committed stories)
   - Board.md (kanban board — single note, updated in place throughout sprint)
   - Standups/ directory
5. **Determine sprint number** — Check existing sprint folders, increment
6. **Archive old sprints** — If >5 completed sprints, archive oldest to Sprints/Archive/

**Output:** `Phase 0: Sprint {N} initialized. {M} stories in backlog. Team: {config}.`

## Phase 1: Sprint Planning Ceremony

See `references/agile-ceremonies.md` for full ceremony protocol.

### Step 1: Product Owner Prioritization

Spawn PM Agent (model: sonnet):
- Task: Read full backlog from Obsidian. Select top 10 stories that have testable acceptance criteria (status: "ready"). Prioritize by business value + dependency order. Write selection + rationale to `Sprints/sprint-{N}/Goal.md`.
- Returns: "Selected 10 stories totaling {X} points. Top priority: STORY-{id} ({reason})."

### Step 2: Technical Assessment

Spawn Architect Agent (model: sonnet):
- Task: Read the 10 selected stories from Obsidian. For each, assess technical risks, cross-story dependencies, architecture concerns. Append technical assessment to each story note. Flag any stories needing design before implementation.
- Returns: "Assessed all 10. {X} have dependencies. {Y} have architecture risks. Recommended order: {list}."

### Step 3: Planning Poker (Estimation)

Spawn 3 agents IN PARALLEL (each estimates ALL 10 stories):

Spawn Sr. Backend Engineer (model: sonnet):
- Task: Estimate story points (1,2,3,5,8,13) for each story based on backend complexity. Write estimates + rationale to each story note under "Estimates" section.

Spawn Sr. Frontend Engineer (model: sonnet):
- Task: Same, based on frontend complexity.

Spawn Sr. QA Engineer (model: sonnet):
- Task: Estimate test effort for each story. Write to story notes.

After all return:
- Orchestrator reads estimates from Obsidian
- IF any story has estimates diverging by >3 points:
  Spawn Staff Engineer (model: sonnet):
  - Task: Read divergent estimates + rationales. Facilitate consensus. Write final agreed estimate.

### Step 4: Capacity Check & Assignment

Spawn EM Agent (model: sonnet):
- Task: Read sprint goal + all estimates from Obsidian. Team capacity based on team config. Assign stories to agents based on domain fit. Write sprint Board.md with assignments and priority order. Flag if overcommitted.
- Returns: "Committed 10 stories, {X} points. Capacity: {Y}. Assignments written."

### Step 5: Strategic Review

Spawn CTO Agent (model: opus):
- Task: Read sprint plan from Obsidian. Review for strategic alignment. Approve or flag concerns. Write review to Goal.md.
- Returns: "Approved." or "Concerns: {list}."

### Step 6: User Checkpoint

Present sprint summary to user:
"Sprint {N} planned: 10 stories, {X} points. Sprint goal: {goal}. Proceed?"

IF user approves → Phase 2
IF user adjusts → spawn relevant agents to handle changes
IF user says "review backlog" → suggest `/autonexus:refine-backlog`

**Output:** `Phase 1: Sprint Planning complete. 10 stories committed. {X} total points.`

## Phase 2: Sprint Execution

Process stories in priority order from Board.md.

FOR each story:

### Step A: Design

Spawn Architect Agent (model: sonnet):
- Task: Read story note from Obsidian. Read Knowledge Center (Architecture.md, Components.md). Design: API contracts, data model, component plan, integration points. Write `Designs/design-STORY-{id}.md` to Obsidian. Update story note "Design" section with summary + link.
- Returns: "Design complete. {brief description}."

Spawn CTO Agent (model: opus):
- Task: Read design doc from Obsidian. Review for architecture compliance, ADR consistency, scalability. Approve or reject with feedback. Write approval/rejection to design doc.
- Returns: "Approved." or "Rejected: {reason}."

**✅ DESIGN GATE** — see `references/agile-gates.md`

IF rejected: Re-spawn Architect with CTO feedback. Max 2 design cycles.
Update Board.md: story status → "implementing"

### Step B: Implement + Test Case Writing (PARALLEL)

Read story's design doc path from Obsidian. Determine implementation tracks from team config.

**Two parallel tracks run simultaneously:**

#### Track 1: Implementation

IF story needs backend + frontend:

  Spawn Sr. Backend Engineer (model: sonnet):
  - Task: Read story + design from Obsidian. Read codebase. Implement backend using a mini autoresearch loop:
    Goal: All backend acceptance tests passing
    Verify: {story's verify_command}
    Iterate until tests pass (max 20 iterations).
    Write implementation log to story note. Commit all changes.
  - Returns: "Implemented. {N} commits. {X}/{Y} tests pass."

  THEN (sequential — frontend may depend on backend API):

  Spawn Sr. Frontend Engineer (model: sonnet):
  - Task: Read story + design + backend API contract from Obsidian. Read codebase. Implement frontend. Write implementation log to story note. Commit all changes.
  - Returns: "Implemented. {N} commits. {X}/{Y} tests pass."

IF story is backend-only:
  Spawn Sr. Backend Engineer only.

IF story is frontend-only:
  Spawn Sr. Frontend Engineer only.

IF story is simple (≤3 points, lean team):
  Spawn Backend Engineer or Frontend Engineer (mid-level) instead.

#### Track 2: Test Case Writing (Jr. QA — runs IN PARALLEL with Track 1)

Spawn Jr. QA Engineer (model: sonnet):
- Task: Read story acceptance criteria + design doc from Obsidian. Write comprehensive manual test cases to story note under "Test Cases" section. Cover ALL acceptance criteria with happy path, edge case, and error path test cases. Use the test case template (TC-{id}-001, TC-{id}-002, etc.). Do NOT wait for implementation to finish — test cases are based on REQUIREMENTS, not code.
- Returns: "{N} test cases written — {X} happy path, {Y} edge cases, {Z} error paths."

**IMPORTANT:** Jr. QA does NOT need implementation to be complete. Test cases are derived from acceptance criteria and design docs. This parallelism saves sprint time.

#### After Both Tracks Complete: Test Case Review

Spawn Sr. QA Engineer (model: sonnet):
- Task: Read Jr. QA's test cases from story note "Test Cases" section. Review for completeness (all ACs covered), specificity (unambiguous steps), edge case coverage, priority accuracy, and Playwright CLI testability. Write review to story note "Test Case Review" section. Approve or request changes.
- Returns: "{Approved | Changes Required}. {X} test cases reviewed. {Y} gaps found."

IF Changes Required:
  Re-spawn Jr. QA Engineer with Sr. QA feedback:
  - Task: Read Sr. QA review feedback from story note "Test Case Review" section. Revise test cases accordingly. Update "Test Cases" section.
  - Returns: "Revised {X} test cases. Added {Y} new test cases."

  Re-spawn Sr. QA Engineer for re-review. Max 2 review cycles.
  IF still not approved after 2 cycles: Escalate to QA Manager for binding decision.

Update Board.md: story status → "in-review"

### Step C: Code Review

Spawn Principal Engineer (model: sonnet) — or Sr. Staff Engineer based on availability:
- Task: Read story note + design doc from Obsidian. Read git diff of implementation (use `git diff` commands). Review for: architecture compliance, code quality, patterns, performance. Write review to `Reviews/review-STORY-{id}.md` in Obsidian.
- Returns: "Approved." or "Changes requested: {N} items. {blocking_count} blocking."

Spawn Security Engineer (model: sonnet):
- Task: Read git diff. OWASP-focused security review. Input validation, auth/authz, data exposure, injection. Write security review to review note.
- Returns: "Passed." or "Failed: {issues}."

**✅ CODE GATE** — see `references/agile-gates.md`

IF blocking feedback:
  Spawn implementation agent again with review feedback context.
  After fix, re-spawn reviewer. Max 3 review cycles.

Update Board.md: story status → "testing"

### Step D: QA (Manual Testing via Playwright CLI)

All testing is manual — no automated test scripts. Jr. QA Engineers execute test cases via Playwright CLI, Sr. QA performs exploratory testing, and QA Manager oversees bug triage.

#### Sub-step D1: QA Manager — Test Plan + Oversight

Spawn QA Manager Agent (model: sonnet):
- Task: Read story + acceptance criteria from Obsidian. Create test plan: happy path (all ACs), edge cases, error paths, integration points, regression scope. Write test plan to story note "QA Log" section. Cross-reference with Jr. QA's existing test cases — identify any gaps the test cases don't cover.
- Returns: "Test plan: {N} test cases covering {M} ACs + {K} edge cases. Jr. QA test case gaps: {list or none}."

#### Sub-step D2: Jr. QA Manual Test Execution via Playwright CLI

Spawn Jr. QA Engineer (model: sonnet) — one per story, or multiple Jr. QAs for 8+ point stories:
- Task: Read approved test cases from story note "Test Cases" section. Execute ALL test cases manually using Playwright CLI browser tools:
  1. `browser_navigate` to the feature URL
  2. `browser_snapshot` to understand page state
  3. Execute each test step using `browser_click`, `browser_fill_form`, `browser_select_option`, `browser_press_key`, etc.
  4. `browser_take_screenshot` after each critical action for evidence
  5. Compare actual vs expected results
  6. Mark each test case PASS / FAIL / BLOCKED
  7. For ANY failure: file bug report with screenshot + console errors (`browser_console_messages`) + network requests (`browser_network_requests`)
  8. Write execution results to story note "QA Execution Log" section
  9. Write bugs to story note "Bugs" section (tagged @sr-qa-eng @qa-manager)
- Returns: "{X}/{Y} tests pass. {Z} bugs found (P0:{a}, P1:{b}, P2:{c}, P3:{d})."

#### Sub-step D3: Sr. QA Exploratory Testing via Playwright CLI

After Jr. QA completes planned test execution:

Spawn Sr. QA Engineer (model: sonnet):
- Task: Read Jr. QA execution results from story note. Perform advanced exploratory testing via Playwright CLI for scenarios the planned test cases didn't cover — complex user flows, state transitions, boundary conditions, error recovery, concurrent access, responsive behavior. Take screenshots for all findings. Write results to story note "QA Log" under "Exploratory Testing" section. File any bugs found to "Bugs" section.
- Returns: "{N} scenarios explored. {X} bugs found. {Y} observations noted."

#### Sub-step D4: Bug Triage (after testing complete)

IF Jr. QA or Sr. QA found bugs:

  Spawn Sr. QA Engineer (model: sonnet):
  - Task: Read all bug reports from story note "Bugs" section (from both Jr. QA and own exploratory testing). Validate reproduction steps, confirm severity, check for duplicates, add technical context. Write validated bugs to "Validated Bugs" section. Route to implementation team.
  - Returns: "{X} bugs validated, {Y} severity adjusted, {Z} routed to dev."

  Spawn QA Manager Agent (model: sonnet):
  - Task: Read all bug reports and Sr. QA's validated bugs. Assess sprint impact. Escalate P0 bugs to EM. Decide if story can proceed to acceptance or must go back to implementation. Write assessment to "QA Manager Assessment" section.
  - Returns: "Sprint impact: {Blocked | Proceed | Clear}. {X} escalations."

#### Sub-step D5: Bug Fix + Re-test Cycle

IF P0 or P1 bugs exist:
  Spawn implementation agent with validated bug details from "Validated Bugs" section.
  After fix:
    Spawn Jr. QA Engineer (model: sonnet):
    - Task: Re-test fixed bugs using Playwright CLI. Execute original reproduction steps. Verify fix. Run related test cases for regression. Update bug status: "Verified Fixed" or "Reopened". Write to "QA Execution Log".
    - Returns: "{X} bugs verified fixed, {Y} bugs reopened."

  IF bugs reopened: loop back to implementation. Max 3 QA cycles.

**✅ QA GATE** — see `references/agile-gates.md`

Pass criteria:
- All manual test cases executed via Playwright CLI (Jr. QA)
- Sr. QA exploratory testing completed
- 0 P0 or P1 bugs outstanding
- P2/P3 bugs documented but do not block
- QA Manager assessment: "Proceed" or "Clear"

Update Board.md: story status → "accepting"

### Step E: Accept

Spawn PM Agent (model: sonnet):
- Task: Read story note from Obsidian (full lifecycle). Verify all acceptance criteria are checked off. Verify QA log shows all tests passing. Verify review log shows all approvals. Mark story as Done. Update story frontmatter: status → "done", completed → {date}.
- Returns: "STORY-{id} accepted. All {N} ACs verified."

Update Board.md: story → Done column. Update points completed.

### Standup Check (between stories)

Every 2-3 stories, run a standup:

Spawn Scrum Master Agent (model: sonnet):
- Task: Read Board.md. Read all in-progress story notes. Compile standup: per-story status, burn metrics (points completed vs remaining, sprint day vs total), blockers. Write to `Standups/{date}.md`. If off-track, recommend action (scope cut, reprioritize).
- Returns: "{X}/{Y} stories done ({P}/{Q} pts). {status}. Blockers: {list or none}."

IF off-track:
  Present to user: "Sprint at risk. Scrum Master recommends: {action}. Proceed?"

IF blocked:
  Spawn EM Agent to resolve blockers.
  If can't resolve → spawn VP Engineering for escalation.

### Conflict Resolution (when detected)

IF two agents produce contradictory outputs (detected by orchestrator reading Obsidian):

1. Identify conflict type
2. Look up escalation chain:

| Conflicting Agents | Supervisor |
|---|---|
| Jr. × Jr. | Sr. Engineer |
| Sr. × Sr. | Staff Engineer |
| Staff × Staff | Principal Engineer |
| Engineer × QA | EM |
| Architect × Sr. Eng | Principal Engineer |
| PM × EM | VP Engineering + VP Product |
| Architect × CTO | CEO |
| Security × Anyone | Security Architect |
| CTO × CPO | CEO |

3. Spawn Supervisor Agent:
   - prompt: "Two agents disagree: Agent A ({role}): {position}. Agent B ({role}): {position}. Read full context from Obsidian: {paths}. Make a binding decision. Write as ADR to Decisions/{date}-{slug}.md. Update story note with decided approach."

4. Read decision. Proceed with decided approach.
   The ADR is BINDING — downstream agents must follow it.

## Phase 3: Sprint Review

See `references/agile-ceremonies.md` for full ceremony protocol.

Spawn PM Agent (model: sonnet):
- Task: Read all Done stories from Board.md. Verify all acceptance criteria met. Compile sprint review: delivered features, unfinished stories, velocity. Write `Sprints/sprint-{N}/Review.md`.
- Returns: "Sprint delivered: {X}/{Y} stories, {P}/{Q} points."

Spawn CEO Agent (model: opus):
- Task: Read Review.md. Assess business value delivered, alignment with goals, ROI. Write business assessment to Review.md.
- Returns: "Business assessment: {summary}."

Present to user: "Sprint {N} complete. {X} stories delivered. Review written. Feedback?"

**Output:** `Phase 3: Sprint Review complete. {X}/{Y} stories, {P}/{Q} points delivered.`

## Phase 4: Retrospective

See `references/agile-ceremonies.md` for full ceremony protocol.

### Step 1: Metrics

Spawn Scrum Master Agent (model: sonnet):
- Task: Read ALL sprint artifacts from Obsidian (Board.md, standups, story notes, Review.md). Calculate: velocity (points committed vs completed), story completion rate, review cycles per story, bug escape rate, blocker frequency. Write metrics to `Sprints/sprint-{N}/Retro.md`.
- Returns: "Metrics compiled. Velocity: {X}/{Y}. Completion: {Z}%."

### Step 2: Agent Reflections (parallel)

For each agent role that was active, spawn in parallel:

Spawn {Agent Role} (model: sonnet):
- Task: "You participated in sprint {N} as {role}. Read sprint artifacts from Obsidian relevant to your work. Reflect: What went well? What could improve? What blocked/slowed you? What would you do differently? Write reflection to Retro.md under '{Role} Reflection'."
- Returns: "{2-3 sentence reflection summary}."

### Step 3: Action Items

Spawn EM Agent (model: sonnet):
- Task: Read complete Retro.md (metrics + all reflections). Identify top 3 things to keep, top 3 to improve, specific action items for next sprint. Write to Retro.md under "Action Items". Update `Team/Norms.md` if process changes adopted.
- Returns: "Action items: {summary}."

Present retro summary to user.

**Output:** `Phase 4: Retrospective complete. {X} action items for next sprint.`

## Phase 5: Next Sprint

1. Move incomplete stories back to backlog (status → "backlog", sprint → null)
2. Spawn PM Agent: Reprioritize backlog based on sprint learnings. Update `Backlog/_index.md`.
3. Ask user: "Start next sprint?" / "Refine backlog first?" / "End"

IF start → Loop to Phase 1
IF refine → Suggest `/autonexus:refine-backlog`
IF end → Session end protocol

## Obsidian Vault Structure

```
Projects/{ProjectName}/
├── Knowledge/              ← EXISTING
├── Decisions/              ← EXISTING (+ new ADRs from conflict resolution)
├── Iterations/             ← EXISTING (+ engineer agents write here)
├── Predictions/            ← EXISTING
│
├── Team/                   ← NEW
│   ├── Roster.md           ← Active team config for this project
│   └── Norms.md            ← Working agreements, updated by retros
│
├── Backlog/                ← NEW
│   ├── _index.md           ← Priority-ordered story list
│   ├── epics/
│   │   └── epic-{slug}.md
│   └── stories/
│       └── STORY-{id}.md   ← Lifecycle document (all agents append to it)
│
├── Sprints/                ← NEW
│   └── sprint-{N}/
│       ├── Goal.md          ← Sprint goal + committed stories
│       ├── Board.md         ← Kanban board (updated in place)
│       ├── Standups/
│       │   └── {YYYY-MM-DD}.md
│       ├── Review.md        ← Sprint review
│       └── Retro.md         ← Retrospective
│
├── Designs/                ← NEW
│   └── design-STORY-{id}.md
│
├── Reviews/                ← NEW
│   └── review-STORY-{id}.md
│
└── Dashboard.md            ← EXISTING, enhanced with velocity chart
```

## Resumability

All state lives in Obsidian. The sprint is fully resumable:

```
/autonexus:agile --resume

Orchestrator:
  1. Read latest Board.md from Obsidian
  2. Find current story + its status from frontmatter
  3. Determine phase:
     - Has design doc? → past design
     - Has implementation log? → past implementation
     - Has review? → past review
     - Has QA log? → past QA
     - Has acceptance? → done, move to next story
  4. Resume from next incomplete phase
```

## Estimated Agent Spawns Per Sprint

| Phase | Spawns |
|-------|--------|
| Sprint Planning | ~6 (PM, Architect, 3×estimators, EM) + 1 (CTO) |
| Per Story (×10) | ~13 each (Architect, CTO, impl, Jr. QA test case writing (parallel w/ impl), Sr. QA test case review, reviewer, security, QA mgr, Jr. QA Playwright execution, Sr. QA exploratory Playwright testing, Sr. QA bug triage, QA mgr bug oversight, PM) |
| Standups (~4) | ~4 (Scrum Master) |
| Sprint Review | ~2 (PM, CEO) |
| Retrospective | ~8 (Scrum Master, 5-6 reflections, EM) |
| **Total** | **~152 spawns** (clean), ~175 with rework/re-tests |

Each spawn is focused: isolated context, clear task, brief return. The orchestrator's context stays clean throughout. Jr. QA test case writing runs in parallel with implementation, so it adds zero wall-clock time to that phase.

## Integration with Existing Commands

Engineer subagents internally run the equivalent of `/autonexus` (core loop):
- Goal: acceptance criteria passing
- Metric: test pass count
- Verify: story's verify_command
- Bounded: max 20 iterations

The orchestrator can optionally invoke:
- `/autonexus:predict` during code review as "Architecture Review Board"
- `/autonexus:security` during code review as "Security Audit"
- `/autonexus:debug` when QA finds bugs with unclear root cause
- `/autonexus:fix` after debug identifies root cause
- `/autonexus:learn` during retrospective for documentation updates
- `/autonexus:ship` after sprint review for deployment

Each runs as its own subagent spawn → separate context → results to Obsidian.

## Safety

### Story Validation
Every story MUST have before entering a sprint:
- ≥3 testable acceptance criteria
- verify_command defined
- test_file path defined
- Architect technical assessment

### Sprint Bounds
- Max 10 stories per sprint
- Max 3 review cycles per story (then escalate)
- Max 3 QA cycles per story (then escalate)
- Max 2 design cycles per story (then escalate)
- Max 20 implementation iterations per engineer agent

### Human Checkpoints
- Sprint planning approval (user approves scope)
- Sprint at-risk warning (user decides scope cut)
- Sprint review feedback (user approves deliverables)
- Retro acknowledgment (user reviews process changes)

## Flags

| Flag | Purpose |
|------|---------|
| `--resume` | Resume interrupted sprint from Obsidian state |
| `--team <config>` | Team size: auto/lean/standard/full |
| `--stories "S1,S2,..."` | Specific stories for this sprint |
| `--sprint N` | Override sprint number |
| `--phase <name>` | Run specific phase only (planning/review/retro) |
| `--parallel` | Enable parallel story execution (multiple worktrees) |
| `--dry-run` | Plan sprint without executing |

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Orchestrator reads source code | Pollutes main context, defeats isolation principle |
| Skip design gate | Architecture drift, rework in review |
| Skip QA phase | Bug escape, tech debt accumulation |
| >10 stories per sprint | Overcommitment, reduced quality |
| Ignore standup warnings | Sprint failure not caught early |
| Skip retro | Same mistakes repeated |
| Manual conflict resolution by orchestrator | Violates isolation; supervisor agents decide |
