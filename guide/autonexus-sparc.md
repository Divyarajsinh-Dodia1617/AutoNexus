# /autonexus:sparc — SPARC 5-Phase Development

Runs a structured 5-phase development workflow — Specification, Pseudocode, Architecture, Refinement, Completion — with dedicated agents for each phase and quality gates between them. Best for medium-to-large features that benefit from rigorous planning before implementation.

## Syntax

```
# Full SPARC workflow for a feature
/autonexus:sparc Build user authentication with OAuth2

# Start from a specific phase
/autonexus:sparc Implement caching layer --skip-to architecture

# Run only one phase
/autonexus:sparc Add payment webhooks --phase specification

# Run phases in parallel where safe
/autonexus:sparc Migrate to GraphQL --parallel

# Specify agents explicitly
/autonexus:sparc Build notification system --agents "spec-writer,architect,cto"
```

## What It Does

1. **Specification** — Spec Writer agent captures requirements, acceptance criteria, constraints, and edge cases. Produces a formal spec document.
2. **Pseudocode** — Pseudocode Agent translates the spec into language-agnostic pseudocode with data structures, algorithms, and interface contracts.
3. **Architecture** — Architect agent designs the system: component diagram, data flow, API contracts, technology choices. CTO reviews the architecture as a quality gate.
4. **Refinement** — Code Agent implements the feature. QA Agent writes tests. Debug Agent runs them. Iterates until tests pass.
5. **Completion** — CTO performs final review. Learn Agent generates documentation. Ship Agent prepares the deliverable.

Each phase has a **quality gate** — the CTO agent reviews output before the next phase begins. If a gate fails, the phase is re-run with feedback.

## Flags

| Flag | Purpose |
|------|---------|
| `--phase <name>` | Run only the specified phase (specification, pseudocode, architecture, refinement, completion) |
| `--skip-to <name>` | Skip earlier phases and start from the named phase (uses existing artifacts) |
| `--parallel` | Run independent phases concurrently where safe (e.g., pseudocode + architecture) |
| `--agents <list>` | Override the default agent assignments for phases |

## Examples

```bash
# Full SPARC workflow — most common usage
/autonexus:sparc Build user authentication with OAuth2

# Output from each phase:
# Phase 1 — Specification
#   ✓ Spec Writer: 14 requirements, 6 acceptance criteria, 3 constraints
#   ✓ Quality Gate: CTO approved specification
#
# Phase 2 — Pseudocode
#   ✓ Pseudocode Agent: 4 modules, 12 functions, 3 data structures
#   ✓ Quality Gate: CTO approved pseudocode
#
# Phase 3 — Architecture
#   ✓ Architect: component diagram, API contracts, tech stack decisions
#   ✓ Quality Gate: CTO approved architecture
#
# Phase 4 — Refinement
#   ✓ Code Agent: 8 files created, 340 lines
#   ✓ QA Agent: 22 tests written
#   ✓ Debug Agent: 22/22 tests passing (2 iterations)
#   ✓ Quality Gate: CTO approved implementation
#
# Phase 5 — Completion
#   ✓ CTO: final review passed
#   ✓ Learn Agent: API docs + usage guide generated
#   ✓ Ship Agent: PR #47 created

# Resume from architecture phase (spec + pseudocode already done)
/autonexus:sparc Implement caching layer --skip-to architecture

# Run just the spec phase to validate requirements before committing
/autonexus:sparc Add payment webhooks --phase specification
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Specification document | `Projects/{ProjectName}/SPARC/{feature}/Specification.md` | After Phase 1 |
| Pseudocode document | `Projects/{ProjectName}/SPARC/{feature}/Pseudocode.md` | After Phase 2 |
| Architecture document | `Projects/{ProjectName}/SPARC/{feature}/Architecture.md` | After Phase 3 |
| Refinement log | `Projects/{ProjectName}/SPARC/{feature}/Refinement-Log.md` | During Phase 4 (updated per iteration) |
| Completion summary | `Projects/{ProjectName}/SPARC/{feature}/Completion.md` | After Phase 5 |

### What Gets Read
- Previous phase artifacts when using `--skip-to`
- Existing project context from `Projects/{ProjectName}/Sessions/`

## Without Obsidian

- All 5 phases execute normally using git state and in-memory context
- `--skip-to` requires Obsidian (needs prior phase artifacts)
- No SPARC documents are persisted, but code changes are committed as usual

## Tips

- **Use SPARC for medium+ features.** For quick bug fixes, `/autonexus:fix` is faster. For small enhancements, `/autonexus` with a clear metric works well.
- **Each phase has a quality gate** — the CTO agent reviews before proceeding. This catches design issues early rather than after implementation.
- **Run `--phase specification` first** if you want to validate requirements with your team before committing to a full build.
- **`--skip-to` is useful for iteration** — if architecture needs rework, skip to that phase without re-running the spec.
- **Pair with `/autonexus:predict`** before SPARC to get multi-persona input on the approach, then feed that into the specification phase.
