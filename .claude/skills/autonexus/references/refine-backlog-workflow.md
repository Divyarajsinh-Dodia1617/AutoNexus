# Refine Backlog Workflow — /autonexus:refine-backlog

Interactive backlog grooming with agent assistance. Users create, refine, reprioritize, split, or archive stories. All stories are stored in Obsidian. Every story must be testable before entering a sprint.

**Core idea:** User provides intent → Agent(s) structure it into proper stories → Validate testability → Persist to Obsidian.

## Trigger

- User invokes `/autonexus:refine-backlog`
- User says "refine backlog", "groom stories", "add stories", "update backlog", "create user stories"

## PREREQUISITE: Interactive Setup

If no arguments provided, use `AskUserQuestion`:

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Action` | "What would you like to do?" | "Add new stories", "Refine existing stories", "Reprioritize backlog", "Split a story", "Archive stories", "Review full backlog" |
| 2 | `Scope` | "Which epic or area?" | List of existing epics from Obsidian + "New epic" + "All" |

## Architecture

```
/autonexus:refine-backlog
  ├── Phase 1: Load — Read current backlog from Obsidian
  ├── Phase 2: Intent — Determine user action
  ├── Phase 3: Execute — Spawn agents for the action
  ├── Phase 4: Validate — Ensure all stories are testable
  └── Phase 5: Persist — Write to Obsidian
```

## Phase 1: Load Current Backlog

1. Check Obsidian connectivity (standard two-step check)
2. Read `Projects/{ProjectName}/Backlog/_index.md`
   - IF exists: Parse story count, epic count, ready/not-ready counts
   - IF not exists: Create initial backlog structure
3. Present to user: "{N} stories across {M} epics. {K} ready for sprint, {J} need refinement."

## Phase 2: Determine User Action

From arguments or `AskUserQuestion` response:

| Action | Agent(s) Needed |
|--------|----------------|
| Add new stories | PM + Architect (+ UX if UI story) |
| Refine existing | PM + Architect |
| Reprioritize | PM |
| Split story | PM + Architect |
| Archive | PM |
| Review full backlog | PM |

## Phase 3: Execute

### Action: Add New Stories

User provides rough description(s) of what they want.

Spawn PM Agent (model: sonnet):
- Task: Convert user's description into properly formatted stories. For each story:
  - Write user story format ("As a..., I want..., so that...")
  - Write ≥3 testable acceptance criteria (each maps to one test assertion)
  - Define verify_command (how to test mechanically)
  - Define test_file (where tests will live)
  - Set status: "draft"
  - Assign to appropriate epic (existing or suggest new)
- Write each story to `Backlog/stories/STORY-{next-id}.md`
- Update `Backlog/_index.md` with new entries

Spawn Architect Agent (model: sonnet):
- Task: Read the new stories from Obsidian. For each:
  - Technical feasibility assessment
  - Identify dependencies on existing stories
  - Estimate complexity (T-shirt: XS, S, M, L, XL)
  - Flag risks
  - Note architecture considerations
- Append "Technical Notes" section to each story note

IF story involves UI:
  Spawn UX Designer Agent (model: sonnet):
  - Task: Read story. Propose user flow. Identify accessibility requirements. Append "UX Notes" to story note.

Present refined stories to user for approval.
User approves → status changes to "ready"
User requests changes → re-spawn PM with feedback

### Action: Refine Existing Stories

User picks story ID(s) or scope (epic, all not-ready).

Spawn PM Agent:
- Read specified stories from Obsidian
- For each:
  - IF acceptance criteria are vague → rewrite to be specific, measurable, testable
  - IF verify_command is missing → define it
  - IF test_file is missing → define it
  - IF acceptance criteria < 3 → add more
- Update story notes in Obsidian

Spawn Architect Agent:
- Read refined stories
- Update technical notes if scope changed
- Re-assess complexity

### Action: Reprioritize

User provides priority guidance (e.g., "auth stories before notifications", "STORY-45 is now critical").

Spawn PM Agent:
- Read all stories + user's guidance
- Reorder `Backlog/_index.md`
- Respect dependency constraints (if A depends on B, B must come before A)
- Write rationale for priority changes

### Action: Split Story

User picks a story to split.

Spawn PM Agent + Architect Agent (sequentially):
- PM reads story, splits into 2-3 smaller stories:
  - Each independently deliverable
  - Each has its own acceptance criteria + test contract
  - Each ≤8 story points
  - Original story archived with links to children
- Architect reviews splits for technical coherence

### Action: Archive Stories

Spawn PM Agent:
- Move selected stories to `Backlog/Archive/`
- Update `_index.md` to remove them
- Add archive reason (completed, cancelled, deferred, merged)

### Action: Review Full Backlog

Spawn PM Agent:
- Read all stories
- Produce summary:

| Status | Count |
|--------|-------|
| Ready | {N} |
| Draft | {M} |
| Needs refinement | {K} |
| Blocked | {J} |

- List stories needing attention
- Present to user

## Phase 4: Validate Testability

For every new or refined story, check:

| Check | Required |
|-------|----------|
| Has ≥3 acceptance criteria | ✓ |
| Each AC is testable (maps to assertion) | ✓ |
| Has verify_command defined | ✓ |
| Has test_file path defined | ✓ |
| Has technical notes from Architect | ✓ |
| No unresolved dependencies | ✓ |
| Status is "ready" | ✓ |

IF any check fails:
- Spawn appropriate agent to fix
- PM for acceptance criteria
- Architect for technical notes
- Loop until all checks pass

Stories that pass all checks → status: "ready"
Stories that fail → status: "needs-refinement" (won't be picked for sprint)

## Phase 5: Persist

All changes written to Obsidian:
- Story notes created/updated
- `_index.md` updated with current counts and priority order
- Epic notes updated if stories added/removed

## Story Note Template

```markdown
---
type: story
id: STORY-{NNN}
epic: "[[epic-{slug}]]"
status: draft|needs-refinement|ready|planned|designing|implementing|in-review|testing|done|blocked|archived
sprint: null
priority: {N}
story_points: null
assigned_to: []
created: {YYYY-MM-DD}
refined: {YYYY-MM-DD}
completed: null
---

# STORY-{NNN}: {Title}

## User Story
As a {role}, I want {feature} so that {benefit}.

## Acceptance Criteria
- [ ] AC-1: {specific, testable criterion}
- [ ] AC-2: {specific, testable criterion}
- [ ] AC-3: {specific, testable criterion}

## Test Contract
verify_command: "{command that runs story tests}"
test_file: "{path/to/test/file}"
coverage_target: 90

## Technical Notes (Architect)
{Added during refinement or sprint planning}

## UX Notes
{Added if story has UI component}

## Estimates
| Agent | Points | Rationale |
|-------|--------|-----------|
| | | |
| **Consensus** | | |

## Design
{Populated during sprint}

## Implementation Log
{Populated during sprint}

## Review Log
{Populated during sprint}

## QA Log
{Populated during sprint}

## Blockers
{Populated during sprint}
```

## Epic Note Template

```markdown
---
type: epic
id: epic-{slug}
status: active|completed|cancelled
created: {YYYY-MM-DD}
stories_total: {N}
stories_completed: {M}
---

# Epic: {Title}

## Objective
{What this epic achieves}

## Success Metrics
- {Measurable outcome 1}
- {Measurable outcome 2}

## Stories
| Story | Status | Points |
|-------|--------|--------|
| [[STORY-{id}|{title}]] | {status} | {pts} |
```

## Backlog Index Template

```markdown
---
type: backlog
project: {ProjectName}
last_refined: {YYYY-MM-DD}
total_stories: {N}
ready_stories: {M}
---

# Product Backlog

## Priority Order
1. [[STORY-{id}|{title}]] — {pts}pts — {status}
2. [[STORY-{id}|{title}]] — {pts}pts — {status}

## Epics
| Epic | Stories | Completed | Remaining |
|------|---------|-----------|-----------|
| [[epic-{slug}|{title}]] | {total} | {done} | {remaining} |

## Refinement History
- {YYYY-MM-DD}: {what was refined}
```

## Flags

| Flag | Purpose |
|------|---------|
| `--new "title"` | Quick-add a story with this title |
| `--story STORY-{id}` | Refine a specific story |
| `--epic epic-{slug}` | Scope to an epic |
| `--review` | Review full backlog status |
| `--archive STORY-{id}` | Archive a specific story |

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Stories without test contracts | Can't verify implementation; sprint execution will fail |
| Acceptance criteria that say "should work" | Not testable; PM must rewrite |
| Stories >13 points | Too large to complete in sprint; must split |
| Skipping Architect assessment | Dependencies and risks not identified |
