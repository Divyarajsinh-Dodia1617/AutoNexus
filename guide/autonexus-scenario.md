# /autonexus:scenario — Scenario-Driven Use Case Generator

Autonomously explores situations, edge cases, failure modes, and derivative scenarios from a seed scenario. Doesn't stop at obvious paths — iteratively discovers what could go wrong, what's missing, and what nobody thought of. High-severity findings are persisted to Obsidian.

## Syntax

```
/autonexus:scenario
Scenario: User attempts to checkout with multiple payment methods
Domain: software
Depth: deep

/autonexus:scenario --domain security --depth shallow
Scenario: Admin resets user password

/autonexus:scenario --format test-scenarios --focus edge-cases
Iterations: 20
```

## What It Does

7 phases per iteration:

1. **Seed** — parse and analyze the scenario; query Obsidian for past explorations on same feature to avoid duplicate work
2. **Decompose** — break into 12 exploration dimensions
3. **Generate** — create ONE new concrete situation (specific trigger, flow, expected outcome)
4. **Classify** — new / variant / duplicate / out-of-scope / low-value
5. **Expand** — derive edge cases, what-ifs, boundary conditions, failure injections
6. **Log** — record to `scenario-results.tsv`; write Critical/High scenarios to Obsidian Decisions/
7. **Repeat** — pick next unexplored dimension/combination, loop

On session end, writes an exploration summary with a dimension coverage map to Obsidian.

## 12 Exploration Dimensions

| Dimension | What It Explores |
|-----------|-----------------|
| Happy path | Normal successful flow |
| Error path | Expected, handled failures |
| Edge case | Boundary conditions (min/max, empty, unicode) |
| Abuse/misuse | Injection, privilege escalation, rate abuse |
| Scale | Concurrent users, large datasets, burst traffic |
| Concurrent | Race conditions, simultaneous edits, distributed locks |
| Temporal | Timeouts, expiry, scheduling, timezone edges |
| Data variation | Null, special chars, max length, format mismatches |
| Permission | Role escalation, shared resources, delegation |
| Integration | External API failures, version mismatches, timeouts |
| Recovery | Crash recovery, retry logic, data consistency after failure |
| State transition | Invalid state transitions, partial updates, rollback |

## 5 Domain Templates

| Domain | Priority Dimensions | Default Format |
|--------|-------------------|----------------|
| **Software/API** | error_path, edge_case, concurrent, integration | test-scenarios |
| **Product/UX** | happy_path, error_path, permission, temporal | user-stories |
| **Business/Process** | happy_path, error_path, permission, recovery | use-cases |
| **Security/Compliance** | abuse, permission, data_variation, concurrent | threat-scenarios |
| **Marketing/Sales** | happy_path, data_variation, temporal, scale | user-stories |

## Depth Presets

| Depth | Iterations | Use When |
|-------|-----------|----------|
| Shallow | 10 | Quick overview, time-constrained |
| Standard | 25 | Recommended for most scenarios |
| Deep | 50+ | Comprehensive coverage, critical features |
| Unlimited | Until interrupted | Exhaustive exploration |

## Flags

| Flag | Purpose |
|------|---------|
| `--domain <type>` | Set domain: software, product, business, security, marketing |
| `--depth <level>` | Exploration depth: shallow (10), standard (25), deep (50+) |
| `--scope <glob>` | Limit to specific files/features for codebase-aware generation |
| `--format <type>` | Output: use-cases, user-stories, test-scenarios, threat-scenarios, mixed |
| `--focus <area>` | Prioritize: edge-cases, failures, security, scale |

## Examples

```bash
# Quick security scan of auth flow
/autonexus:scenario --domain security --depth shallow
Scenario: User authenticates via OAuth2

# Deep exploration of payment system
/autonexus:scenario --domain software --depth deep --focus failures
Scenario: User completes checkout with split payment across credit card and store credit

# Generate formal test scenarios for CI
/autonexus:scenario --format test-scenarios --domain software
Iterations: 30
Scenario: File upload and processing pipeline

# Business process stress test
/autonexus:scenario --domain business --focus edge-cases
Scenario: Employee submits expense report requiring multi-level approval

# Broad UX exploration
/autonexus:scenario --domain product
Scenario: New user onboarding flow for mobile app
```

## Obsidian Integration

### What Gets Written

- **High-severity scenarios (Critical/High):** Written as decision notes at `Projects/{ProjectName}/Decisions/{date}-scenario-{slug}.md` with full situation format, actors, flow, and risk assessment
- **Exploration summary:** Written at session end with dimension coverage map showing which dimensions were explored, gap areas, severity distribution, and composite score
- **Daily note entry:** Appended with scenario count, dimension coverage, and link to full summary

### Deduplication

Phase 1 queries Obsidian for past scenario explorations on the same feature/domain. If prior coverage exists, already well-explored dimensions are deprioritized so iterations focus on gaps.

### Resilience

All Obsidian writes use retry-then-queue protocol. If Obsidian is unavailable, the local `scenario-results.tsv` and output files are unaffected.

## Chain Patterns

```bash
# Explore scenarios, then hunt for bugs in those areas
/autonexus:scenario --domain software
Iterations: 25
/autonexus:debug --scope src/checkout/**

# Explore, then security audit the weak spots
/autonexus:scenario --domain security
Iterations: 15
/autonexus:security --scope src/auth/**

# Scenario → predict pipeline (find scenarios, then get expert opinions)
/autonexus:scenario --domain software --depth shallow
Iterations: 10
/autonexus:predict --scope src/**
```

## Tips

- **Start with `--depth shallow`** to get a quick dimension sweep, then go deeper on interesting areas.
- **Use `--domain`** to get domain-appropriate dimension prioritization and output format.
- **Use `--focus`** when you already know which dimension class matters most (e.g., `--focus security` for a pre-launch audit).
- **Check Obsidian after a run** — high-severity scenario notes link to specific decision records you can reference in planning.
- **Combine dimensions** emerge naturally in mid-to-late iterations (e.g., edge_case + concurrent). Let the exploration reach them organically rather than forcing it.
- **Bounded mode** (`Iterations: N`) is recommended for most uses — unbounded mode rarely adds value past 50 iterations.
- The composite metric incentivizes both breadth (dimension coverage) and depth (edge case discovery). Aim for 70%+ dimension coverage.
