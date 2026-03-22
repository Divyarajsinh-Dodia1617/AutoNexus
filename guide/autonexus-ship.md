# Guide: /autonexus:ship

Universal shipping workflow that takes any artifact from "done" to "deployed/published/delivered" through a structured 8-phase process with Obsidian-backed ship history.

## What It Does

Applies a universal shipping pattern to any domain — code, content, marketing, sales, research, or design. The workflow ensures nothing ships without a checklist, dry-run validation, and post-ship verification. Obsidian stores ship records as Decision notes for traceability across sessions.

## 8 Phases

1. **Identify** — Auto-detect what you are shipping (code PR, release, deployment, content, etc.)
2. **Inventory** — Assess current state + query Obsidian Knowledge Center and recent Decisions to understand what changed since last ship
3. **Checklist** — Generate domain-specific pre-ship gates + query Obsidian for past ship logs to inform the checklist with recurring failure patterns
4. **Prepare** — Iterative loop to fix failing checklist items (bounded or unbounded)
5. **Dry-run** — Simulate the ship action without side effects
6. **Ship** — Execute the actual delivery
7. **Verify** — Post-ship health check (with optional monitoring)
8. **Log** — Record locally to TSV + write ship decision to Obsidian at `Projects/{ProjectName}/Decisions/{date}-ship-{slug}.md`

## 9 Shipment Types

| Type | Domain | What Gets Checked |
|------|--------|-------------------|
| `code-pr` | Software | Tests, lint, types, PR description, secrets scan, review |
| `code-release` | Software | Version bump, changelog, migrations, dependency audit, tag |
| `deployment` | DevOps | Build, env vars, health check, rollback plan, feature flags |
| `content` | Publishing | Title, links, images/alt text, meta description, grammar |
| `marketing-email` | Marketing | Subject line, links/UTM, unsubscribe, CAN-SPAM, responsive |
| `marketing-campaign` | Marketing | Assets, tracking, audience, budget, landing page, schedule |
| `sales` | Sales | Pricing, branding, contact info, CTA, case studies, format |
| `research` | Academic | Abstract, citations, data sources, methodology, figures |
| `design` | Design | Export formats, responsive variants, WCAG contrast, brand |

## Flags

| Flag | Purpose |
|------|---------|
| `--dry-run` | Run all phases except actual ship (stop at Phase 5) |
| `--auto` | Auto-approve dry-run gate if no errors found |
| `--force` | Skip non-critical checklist items (blockers still enforced) |
| `--rollback` | Undo the last ship action (if reversible) |
| `--monitor N` | Post-ship monitoring for N minutes |
| `--type <type>` | Override auto-detection with explicit shipment type |
| `--checklist-only` | Only generate and evaluate checklist (stop at Phase 3) |
| `--iterations N` | Run exactly N preparation iterations before shipping |

## Syntax

```
/autonexus:ship [flags]
```

## Examples

**Ship interactively (prompted for type and mode):**
```
/autonexus:ship
```
Scans for staged changes, open PRs, and recent commits, then asks what you are shipping and how.

**Ship a code PR with auto-approve:**
```
/autonexus:ship --type code-pr --auto
```
Auto-detects the PR, runs the checklist, dry-runs, and ships if all checks pass.

**Dry-run a deployment:**
```
/autonexus:ship --type deployment --dry-run
```
Runs all checks and simulates the deployment without actually deploying.

**Check readiness only:**
```
/autonexus:ship --checklist-only
```
Generates the domain-specific checklist, evaluates it, and stops. No shipping.

**Ship with preparation loop and monitoring:**
```
/autonexus:ship --type code-release --monitor 10
Iterations: 5
```
Runs 5 preparation iterations to fix checklist items, ships the release, then monitors for 10 minutes.

## Obsidian Integration

**What gets read (Phase 2 + Phase 3):**
- `Projects/{ProjectName}/Knowledge/Architecture.md` — understand system components
- Recent Decision notes — identify what changed since last ship
- Past ship logs (tagged `autonexus/ship`) — detect recurring checklist failures

**What gets written (Phase 8):**
- Ship record at `Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-ship-{slug}.md` containing:
  - What was shipped (type, target, description)
  - Full checklist results (each item with pass/fail)
  - Preparation loop summary (items fixed during prep)
  - Dry-run results
  - Verification status
  - Changes since last ship
  - Rollback plan
- Tagged with `autonexus/decision`, `autonexus/ship`, `autonexus/ship-record`
- All writes use retry-then-queue protocol — Obsidian failures never block the ship

**Why it matters:** Past ship records inform future checklists. If an item failed in 3 of the last 5 ships, it gets elevated to priority in the current checklist.

## Local Output

Creates `ship/{YYMMDD}-{HHMM}-{ship-slug}/` with:
- `checklist.md` — full checklist with pass/fail status
- `ship-log.tsv` — iteration log (if preparation loop ran)
- `summary.md` — final ship report

Also appends to `ship-log.tsv` at the project root for cumulative history.

## Composite Metric

For the preparation loop:
```
ship_score = (checklist_passing / checklist_total) * 80
           + (dry_run_passed ? 15 : 0)
           + (no_blockers ? 5 : 0)
```
- **100** = fully ready to ship
- **80-99** = ready with minor items (can ship with `--force`)
- **<80** = not ready, continue preparing

## Chain Patterns

**Fix then ship:**
```
/autonexus:security --fix --iterations 10
/autonexus:ship --type code-pr --auto
```
Fix security findings, then ship the fixes as a PR.

**Plan then ship:**
```
/autonexus:plan
/autonexus:ship --type code-release
```
Plan the implementation, then ship the release.

**Predict then ship (risk check):**
```
/autonexus:predict
Focus: deployment risks
/autonexus:ship --type deployment --dry-run
```
Get multi-persona risk predictions, then dry-run the deployment to validate.

**Ship with rollback:**
```
/autonexus:ship --type deployment
# If something goes wrong:
/autonexus:ship --rollback
```

## Tips

- When invoked without flags, the interactive setup asks 3 questions in a single prompt — answer all at once.
- Use `--checklist-only` to check readiness before committing to a ship. Useful for "are we ready?" assessments.
- The `--force` flag skips non-critical items but still enforces blockers (security issues, broken builds). It is not a "skip everything" flag.
- Non-reversible actions (emails, some publications) are always flagged before the ship phase. You get a confirmation prompt.
- Use `--monitor 10` for deployments to watch health metrics for 10 minutes after shipping.
- Past ship records in Obsidian build institutional memory — the checklist gets smarter over time by learning from past failures.
