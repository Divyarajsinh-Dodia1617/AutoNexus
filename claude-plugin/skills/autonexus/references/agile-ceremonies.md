# Agile Ceremonies — /autonexus:agile

Detailed protocols for all Agile ceremonies in the AutoNexus agent team workflow. Each ceremony follows a structured agent-spawning protocol where every participant runs in its own isolated context window. All communication happens through Obsidian notes.

## Ceremony 1: Sprint Planning

### Duration: ~7 agent spawns

### Participants
- PM Agent (prioritization)
- Architect Agent (technical assessment)
- Sr. Backend Engineer (estimation)
- Sr. Frontend Engineer (estimation)
- Sr. QA Engineer (estimation)
- Staff Engineer (consensus facilitator, if needed)
- EM Agent (assignment)
- CTO Agent (strategic review)

### Protocol

#### Step 1: Backlog Prioritization

Spawn PM Agent:
- Read `Backlog/_index.md` and all story files from Obsidian
- Select top 10 stories with status "ready" (have testable ACs + test contract)
- Rank by business value, accounting for dependency order
- Write to `Sprints/sprint-{N}/Goal.md`:
  - Sprint goal (theme)
  - Ordered story list with rationale for each selection
  - Dependencies graph (which stories block which)
  - Stories deferred and why

#### Step 2: Technical Assessment

Spawn Architect Agent:
- Read the 10 selected stories from Obsidian
- For each story:
  - Technical feasibility (1-5 scale)
  - Risks (list)
  - Dependencies on other stories or external systems
  - Architecture concerns
  - Needs design phase? (yes/no)
- Append to each story note under "Technical Notes"
- If any story is infeasible → flag it, suggest alternative

#### Step 3: Planning Poker

Spawn 3 estimation agents IN PARALLEL:

**Sr. Backend Engineer:**
- Estimate story points (1,2,3,5,8,13) for backend complexity
- Write estimate + 1-sentence rationale to story note "Estimates" table

**Sr. Frontend Engineer:**
- Same for frontend complexity

**Sr. QA Engineer:**
- Estimate test effort
- Write to story note "Estimates" table

After all return, orchestrator reads estimates from Obsidian.

**Divergence handling:**
- IF all estimates within 2 points → use median
- IF spread > 3 points → spawn Staff Engineer as facilitator:
  - Reads all estimates + rationales
  - Identifies cause of disagreement
  - Writes consensus estimate with combined rationale

#### Step 4: Capacity Check & Assignment

Spawn EM Agent:
- Read Goal.md + all story estimates
- Team capacity based on team config:

| Config | Points/Sprint |
|--------|--------------|
| Lean (5 agents) | ~25 pts |
| Standard (12 agents) | ~50 pts |
| Full (20+ agents) | ~80 pts |

- Assign stories to agents by domain fit
- Create Board.md with kanban columns: To Do | In Design | In Progress | In Review | In QA | Done
- Flag overcommitment if total points > capacity

#### Step 5: Strategic Review

Spawn CTO Agent (model: opus):
- Read Goal.md + Board.md
- Review: strategic alignment, architecture decisions, tech debt balance
- Approve or flag concerns
- Write review to Goal.md

#### Step 6: User Checkpoint

Orchestrator presents summary. User approves/adjusts/cancels.

### Outputs
- `Sprints/sprint-{N}/Goal.md` — Sprint goal + story selections
- `Sprints/sprint-{N}/Board.md` — Kanban board with assignments
- Updated story notes with estimates + technical assessments

---

## Ceremony 2: Daily Standup

### Duration: 1 agent spawn

### Participants
- Scrum Master Agent

### Trigger

Run between every 2-3 stories during sprint execution, or when orchestrator detects a status change.

### Protocol

Spawn Scrum Master Agent:
- Read Board.md (current kanban state)
- Read all story notes for in-progress stories (last log entry)
- Compile standup report:

| Story | Status | Last Update | Next Step | Blocker? |
|-------|--------|-------------|-----------|----------|

Burn metrics:
- Points completed: {X}/{Y} ({Z}%)
- Stories completed: {A}/{B}
- Sprint day: {D}/{total}
- Projected completion: on-track / at-risk / behind

IF at-risk (burn rate suggests won't complete all stories):
- Calculate shortfall
- Recommend: which stories to defer
- Flag: "SPRINT AT RISK"

IF blocked:
- Identify blocker type (technical, dependency, external)
- Propose resolution
- Recommend escalation path if needed

### Output
- `Sprints/sprint-{N}/Standups/{YYYY-MM-DD}.md`

### Escalation

If standup flags "at risk" → orchestrator presents to user with options:
1. Continue (optimistic)
2. Cut scope (remove lowest-priority stories)
3. Extend sprint
4. Add agents (upgrade team config)

---

## Ceremony 3: Sprint Review

### Duration: ~3 agent spawns

### Participants
- PM Agent (acceptance verification)
- CEO Agent (business assessment)
- CPO Agent (product assessment, optional for Full team)

### Protocol

#### Step 1: Acceptance Verification

Spawn PM Agent:
- Read Board.md → list all stories in Done column
- For each Done story:
  - Verify ALL acceptance criteria checked off in story note
  - Verify QA log shows all tests passing
  - Verify review log shows required approvals
  - Mark acceptance status
- For incomplete stories:
  - Document what was completed vs remaining
  - Reason for incompleteness
- Write `Review.md`:
  - Delivered features list with story links
  - Unfinished stories + reasons
  - Velocity: committed vs delivered (points + stories)
  - Acceptance test summary

#### Step 2: Business Assessment

Spawn CEO Agent (model: opus):
- Read Review.md
- Assess:
  - Business value delivered
  - Alignment with quarterly OKRs
  - ROI of engineering effort
  - Strategic recommendations for next sprint
  - Market/competitive considerations
- Append to Review.md under "Business Assessment"

#### Step 3: User Review

Orchestrator presents review summary.
User provides feedback → appended to Review.md.

### Output
- `Sprints/sprint-{N}/Review.md`

---

## Ceremony 4: Sprint Retrospective

### Duration: ~8 agent spawns

### Participants
- Scrum Master Agent (metrics + facilitation)
- All active agent roles (reflections, in parallel)
- EM Agent (action items)

### Protocol

#### Step 1: Metrics Compilation

Spawn Scrum Master Agent:
- Read ALL sprint artifacts:
  - Board.md (final state)
  - All standup notes
  - All story notes (implementation logs, review logs, QA logs)
  - Review.md
- Calculate:

| Metric | Value |
|--------|-------|
| Velocity (committed/delivered) | {X}/{Y} pts |
| Story completion rate | {Z}% |
| Avg review cycles per story | {N} |
| Bug escape rate | {bugs_in_QA}/{total_bugs} |
| Blocker count + avg resolution time | {N}, {T} |
| Agent rework rate | {rework_spawns}/{total_spawns} |

- Write to `Retro.md` under "Metrics"

#### Step 2: Agent Reflections (PARALLEL)

For each agent role active this sprint, spawn in parallel:

Each agent:
- Read sprint artifacts relevant to their domain
- Answer:
  1. What went well in my domain?
  2. What could be improved?
  3. What blocked or slowed me?
  4. What would I do differently?
- Write reflection to `Retro.md` under "{Role} Reflection"

Typical parallel spawns: Sr. BE, Sr. FE, QA, Architect, Security Eng (5-6 agents)

#### Step 3: Action Items

Spawn EM Agent:
- Read complete Retro.md (metrics + all reflections)
- Identify:
  - Top 3 things to KEEP doing
  - Top 3 things to IMPROVE
  - Specific action items for next sprint
  - Process changes to adopt
- Write to Retro.md under "Action Items"
- IF process changes: update `Team/Norms.md`

#### Step 4: Present to User

Orchestrator presents retro summary. User acknowledges or adds feedback.

### Output
- `Sprints/sprint-{N}/Retro.md`
- Updated `Team/Norms.md` (if process changes)

---

## Ceremony 5: Backlog Refinement

### Trigger
User invokes `/autonexus:refine-backlog`

### See
`references/refine-backlog-workflow.md` for full protocol.

This ceremony runs INDEPENDENTLY of sprints. Can be invoked anytime.

---

## Cross-Ceremony State Flow

```
Backlog Refinement → stories refined, ready for selection
       ↓
Sprint Planning → 10 stories committed, board created
       ↓
Sprint Execution → stories flow through kanban columns
       ↓ (standups every 2-3 stories)
Sprint Review → delivered features assessed
       ↓
Retrospective → lessons learned, process updated
       ↓
Next Sprint → backlog re-prioritized → Sprint Planning
```

## Obsidian Note Templates

### Sprint Goal (Goal.md)

```markdown
---
type: sprint-goal
sprint: {N}
start: {YYYY-MM-DD}
planned_stories: 10
planned_points: {total}
goal: "{sprint theme}"
cto_approved: true|false
---
```

### Sprint Board (Board.md)

```markdown
---
type: sprint-board
sprint: {N}
points_completed: {X}
points_remaining: {Y}
stories_completed: {A}
stories_remaining: {B}
updated_at: {ISO timestamp}
---
```

### Standup Note

```markdown
---
type: standup
sprint: {N}
date: {YYYY-MM-DD}
on_track: true|false
blockers: {count}
---
```

### Review Note

```markdown
---
type: sprint-review
sprint: {N}
velocity_committed: {X}
velocity_delivered: {Y}
completion_rate: {Z}%
---
```

### Retro Note

```markdown
---
type: sprint-retro
sprint: {N}
keep_count: 3
improve_count: 3
action_items: {count}
---
```
