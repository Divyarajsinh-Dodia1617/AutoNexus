# Refine Backlog Workflow — /autonexus:refine-backlog

Executive-driven backlog grooming with full leadership involvement. Before any refinement, the system learns the project (codebase + docs + existing backlog). Then executive agents (CEO, CTO, CPO) analyze the project as a product — assessing business strategy, technical landscape, and user needs. They ask the user clarifying questions via AskUserQuestion. TPM then writes refined epics, features, and user stories based on executive input, with EM providing technical guidance. All stories are stored in Obsidian.

**Core idea:** Learn project → Executives assess & ask questions → TPM writes stories (EM assists technically) → Validate → Persist to Obsidian.

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
  ├── Phase 1: Learn Project — Scout codebase, read docs, load existing backlog
  ├── Phase 2: Executive Assessment — CEO, CTO, CPO analyze project-as-product, ask user questions
  ├── Phase 3: Architecture Assessment — Solutions Architect + Security Architect translate executive direction into technical blueprints
  ├── Phase 4: Intent — Determine user action
  ├── Phase 5: Execute — TPM writes stories with EM technical guidance, informed by executive + architect input
  ├── Phase 6: Executive Review — CEO, CTO, CPO review refined stories for alignment
  ├── Phase 7: Validate — Ensure all stories are testable
  └── Phase 8: Persist — Write to Obsidian
```

## Phase 1: Learn Project

**Purpose:** Build a complete picture of the project before any refinement begins. Executives need context to give informed input.

### Sub-step 1A: Codebase Scout

Run a lightweight version of the `/autonexus:learn` scout phase:

1. Scan codebase structure — directories, file counts, LOC per directory
2. Detect project type: library, web app, CLI, API server, monorepo
3. Detect tech stack: languages, frameworks, build tools, test runners
4. **Exclusion list:** `.claude`, `.opencode`, `.git`, `node_modules`, `__pycache__`, `vendor`, `dist`, `build`, `.next`, `coverage`

### Sub-step 1B: Read Existing Documentation

Read project docs for executive context:

1. Check for `docs/` directory: `ls docs/*.md 2>/dev/null`
2. IF docs exist: Read key docs in parallel via Explore agents:
   - `docs/project-overview-pdr.md` or `README.md` — what the project IS
   - `docs/system-architecture.md` — how it's built
   - `docs/codebase-summary.md` — current state
   - `docs/project-roadmap.md` — where it's headed (if exists)
3. IF no docs: Use scout results as sole context

### Sub-step 1C: Read Obsidian Knowledge Center (if available)

```
TRY:
  arch = obsidian_read_note("Projects/{ProjectName}/Knowledge/Architecture.md")
  comp = obsidian_read_note("Projects/{ProjectName}/Knowledge/Components.md")
  deps = obsidian_read_note("Projects/{ProjectName}/Knowledge/Dependencies.md")
CATCH:
  Log: "Knowledge Center not available — using scout + docs only"
```

### Sub-step 1D: Load Existing Backlog

1. Check Obsidian connectivity (standard two-step check)
2. Read `Projects/{ProjectName}/Backlog/_index.md`
   - IF exists: Parse all story notes, epic notes — build full picture of what's already planned/refined/completed
   - IF not exists: Create initial backlog structure
3. Read all existing story notes to understand:
   - What features are already planned
   - What dependencies exist between stories
   - What epics are in progress vs completed
   - What the current priority order is

### Sub-step 1E: Compile Project Context Document

Merge all gathered information into a structured context document (kept in orchestrator memory, NOT written to Obsidian):

```markdown
## Project Context for Executive Review

### Project Identity
- Name: {project}
- Type: {library/web app/API/CLI/monorepo}
- Tech Stack: {languages, frameworks}
- Scale: {N} files, {M} LOC

### Current State (from docs/Knowledge Center)
- Architecture summary: {from Architecture.md or system-architecture.md}
- Key components: {from Components.md or codebase-summary.md}
- Key dependencies: {from Dependencies.md}

### Current Backlog State
- Total stories: {N} across {M} epics
- Ready for sprint: {K}
- In progress: {J}
- Completed: {L}
- Epics: {list with status}

### Existing Features (completed stories)
{List of done stories — executives need to know what's already built}

### Planned Features (ready/draft stories)
{List of planned stories — executives need to know what's already in the pipeline}

### Dependency Map
{Which stories depend on which — critical for executives to assess new feature impact}
```

Output: `Phase 1: Project learned — {type} project, {N} files, {M} existing stories across {K} epics.`

## Phase 2: Executive Assessment

**Purpose:** CEO, CTO, and CPO analyze the project holistically as a PRODUCT — not just code. They provide strategic direction, ask clarifying questions, and identify concerns BEFORE any stories are written or refined.

### Sub-step 2A: CTO — Technical Landscape Assessment

Spawn CTO Agent (model: opus):
- Task: Read the Project Context Document. Analyze the technical landscape of the project. You are NOT reviewing a design — you are assessing the ENTIRE PROJECT as a technology product.

  Assess:
  1. **Technical maturity** — Is the architecture solid? Are there tech debt concerns?
  2. **Scalability readiness** — Can the current architecture handle growth?
  3. **Technology gaps** — Are there missing capabilities the project needs?
  4. **Dependency risks** — Are there risky or outdated dependencies?
  5. **Integration points** — What external systems does this interact with?

  Then, based on the user's refinement action and scope, assess:
  6. **Feature feasibility** — Can the proposed features be built on the current architecture?
  7. **Technical dependencies** — Will the proposed features depend on existing features? Which ones?
  8. **Technical risks** — What could go wrong technically?
  9. **Architecture impact** — Does this require architectural changes?

  Write your technical assessment to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## CTO Technical Assessment"

  **CRITICAL:** Formulate 2-5 clarifying questions for the user about technical aspects. Questions should focus on:
  - Technical constraints the user hasn't mentioned
  - Integration requirements with existing systems
  - Performance/scale expectations
  - Whether proposed features conflict with or depend on existing planned features

  Return: Your assessment summary + your questions for the user.

### Sub-step 2B: CPO — Product & User Assessment

Spawn CPO Agent (model: opus):
- Task: Read the Project Context Document. Analyze the project as a PRODUCT from the user's and market's perspective.

  Assess:
  1. **Product vision coherence** — Do the existing epics tell a coherent product story?
  2. **User value gaps** — What user needs are unmet by the current backlog?
  3. **Feature prioritization** — Are the right things being built first?
  4. **Product-market fit** — Does the planned roadmap serve the target users?
  5. **UX coherence** — Do the planned features create a cohesive user experience?

  Then, based on the user's refinement action and scope, assess:
  6. **User impact** — Who benefits from the proposed features? How much?
  7. **Feature completeness** — Is the proposed scope enough to deliver real user value, or is it too thin?
  8. **Cannibalization risk** — Does this overlap with or undermine existing features?
  9. **Success metrics** — How will we know this feature succeeded?

  Write your product assessment to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## CPO Product Assessment"

  **CRITICAL:** Formulate 2-5 clarifying questions for the user about product aspects. Questions should focus on:
  - Target user segment for the proposed features
  - How this fits the broader product vision
  - Priority relative to existing planned features
  - Success criteria and metrics
  - Whether this feature depends on or enhances existing features

  Return: Your assessment summary + your questions for the user.

### Sub-step 2C: CEO — Business Strategy Assessment

Spawn CEO Agent (model: opus):
- Task: Read the Project Context Document + CTO's technical assessment + CPO's product assessment (from Obsidian). Synthesize a business-level view.

  Assess:
  1. **Business value** — Does the proposed work deliver meaningful business outcomes?
  2. **ROI assessment** — Is the engineering effort justified by the expected return?
  3. **Strategic alignment** — Does this advance the project's strategic goals?
  4. **Opportunity cost** — What are we NOT building by building this?
  5. **Market timing** — Is now the right time for this feature?
  6. **Resource allocation** — Is the team sized right for this scope?

  Read CTO and CPO assessments and:
  7. **Resolve tensions** — If CTO and CPO have conflicting views, provide direction
  8. **Set priorities** — Which executive concerns should weigh heaviest in refinement?

  Write your business assessment to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## CEO Business Assessment"

  **CRITICAL:** Formulate 1-3 clarifying questions for the user about business/strategic aspects. Questions should focus on:
  - Business goals and timeline expectations
  - Budget/resource constraints
  - Go-to-market considerations
  - Whether this is a must-have or nice-to-have

  Return: Your assessment summary + your questions for the user.

### Sub-step 2D: Ask User Questions (CONSOLIDATED)

**CRITICAL:** Collect ALL questions from CEO, CTO, and CPO into a SINGLE `AskUserQuestion` call. Do NOT ask questions one at a time.

```
AskUserQuestion:
  Header: "Executive Team Questions"
  Body: |
    The executive team has reviewed your project and the proposed refinement.
    Before we proceed, they have some questions:

    ### CTO Questions (Technical)
    {CTO's 2-5 questions — numbered}

    ### CPO Questions (Product)
    {CPO's 2-5 questions — numbered}

    ### CEO Questions (Business)
    {CEO's 1-3 questions — numbered}

    Please answer as many as you can. Type "skip" for any you'd rather not answer now.
```

### Sub-step 2E: Executive Alignment (Post-Answers)

After user answers, spawn CEO Agent (model: opus) one more time:
- Task: Read the user's answers + all three executive assessments from Obsidian. Write a unified **Executive Direction** that synthesizes all input into clear guidance for the TPM and EM who will do the actual refinement.

  Write to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## Executive Direction"

  The Executive Direction MUST include:
  1. **Approved scope** — What should be refined/created (based on user's answers + executive assessment)
  2. **Priority guidance** — What matters most, what can be deferred
  3. **Dependency callouts** — Features that depend on existing stories (from CTO + user answers)
  4. **Constraints** — Technical, product, or business constraints to respect
  5. **Success criteria** — How the team will know this refinement was successful
  6. **Open risks** — Concerns that weren't fully resolved by user answers

  Return: Executive Direction summary.

Output: `Phase 2: Executive assessment complete. {N} questions asked, direction set.`

## Phase 3: Architecture Assessment

**Purpose:** Solutions Architect and Security Architect translate the Executive Direction into concrete technical blueprints, security requirements, and architecture constraints BEFORE TPM and EM write stories. This ensures stories are technically grounded and security-aware from the start — not retrofitted later.

### Sub-step 3A: Solutions Architect — Technical Blueprint

Spawn Solutions Architect Agent (model: sonnet):
- Task: Read the Executive Direction + Project Context Document + CTO's technical assessment from Obsidian. Read the Knowledge Center (Architecture.md, Components.md, Dependencies.md).

  Your job: Translate the executive direction into a concrete technical blueprint that TPM and EM can use to write well-informed stories.

  Produce:
  1. **Architecture Impact Analysis** — How do the proposed features affect the current architecture?
     - New components/services needed
     - Existing components that need modification
     - New API endpoints or data models required
     - Integration points with existing systems
  2. **Technical Dependency Map** — Concrete dependency chains:
     - Which existing features/components MUST exist before the new features can be built?
     - Which new features depend on each other? In what order should they be built?
     - External system dependencies (third-party APIs, infrastructure)
  3. **Implementation Strategy** — High-level technical approach for each feature area:
     - Recommended patterns (e.g., event-driven, REST, GraphQL, queue-based)
     - Data flow diagrams (describe, not draw — TPM will reference in stories)
     - Suggested service boundaries
  4. **Technical Constraints for Stories** — Rules TPM must follow when writing stories:
     - "Feature X requires database migration before any API work"
     - "Feature Y must use the existing auth middleware, not a new one"
     - "Feature Z needs a new microservice — that's a separate epic"
  5. **Complexity Estimates** — T-shirt sizing per feature area (XS, S, M, L, XL, XXL)
  6. **ADR Candidates** — Decisions that should be recorded as Architecture Decision Records

  Write to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## Solutions Architect Blueprint"

  Return: Architecture impact summary + dependency chain + {N} technical constraints for TPM.

### Sub-step 3B: Security Architect — Security Requirements

Spawn Security Architect Agent (model: sonnet):
- Task: Read the Executive Direction + Solutions Architect's Blueprint + Project Context Document from Obsidian. Read existing Security ADRs from `Projects/{project}/Decisions/`.

  Your job: Define security requirements and threat considerations for the proposed features BEFORE stories are written. This ensures security is baked into story acceptance criteria, not bolted on during code review.

  Produce:
  1. **Threat Assessment** — STRIDE analysis for each proposed feature area:
     - Spoofing: Authentication requirements
     - Tampering: Data integrity concerns
     - Repudiation: Audit/logging needs
     - Information Disclosure: Data exposure risks
     - Denial of Service: Rate limiting / resource protection
     - Elevation of Privilege: Authorization requirements
  2. **Security Requirements per Feature** — Concrete requirements TPM MUST include in story acceptance criteria:
     - "All endpoints in Feature X must require authenticated user with role Y"
     - "User input in Feature Z must be sanitized against XSS before storage"
     - "PII data in Feature W must be encrypted at rest"
     - "Feature V must implement rate limiting of N requests per minute"
  3. **Compliance Considerations** — Regulatory or policy requirements that affect story scope:
     - Data retention policies
     - Privacy requirements (GDPR, CCPA, etc. if applicable)
     - Access control requirements
  4. **Security Dependencies** — Security infrastructure that must exist before features can be built:
     - "Requires auth middleware upgrade before Feature X"
     - "Needs audit logging service before Feature Y goes live"
  5. **Security ADR Candidates** — Security decisions that need formal documentation

  Write to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## Security Architect Assessment"

  Return: {N} security requirements for stories + {M} threat areas identified + {K} security dependencies.

### Sub-step 3C: Architecture Alignment Check

After both architects complete, the orchestrator reads their outputs from Obsidian and checks for conflicts:

- IF Solutions Architect's blueprint conflicts with Security Architect's requirements (e.g., architect proposes public API, security requires auth):
  - Spawn Solutions Architect again with Security Architect's constraints to reconcile
  - Max 1 reconciliation cycle — if unresolved, flag to CTO

- IF Security Architect identifies dependencies that change the Solutions Architect's build order:
  - Update the dependency map in the blueprint

Write the combined **Architecture Brief** to Obsidian at: `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## Architecture Brief (Combined)"

The Architecture Brief is the consolidated output that TPM and EM will use:
- Technical blueprint (from Solutions Architect)
- Security requirements (from Security Architect)
- Resolved conflicts (if any)
- Final dependency chain (technical + security dependencies merged)
- Technical constraints for story writing
- Complexity estimates

Output: `Phase 3: Architecture assessment complete. {N} technical constraints, {M} security requirements, {K} dependencies mapped.`

## Phase 4: Determine User Action

From arguments or initial `AskUserQuestion` response:

| Action | Agents Needed |
|--------|--------------|
| Add new stories | TPM (writer) + EM (technical advisor) + PM (acceptance criteria) + UX Designer (if UI) |
| Refine existing | TPM (rewriter) + EM (technical advisor) |
| Reprioritize | TPM (with CEO priority guidance + architect dependency chain) |
| Split story | TPM + EM |
| Archive | TPM |
| Review full backlog | TPM + CPO (product coherence check) |

**Note:** Solutions Architect and Security Architect have already done their work in Phase 3. Their output (Architecture Brief) feeds directly into TPM and EM in Phase 5. The Architect agent is no longer spawned separately in Phase 5 — that role is covered by Phase 3.

## Phase 5: Execute

**Key input:** TPM and EM now have THREE sources of guidance:
1. **Executive Direction** (from Phase 2) — business priorities, product vision, strategic constraints
2. **Architecture Brief** (from Phase 3) — technical blueprint, security requirements, dependency chain, complexity estimates, implementation constraints
3. **User's original request** — what they want to build/refine

### Action: Add New Stories

#### Sub-step 5A: TPM Writes Epics, Features, and Stories

Spawn TPM Agent (model: sonnet):
- Task: Read from Obsidian at `Projects/{project}/Backlog/Refinement/executive-input-{date}.md`:
  - Executive Direction (Phase 2)
  - Architecture Brief — Solutions Architect blueprint + Security Architect requirements (Phase 3)
  - Full Project Context (existing backlog, dependencies, completed features)

  Based on executive direction, architecture brief, and user's feature descriptions, create a structured breakdown:

  **For each new feature area:**
  1. **Epic** — Create or assign to existing epic
     - Write epic objective aligned with CEO's business goals and CPO's product vision
     - Define success metrics (from CPO's assessment)
     - Note architecture strategy (from Solutions Architect's blueprint)
  2. **Features within epic** — Break down into logical feature groups following Solutions Architect's recommended service boundaries and implementation strategy
  3. **User Stories per feature** — Write detailed stories:
     - User story format: "As a {role}, I want {feature} so that {benefit}"
     - ≥3 testable acceptance criteria per story
     - **Include Security Architect's requirements as ACs** — e.g., "AC-4: All API endpoints require authenticated user with role X" (from Security Architect's per-feature security requirements)
     - Define verify_command and test_file
     - Status: "draft"
     - **Dependency links** — Use the Architecture Brief's dependency chain (technical + security dependencies merged). Add explicit `depends_on: [STORY-{id}]` in frontmatter with explanation
     - **Priority** — Based on CEO's priority guidance + Architecture Brief's recommended build order
     - **Technical constraints** — Reference Solutions Architect's constraints in story body (e.g., "Must use existing auth middleware per architect constraint #3")

  Write each epic to `Backlog/epics/epic-{slug}.md`
  Write each story to `Backlog/stories/STORY-{next-id}.md`
  Update `Backlog/_index.md` with new entries

  Return: "{N} stories across {M} epics created. {K} dependencies identified. {J} security ACs included."

#### Sub-step 5B: EM Technical Review

Spawn EM Agent (model: sonnet):
- Task: Read the TPM's newly created stories + Architecture Brief from Obsidian. Review from a technical execution perspective:

  For each story:
  1. **Effort realism** — Is this story small enough to complete in a sprint? Cross-check against Solutions Architect's complexity estimates. Suggest splits if >13 points
  2. **Technical clarity** — Are the acceptance criteria technically precise enough for an engineer to implement? Do they align with the Solutions Architect's API contracts and data models?
  3. **Dependency ordering** — Do the dependencies match the Architecture Brief's dependency chain? Are there hidden dependencies the TPM missed?
  4. **Security AC completeness** — Did the TPM include ALL of the Security Architect's requirements for this feature? Flag any missing security ACs
  5. **Architecture constraint compliance** — Does the story respect the Solutions Architect's technical constraints?
  6. **Verify command validity** — Is the verify_command realistic and executable?
  7. **Test file path** — Is the test_file path following project conventions?
  8. **Team assignment hints** — Which agent roles would best implement this? (backend, frontend, fullstack)

  Update each story note with:
  - "## EM Technical Review" section with findings
  - Suggested story point estimate (cross-referenced with Solutions Architect's T-shirt sizing: XS=1, S=2, M=3, L=5, XL=8, XXL=13)
  - Corrected dependencies if any were missed
  - Missing security ACs flagged for TPM to add
  - Recommended team assignment

  Return: "{N} stories reviewed. {X} need splitting. {Y} dependencies corrected. {Z} missing security ACs flagged."

#### Sub-step 5C: PM Acceptance Criteria Hardening

Spawn PM Agent (model: sonnet):
- Task: Read all new stories from Obsidian (with TPM's draft, EM's review, Architecture Brief). Harden the acceptance criteria:
  - Ensure every AC is specific, measurable, and maps to exactly one test assertion
  - Ensure ≥3 ACs per story (not counting security ACs)
  - Ensure security ACs from Security Architect are properly included and testable
  - Ensure verify_command and test_file are correctly defined
  - Cross-check ACs against CPO's success metrics — are we testing what matters to users?
  - Update story notes with hardened ACs

  Return: "{N} stories hardened. All ACs testable. {M} security ACs verified."

IF story involves UI:
  Spawn UX Designer Agent (model: sonnet):
  - Task: Read story + Architecture Brief (for API contracts/data models). Propose user flow. Identify accessibility requirements. Append "## UX Notes" to story note.

### Action: Refine Existing Stories

User picks story ID(s) or scope (epic, all not-ready).

#### Sub-step: TPM Rewrites with Executive + Architecture Context

Spawn TPM Agent (model: sonnet):
- Task: Read the Executive Direction + Architecture Brief + specified stories from Obsidian. For each story:
  - IF acceptance criteria are vague → rewrite to be specific, measurable, testable
  - IF verify_command is missing → define it
  - IF test_file is missing → define it
  - IF acceptance criteria < 3 → add more
  - IF dependency on existing features identified by architects → add `depends_on` links
  - IF Security Architect identified security requirements for this feature → add security ACs
  - Align story priority with CEO's priority guidance + Architecture Brief's build order
  - Ensure story aligns with CPO's product vision
  - Ensure story respects Solutions Architect's technical constraints
- Update story notes in Obsidian

Spawn EM Agent (model: sonnet):
- Task: Read refined stories + Architecture Brief. Review for technical execution clarity, architecture constraint compliance, security AC completeness. Update EM Technical Review section. Correct dependencies. Suggest story point estimates.

### Action: Reprioritize

User provides priority guidance.

Spawn TPM Agent (model: sonnet):
- Task: Read all stories + user's guidance + Executive Direction + Architecture Brief's dependency chain.
  - Reorder `Backlog/_index.md`
  - Respect dependency constraints from Architecture Brief (technical + security dependencies)
  - Respect Solutions Architect's recommended build order (infrastructure before features)
  - Respect Security Architect's security dependencies (auth before features that need auth)
  - Respect CEO's business priorities and CPO's user value prioritization
  - Write rationale for priority changes (referencing which executive/architect input drove each decision)

### Action: Split Story

User picks a story to split.

Spawn TPM Agent + EM Agent (sequentially):
- TPM reads story + Executive Direction + Architecture Brief, splits into 2-3 smaller stories:
  - Each independently deliverable
  - Each has its own acceptance criteria + test contract
  - Each ≤8 story points
  - Original story archived with links to children
  - Maintain dependency links from original story
  - Maintain security ACs in the appropriate child stories
- EM reviews splits for technical coherence, execution clarity, correct dependencies, and security AC preservation

### Action: Archive Stories

Spawn TPM Agent:
- Move selected stories to `Backlog/Archive/`
- Update `_index.md` to remove them
- Add archive reason (completed, cancelled, deferred, merged)
- Check: are any other stories dependent on the archived story? If so, flag to user before archiving.

### Action: Review Full Backlog

Spawn TPM Agent:
- Read all stories
- Produce summary:

| Status | Count |
|--------|-------|
| Ready | {N} |
| Draft | {M} |
| Needs refinement | {K} |
| Blocked | {J} |

- List stories needing attention
- Map all dependencies (which stories block which)

Spawn CPO Agent (model: opus):
- Task: Read full backlog from Obsidian. Assess product coherence:
  - Do the epics tell a coherent product story?
  - Are there user value gaps?
  - Are priorities aligned with product vision?
  - Write product coherence assessment to `Backlog/Refinement/backlog-review-{date}.md`

Present combined review to user.

## Phase 6: Executive Review

**Purpose:** After TPM and EM have written/refined the stories, executives do a final review to ensure alignment with their direction. Architects verify technical and security fidelity.

Spawn CTO Agent (model: opus):
- Task: Read the newly created/refined stories from Obsidian. Quick review for:
  - Technical feasibility confirmed
  - Dependencies correctly mapped (match Architecture Brief)
  - Architecture constraints respected
  - Stories are in correct technical sequence
- Write brief review to `Projects/{project}/Backlog/Refinement/executive-input-{date}.md` under "## CTO Final Review"
- IF concerns found: flag specific stories and issues
- Returns: "Approved." or "Concerns: {list of stories + issues}."

Spawn CPO Agent (model: opus):
- Task: Read the newly created/refined stories from Obsidian. Quick review for:
  - User value is clear in every story
  - Product vision alignment
  - Success metrics are defined at epic level
  - No feature gaps in the breakdown
- Write brief review under "## CPO Final Review"
- IF concerns found: flag specific stories and issues
- Returns: "Approved." or "Concerns: {list of stories + issues}."

Spawn Solutions Architect Agent (model: sonnet):
- Task: Read refined stories. Quick verification:
  - Do the stories match the technical blueprint from Phase 3?
  - Are dependency chains preserved correctly?
  - Are technical constraints referenced in the right stories?
- Write brief review under "## Solutions Architect Final Review"
- Returns: "Verified." or "Gaps: {list}."

Spawn Security Architect Agent (model: sonnet):
- Task: Read refined stories. Quick verification:
  - Are ALL security requirements from Phase 3 reflected as ACs in the appropriate stories?
  - Are security dependencies in the correct build order?
  - Any new security concerns based on the final story breakdown?
- Write brief review under "## Security Architect Final Review"
- Returns: "Verified." or "Missing: {list of unaddressed security requirements}."

IF any reviewer raises concerns:
  Use `AskUserQuestion` to present concerns to user:
  ```
  AskUserQuestion:
    Header: "Review Feedback"
    Body: |
      The leadership team reviewed the refined stories and has feedback:

      ### CTO Feedback
      {CTO's concerns or "Approved — no concerns"}

      ### CPO Feedback
      {CPO's concerns or "Approved — no concerns"}

      ### Solutions Architect Feedback
      {Architect's concerns or "Verified — no gaps"}

      ### Security Architect Feedback
      {Security's concerns or "Verified — all requirements covered"}

      How would you like to proceed?
    Options: ["Accept as-is", "Revise stories based on feedback", "Discuss further"]
  ```

  IF "Revise stories based on feedback":
    Re-spawn TPM + EM with feedback to revise. Max 2 revision cycles.

Output: `Phase 6: Review complete. {Approved | Revised}.`

## Phase 7: Validate Testability

For every new or refined story, check:

| Check | Required |
|-------|----------|
| Has ≥3 acceptance criteria | ✓ |
| Each AC is testable (maps to assertion) | ✓ |
| Has verify_command defined | ✓ |
| Has test_file path defined | ✓ |
| Has EM Technical Review | ✓ |
| Security ACs included (from Security Architect) | ✓ |
| Dependencies match Architecture Brief | ✓ |
| No unresolved dependencies | ✓ |
| Status is "ready" | ✓ |

IF any check fails:
- Spawn appropriate agent to fix:
  - TPM for acceptance criteria gaps or missing security ACs
  - EM for technical review gaps
  - PM for AC hardening
- Loop until all checks pass

Stories that pass all checks → status: "ready"
Stories that fail → status: "needs-refinement" (won't be picked for sprint)

## Phase 8: Persist

All changes written to Obsidian:
- Story notes created/updated
- Epic notes created/updated
- `_index.md` updated with current counts and priority order
- Executive input document persisted at `Backlog/Refinement/executive-input-{date}.md` (includes executive assessments, Architecture Brief, final reviews)
- Dependency map updated
- Architecture ADR candidates written to `Decisions/` (if any from Solutions Architect)
- Security ADR candidates written to `Decisions/` (if any from Security Architect)

Present final summary to user:
```
Refinement complete:
- {N} stories created/refined across {M} epics
- {K} stories ready for sprint
- {J} dependencies identified (technical + security)
- {L} security requirements embedded as ACs
- Executive direction: {1-line summary}
- Architecture strategy: {1-line summary}
- Next step: Run /autonexus:agile to start a sprint
```

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
depends_on: []
assigned_to: []
created: {YYYY-MM-DD}
refined: {YYYY-MM-DD}
completed: null
executive_input: "[[executive-input-{date}]]"
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

## Dependencies
{List of stories this depends on, with explanation of WHY}
- depends on STORY-{id}: {reason — e.g., "needs auth API endpoints from STORY-42"}

## Technical Notes (Architect)
{Added during refinement}

## EM Technical Review
{Added during refinement — execution clarity, effort estimate, team assignment hints}

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

## Test Cases
{Populated during sprint by Jr. QA Engineer}

## Test Case Review
{Populated during sprint by Sr. QA Engineer}

## QA Execution Log
{Populated during sprint by Jr. QA Engineer via Playwright CLI}

## Bugs
{Populated during sprint — bug reports from QA}

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
executive_input: "[[executive-input-{date}]]"
---

# Epic: {Title}

## Objective
{What this epic achieves — aligned with CEO business goals}

## Product Vision
{How this epic serves users — from CPO assessment}

## Success Metrics
- {Measurable outcome 1 — from CPO}
- {Measurable outcome 2}

## Technical Strategy
{Architecture approach — from Solutions Architect blueprint}

## Security Considerations
{Security requirements for this epic — from Security Architect assessment}

## Stories
| Story | Status | Points | Dependencies |
|-------|--------|--------|-------------|
| [[STORY-{id}|{title}]] | {status} | {pts} | {depends_on} |
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
1. [[STORY-{id}|{title}]] — {pts}pts — {status} — depends on: {list or "none"}
2. [[STORY-{id}|{title}]] — {pts}pts — {status} — depends on: {list or "none"}

## Epics
| Epic | Stories | Completed | Remaining |
|------|---------|-----------|-----------|
| [[epic-{slug}|{title}]] | {total} | {done} | {remaining} |

## Dependency Map
{Which stories block which — maintained by TPM}

## Refinement History
- {YYYY-MM-DD}: {what was refined} — Executive input: [[executive-input-{date}]]
```

## Executive Input Document Template

```markdown
---
type: refinement-executive-input
date: {YYYY-MM-DD}
project: {ProjectName}
action: {add|refine|reprioritize|split|archive|review}
scope: {epic or "all"}
---

# Executive Input — {date}

## CTO Technical Assessment
{CTO's analysis of technical landscape + feature feasibility}

## CPO Product Assessment
{CPO's analysis of product vision + user value}

## CEO Business Assessment
{CEO's synthesis of business strategy + priority direction}

## User Answers
{User's responses to executive questions}

## Executive Direction
{Unified guidance — approved scope, priorities, constraints, success criteria}

## Solutions Architect Blueprint
{Technical blueprint — architecture impact, dependency map, implementation strategy, technical constraints, complexity estimates, ADR candidates}

## Security Architect Assessment
{Threat assessment (STRIDE), per-feature security requirements, compliance considerations, security dependencies, security ADR candidates}

## Architecture Brief (Combined)
{Merged output from both architects — technical blueprint + security requirements + resolved conflicts + final dependency chain}

## CTO Final Review
{Post-refinement technical sign-off}

## CPO Final Review
{Post-refinement product sign-off}

## Solutions Architect Final Review
{Post-refinement technical blueprint verification}

## Security Architect Final Review
{Post-refinement security requirements verification}
```

## Estimated Agent Spawns Per Refinement

| Phase | Spawns |
|-------|--------|
| Phase 1: Learn | ~2-4 (Explore agents for docs + backlog reading) |
| Phase 2: Executive Assessment | 4 (CTO, CPO, CEO, CEO alignment) |
| Phase 3: Architecture Assessment | 2-3 (Solutions Architect, Security Architect, + reconciliation if conflict) |
| Phase 5: Execute (Add) | 3-4 (TPM, EM, PM, + UX if UI) |
| Phase 5: Execute (Refine) | 2 (TPM, EM) |
| Phase 5: Execute (Other) | 1-2 (TPM, + others as needed) |
| Phase 6: Executive Review | 4 (CTO, CPO, Solutions Architect, Security Architect) |
| **Total (Add new)** | **~15-19 spawns** |
| **Total (Refine)** | **~12-15 spawns** |
| **Total (Reprioritize)** | **~9-11 spawns** |

## Flags

| Flag | Purpose |
|------|---------|
| `--new "title"` | Quick-add a story with this title |
| `--story STORY-{id}` | Refine a specific story |
| `--epic epic-{slug}` | Scope to an epic |
| `--review` | Review full backlog status |
| `--archive STORY-{id}` | Archive a specific story |
| `--skip-exec` | Skip executive assessment (for quick adds — NOT recommended) |

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Stories without test contracts | Can't verify implementation; sprint execution will fail |
| Acceptance criteria that say "should work" | Not testable; TPM must rewrite with PM hardening |
| Stories >13 points | Too large to complete in sprint; must split |
| Skipping architecture assessment | Technical constraints and security requirements not captured in stories — discovered too late during implementation |
| Skipping executive assessment | Stories lack strategic alignment, dependencies missed, user value unclear |
| Executives writing stories directly | Executives assess and direct; architects translate to blueprints; TPM writes stories, EM reviews technically |
| Architects writing stories directly | Architects provide blueprints and constraints; TPM translates to stories |
| Asking user questions one at a time | Consolidate all executive questions into ONE AskUserQuestion call |
| Ignoring dependency links | Features built on missing foundations will fail in implementation |
| Refining without learning the project first | Executives and architects can't give informed input without understanding what exists |
| Security requirements as afterthought | Bolting security onto stories during code review is expensive — Security Architect bakes it in during refinement |
| Skipping architecture alignment check | Solutions Architect and Security Architect may contradict each other — reconcile before TPM writes stories |
