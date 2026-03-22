# Quality Gates — /autonexus:agile

Quality gates control story progression through the sprint lifecycle. Each gate defines required approvals, pass criteria, and failure handling. No story advances past a gate without meeting its requirements.

## Gate 1: Design Gate

**Position:** Between Design phase and Implementation phase

**Required approvals:**
- Architect: design document complete ✓
- CTO: architecture compliance approved ✓

**Pass criteria:**
- Design doc exists in Obsidian (`Designs/design-STORY-{id}.md`)
- CTO has written "Approved" to design doc
- No unresolved architecture concerns flagged

**Failure handling:**
- IF CTO rejects: Re-spawn Architect with feedback. Max 2 design cycles.
- IF still rejected after 2 cycles: Escalate to Principal Engineer for binding decision.
- IF Principal can't resolve: Story removed from sprint, returned to backlog with "needs-redesign" tag.

## Gate 2: Code Gate

**Position:** Between Review phase and QA phase

**Required approvals:**
- At least 1 Sr. Engineer (or higher) code review approval ✓
- Security Engineer: security review passed ✓

**Pass criteria:**
- Review note exists (`Reviews/review-STORY-{id}.md`)
- ≥1 reviewer status = "Approved" (no blocking comments unresolved)
- Security review status = "Passed"
- All blocking review comments addressed

**Failure handling:**
- IF reviewer requests changes: Re-spawn implementation agent with feedback. Re-spawn reviewer after fix.
- Max 3 review cycles.
- IF still failing after 3 cycles: Escalate to Staff Engineer.
  - Staff Engineer reads all review history, makes binding decision on remaining issues.

## Gate 3: QA Gate

**Position:** Between QA phase and Acceptance phase

**Required approvals:**
- QA: all tests passing ✓
- QA: coverage target met ✓

**Pass criteria:**
- All acceptance criteria have corresponding tests
- All tests pass (0 failures)
- Coverage ≥ story's coverage_target (default 90%)
- 0 P0 or P1 bugs outstanding
- P2 bugs documented but do not block

**Failure handling:**
- IF tests fail: Re-spawn implementation agent with bug details. Re-spawn QA after fix.
- Max 3 QA cycles.
- IF still failing: Escalate to EM.
  - EM decides: fix critical path only and defer rest, or remove story from sprint.

## Gate 4: Acceptance Gate

**Position:** Between QA phase and Done

**Required approvals:**
- PM: all acceptance criteria verified ✓

**Pass criteria:**
- Every acceptance criterion in story note is checked off
- QA Gate passed
- Code Gate passed
- Design Gate passed (verified by gate chain)

**Failure handling:**
- IF PM finds unmet criteria: Document which criteria failed.
  - If minor: re-spawn implementation agent with specific gap.
  - If major (acceptance criteria were wrong): update story, send back to design.

## Gate 5: Deploy Gate (Optional — for /autonexus:ship integration)

**Position:** After Sprint Review, before deployment

**Required approvals:**
- SRE: deployment readiness ✓
- PM: acceptance confirmed ✓
- Security: final clearance ✓

**Pass criteria:**
- All Done stories have passed Gates 1-4
- No CRITICAL security findings open
- Deployment checklist complete (monitoring, rollback plan, feature flags)

**Failure handling:**
- IF SRE rejects: Address deployment concerns. Re-run checklist.
- IF security blocks: Re-spawn Security Engineer. Escalate to Security Architect if unresolved.

## Escalation Chain

When a gate fails repeatedly (exhausts max cycles):

| Gate | Max Cycles | Escalates To | Final Fallback |
|------|-----------|-------------|----------------|
| Design | 2 | Principal Engineer | Remove from sprint |
| Code Review | 3 | Staff Engineer | Staff makes binding call |
| QA | 3 | EM | EM decides: fix or defer |
| Acceptance | 2 | PM + EM joint | Revise acceptance criteria |
| Deploy | 2 | VP Engineering | VP decides go/no-go |

## Gate Metrics (tracked in retro)

| Metric | What It Measures |
|--------|-----------------|
| First-pass rate | % stories passing gate on first attempt |
| Avg cycles per gate | How many rework rounds per story |
| Gate bottleneck | Which gate causes most delays |
| Rework cost | Extra agent spawns due to gate failures |

These metrics feed into Sprint Retrospective for process improvement.

## Gate Enforcement Rules

1. **No bypass without escalation.** Gates cannot be skipped. If a gate seems unnecessary for a story, the decision to skip must be made by the CTO and recorded as an ADR.
2. **Gates are sequential.** A story must pass Gate N before entering the phase guarded by Gate N+1.
3. **Gate state is recorded.** Each gate pass/fail is logged in the story note with timestamp and approver.
4. **Automated verification.** Where possible, gates verify mechanically (test results, coverage numbers) rather than relying on subjective assessment.
