# Ship Workflow — /autonexus:ship

Universal shipping workflow that applies autonexus loop principles to the last mile — taking anything from "done" to "deployed/published/delivered." Works for code, content, marketing, sales, research, design, or any artifact that needs to reach its audience. Obsidian serves as the persistent knowledge backbone for ship history and checklists.

**Core idea:** Shipping has a universal pattern regardless of domain. Identify → Inventory → Checklist → Prepare → Dry-run → Ship → Verify → Log.

## Trigger

- User invokes `/autonexus:ship`
- User says "ship it", "deploy this", "publish this", "launch this", "release this"
- User says "get this out the door", "push to prod", "send this out", "go live"

## Loop Support

Works with bounded mode for iterative pre-ship preparation:

```
# Ship with automatic preparation loop
/autonexus:ship

# Bounded preparation — iterate N times before shipping
/autonexus:ship
Iterations: 10

# Ship specific artifact
/autonexus:ship
Target: src/features/auth/**
Destination: production
```

## PREREQUISITE: Interactive Setup (when invoked without flags)

**CRITICAL — BLOCKING PREREQUISITE:** If `/autonexus:ship` is invoked without `--type` or target, you MUST scan for staged changes, open PRs, and recent commits, then use `AskUserQuestion` to gather user input BEFORE proceeding to ANY phase. DO NOT skip this step.

**Single batched call — all 3 questions at once:**

You MUST call `AskUserQuestion` with all 3 questions in ONE call:

| # | Header | Question | Options (from context scan) |
|---|--------|----------|----------------------------|
| 1 | `What` | "What are you shipping?" | "Code PR", "Release / version tag", "Deployment to production", "Blog post / documentation" |
| 2 | `Mode` | "How should I ship it?" | "Full workflow (checklist → dry-run → ship → verify)", "Dry-run only (validate without shipping)", "Checklist only (just check readiness)", "Auto-approve (ship if checklist passes)" |
| 3 | `Monitor` | "Post-ship monitoring?" | "No monitoring", "5 minutes", "10 minutes", "30 minutes" |

**IMPORTANT:** Always ask all questions in a single call — never one at a time.

If `--type`, `--dry-run`, `--auto`, or `--checklist-only` flags are provided, skip interactive setup and proceed directly.

## Architecture

```
/autonexus:ship
  ├── Phase 1: Identify (what are we shipping?)
  ├── Phase 2: Inventory (current state + Obsidian Knowledge Center query)
  ├── Phase 3: Checklist (domain-specific pre-ship gates + Obsidian past ship logs)
  ├── Phase 4: Prepare (autonexus loop until checklist passes)
  ├── Phase 5: Dry-run (simulate the ship action)
  ├── Phase 6: Ship (execute the actual delivery)
  ├── Phase 7: Verify (post-ship health check)
  └── Phase 8: Log (record the shipment + write to Obsidian Decisions)
```

---

## Phase 1: Identify — What Are We Shipping?

Auto-detect the shipment type from context, or ask the user.

**Detection algorithm:**
```
FUNCTION detectShipmentType(context):
  # Check explicit user input first
  IF user specifies type → USE IT

  # Auto-detect from context
  IF git diff has staged changes OR user mentions "deploy/release/merge":
    IF has Dockerfile/k8s/deploy configs → "deployment"
    IF has open PR or branch changes → "code-pr"
    ELSE → "code-release"

  IF context mentions "blog/article/post" OR target files are *.md in content/:
    → "content"

  IF context mentions "email/campaign/newsletter":
    → "marketing-email"

  IF context mentions "landing page/ad/social":
    → "marketing-campaign"

  IF context mentions "deck/proposal/pitch/quote":
    → "sales"

  IF context mentions "paper/report/analysis/findings":
    → "research"

  IF context mentions "assets/mockup/design/figma":
    → "design"

  # Default: ask user
  → ASK "What are you shipping? (code/content/marketing/sales/research/design/other)"
```

**Output:** `Phase 1: Identified shipment — [type]: [brief description]`

---

## Phase 2: Inventory — Current State Assessment + Obsidian Query

Scan the artifact and its environment to understand readiness. Query Obsidian for additional context.

### Step 2a: Query Obsidian Knowledge Center

If `OBSIDIAN_AVAILABLE = true` (set at Phase 0 per obsidian-integration.md):

```
TRY (retry-then-queue):
  # Read architecture to understand what components exist
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Knowledge/Architecture.md",
    format: "markdown"
  })

  # Read recent Decisions to understand what changed since last ship
  obsidian_global_search({
    query: "ship",
    searchInPath: "Projects/{ProjectName}/Decisions/",
    contextLength: 300,
    pageSize: 10
  })

IF Knowledge Center and recent Decisions found:
  - Use architecture overview to understand system components and their dependencies
  - Use recent Decision notes to identify what has changed since last ship
  - Cross-reference with git log to build a complete changelog
  - Note any recent security audit decisions that may affect ship readiness
  LOG: "Loaded project context from Obsidian — last ship was {date}, {N} decisions since then"

IF no data found:
  - Proceed with standard inventory scan
  LOG: "No Obsidian ship history found — running standard inventory"
```

### Step 2b: Standard Inventory Checks

**For each shipment type, gather:**

| Type | Inventory Checks |
|------|-----------------|
| code-pr | Changed files, test status, lint status, PR description, review status |
| code-release | Version tag, changelog, migration status, dependency audit |
| deployment | Build status, env vars, infra health, rollback plan |
| content | Word count, links checked, images present, metadata/frontmatter |
| marketing-email | Subject line, preview text, links, unsubscribe, CAN-SPAM compliance |
| marketing-campaign | Assets ready, tracking pixels, UTM params, A/B variants |
| sales | Pricing current, branding consistent, contact info, CTA clear |
| research | Citations complete, methodology documented, data sources linked |
| design | Formats exported, responsive variants, accessibility checked |

**Output:** `Phase 2: Inventory complete — [N] items assessed, [M] gaps found`

---

## Phase 3: Checklist — Domain-Specific Pre-Ship Gates + Obsidian History

Generate a mechanical checklist based on shipment type. Every item must be verifiable (pass/fail).

### Step 3a: Query Obsidian for Past Ship Logs

If `OBSIDIAN_AVAILABLE = true`:

```
TRY (retry-then-queue):
  obsidian_global_search({
    query: "autonexus/ship checklist",
    searchInPath: "Projects/{ProjectName}/Decisions/",
    contextLength: 300,
    pageSize: 5
  })

IF past ship logs found:
  - Extract checklist items that previously FAILED
  - Add those as priority items in the current checklist
  - Note patterns: "This item has failed in {N} of last {M} ships"
  - If a checklist item was manually overridden before, flag it
  LOG: "Informed checklist with {N} past ship records — {M} recurring failure patterns detected"
```

### Code Checklists

**code-pr:**
- [ ] All tests pass (`npm test` / `pytest` / language-specific)
- [ ] Lint clean (no errors, warnings acceptable)
- [ ] Type check passes (if applicable)
- [ ] PR description explains the "why"
- [ ] No secrets in diff (`git diff --cached | grep -i "password\|secret\|api_key"`)
- [ ] No TODO/FIXME in new code (or documented as intentional)
- [ ] Breaking changes documented (if any)
- [ ] Reviewer assigned or review complete

**code-release:**
- [ ] All code-pr checks pass
- [ ] Version bumped in package.json/pyproject.toml/Cargo.toml
- [ ] CHANGELOG updated with release notes
- [ ] Migration scripts tested (if DB changes)
- [ ] Dependency audit clean (`npm audit` / `pip audit`)
- [ ] Tag created matching version

**deployment:**
- [ ] All code-release checks pass
- [ ] Build succeeds in CI
- [ ] Environment variables set for target env
- [ ] Health check endpoint responds
- [ ] Rollback plan documented
- [ ] Monitoring/alerting configured
- [ ] Feature flags set correctly

### Content Checklists

**content (blog/docs):**
- [ ] Title present and descriptive
- [ ] No broken links (internal or external)
- [ ] Images have alt text
- [ ] Meta description present (<=160 chars)
- [ ] No placeholder text ("Lorem ipsum", "TODO", "TBD")
- [ ] Grammar/spell check passes
- [ ] Publish date set
- [ ] Author attribution present

### Marketing Checklists

**marketing-email:**
- [ ] Subject line present (<=60 chars recommended)
- [ ] Preview text set
- [ ] All links working and tracked (UTM parameters)
- [ ] Unsubscribe link present and functional
- [ ] Physical address included (CAN-SPAM)
- [ ] Responsive on mobile (test render)
- [ ] Plain text fallback exists
- [ ] Sender name and reply-to configured

**marketing-campaign:**
- [ ] All creative assets finalized
- [ ] Tracking pixels/UTM parameters configured
- [ ] Target audience defined and segmented
- [ ] Budget allocated and approved
- [ ] Landing page live and tested
- [ ] A/B test variants set (if applicable)
- [ ] Schedule confirmed

### Sales Checklists

**sales (deck/proposal):**
- [ ] Company/prospect name correct throughout
- [ ] Pricing is current and approved
- [ ] Contact information accurate
- [ ] Branding consistent (logos, colors, fonts)
- [ ] No competitor names misspelled
- [ ] CTA is clear and actionable
- [ ] Attached case studies/testimonials current
- [ ] File format appropriate (PDF for external, editable for internal)

### Research Checklists

**research (paper/report):**
- [ ] Abstract/executive summary present
- [ ] All citations properly formatted
- [ ] Data sources linked and accessible
- [ ] Methodology section complete
- [ ] Figures/charts labeled and referenced
- [ ] Conclusion addresses stated hypothesis
- [ ] Acknowledgments included
- [ ] No placeholder references ("[citation needed]")

### Design Checklists

**design (assets/mockups):**
- [ ] All requested formats exported (PNG, SVG, PDF)
- [ ] Responsive variants provided (mobile, tablet, desktop)
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] No placeholder images or text
- [ ] Source files organized and named
- [ ] Brand guidelines followed
- [ ] Handoff notes/specs documented

**Output:** `Phase 3: Checklist generated — [N] items, [P] passing, [F] failing`

---

## Phase 4: Prepare — Iterative Improvement Loop

Apply the autonexus loop to fix failing checklist items.

```
metric = count_passing_checklist_items / total_checklist_items * 100
direction = higher_is_better
target = 100 (all items pass)

LOOP (until all pass OR max iterations):
  1. Read checklist status
  2. Pick highest-priority failing item
  3. Fix it (one atomic change)
  4. Re-run checklist verification
  5. IF item now passes → keep, log "fixed: [item]"
  6. IF item still fails → revert, try different approach
  7. IF all items pass → EXIT LOOP with "ready to ship"
```

**Priority order for fixes:**
1. **Blockers** — security issues, broken builds, missing critical content
2. **Required** — tests, lint, links, compliance items
3. **Recommended** — descriptions, documentation, polish

**Auto-fix capabilities:**
- Run test suites and fix failures
- Fix lint errors automatically
- Add missing meta descriptions
- Generate changelog entries from git log
- Check and fix broken links
- Add alt text to images (describe or prompt user)
- Format citations
- Export missing design formats

**Items that require human input:**
- Pricing approval
- Legal review sign-off
- Brand approval
- Strategic decisions (A/B test variants)
- Flag these and ask user, don't block on them

**Output:** `Phase 4: Preparation complete — [N/N] checklist items passing`

---

## Phase 5: Dry-Run — Simulate Before Shipping

Execute a simulation of the ship action without side effects.

| Type | Dry-Run Action |
|------|---------------|
| code-pr | `gh pr create --draft` or preview PR diff |
| code-release | Create tag locally (don't push), preview changelog |
| deployment | Build Docker image, run health checks locally |
| content | Preview render, check all links resolve |
| marketing-email | Send test email to sender's own address |
| marketing-campaign | Preview in ad platform, estimate reach |
| sales | Preview PDF render, check all pages |
| research | Export to final format, check pagination |
| design | Preview all exported formats, check dimensions |

**Dry-run gate:**
- Present dry-run results to user
- `--auto` flag: auto-approve if no errors
- Default: ask user "Ready to ship?" before proceeding
- `--dry-run` flag: stop here, don't actually ship

**Output:** `Phase 5: Dry-run complete — [result summary]`

---

## Phase 6: Ship — Execute the Delivery

The actual ship action. Domain-specific.

| Type | Ship Action |
|------|------------|
| code-pr | `gh pr create` with full description, request reviewers |
| code-release | `git tag`, `git push --tags`, create GitHub release |
| deployment | `git push` to deploy branch, trigger CI/CD, or `kubectl apply` |
| content | Publish via CMS API, or commit to content branch |
| marketing-email | Send via ESP API (SendGrid, Mailchimp, etc.) |
| marketing-campaign | Activate campaign in ad platform |
| sales | Send email with attachment, or share link |
| research | Upload to repository, submit to journal/platform |
| design | Upload to asset library, share with stakeholders |

**Safety rails:**
- Confirm target (staging vs production, draft vs publish)
- Log the exact command/action taken
- Record timestamp
- Capture any response/confirmation IDs

**Output:** `Phase 6: Shipped — [action taken] at [timestamp]`

---

## Phase 7: Verify — Post-Ship Health Check

Confirm the shipment actually landed and is healthy.

| Type | Verification |
|------|-------------|
| code-pr | PR created, CI running, link accessible |
| code-release | Tag visible, release page published, assets attached |
| deployment | Health endpoint returns 200, no error spike in logs |
| content | Page loads, links work, appears in sitemap |
| marketing-email | Delivery rate > 95%, no bounce spike |
| marketing-campaign | Ads serving, landing page loading, tracking firing |
| sales | Email delivered, link tracking active |
| research | Accessible via URL/DOI, properly indexed |
| design | Assets downloadable, correct dimensions |

**Post-ship monitoring (with `--monitor N` flag):**
```
FOR N minutes:
  Check health metrics every 60 seconds
  IF anomaly detected → ALERT user immediately
  Log metrics to ship-log
```

**Output:** `Phase 7: Verified — [health status summary]`

---

## Phase 8: Log — Record the Shipment + Obsidian Decision

Create a ship log entry for traceability, both locally and in Obsidian.

### Local Log

**Log format (append to `ship-log.tsv`):**
```tsv
timestamp	type	target	checklist_score	dry_run	shipped	verified	duration	notes
2026-03-22T14:30:00Z	code-pr	#42	18/18	pass	pass	pass	4m32s	auth feature PR
```

### Obsidian Ship Record

If `OBSIDIAN_AVAILABLE = true`, write the ship record to Obsidian using retry-then-queue:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-ship-{slug}.md",
  content: "---
tags: [autonexus/decision, autonexus/ship, autonexus/{project}]
date: {YYYY-MM-DD}
status: accepted
type: ship
shipment_type: {code-pr|code-release|deployment|content|...}
target: {what was shipped}
checklist_score: {P}/{T}
dry_run: {pass|fail|skipped}
shipped: {pass|fail}
verified: {pass|fail|skipped}
duration: {total time}
session: {session-id}
commit_hash: {git rev-parse HEAD}
---

# Ship: {slug}

## What Was Shipped
- **Type:** {shipment type}
- **Target:** {specific target — PR number, version tag, deployment target, etc.}
- **Description:** {one-line description of what was shipped}
- **Timestamp:** {ISO timestamp}

## Checklist Results

| # | Item | Status |
|---|------|--------|
| 1 | {checklist item 1} | Pass |
| 2 | {checklist item 2} | Pass |
| 3 | {checklist item 3} | Pass (fixed in prep) |
| ... | ... | ... |

**Score:** {P}/{T} items passing
**Items fixed during preparation:** {list any items that were auto-fixed}
**Items skipped (--force):** {list any items skipped, if applicable}

## Preparation Loop
- **Iterations:** {N} (if preparation loop ran)
- **Items fixed:** {list of checklist items fixed during prep}

## Dry-Run
- **Result:** {pass|fail|skipped}
- **Details:** {dry-run output summary}

## Verification
- **Result:** {pass|fail|skipped}
- **Health check:** {verification details}
- **Monitoring:** {monitoring duration and results, if --monitor was used}

## Changes Since Last Ship
{Summary of what changed between this ship and the previous one, informed by Obsidian history}

## Rollback Plan
{How to undo this shipment if needed}
",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

### Obsidian: Tag the Ship Record

```
obsidian_manage_tags({
  filePath: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-ship-{slug}.md",
  operation: "add",
  tags: ["autonexus/ship-record"]
})
```

**Summary output:**
```
=== Ship Complete ===
Type: [shipment type]
Target: [where it went]
Checklist: [P/T] items passed
Duration: [total time]
Status: SHIPPED
Obsidian: Decision record written to Projects/{ProjectName}/Decisions/{date}-ship-{slug}.md
```

---

## Flags

| Flag | Purpose |
|------|---------|
| `--dry-run` | Run all phases except actual ship (stop at Phase 5) |
| `--auto` | Auto-approve dry-run gate if no errors found |
| `--force` | Skip non-critical checklist items (still enforce blockers) |
| `--rollback` | Undo the last ship action (if reversible) |
| `--monitor N` | Post-ship monitoring for N minutes |
| `--type <type>` | Override auto-detection with explicit shipment type |
| `--checklist-only` | Only generate and evaluate checklist (stop at Phase 3) |

---

## Composite Metric

For bounded loop mode, the ship readiness metric:

```
ship_score = (checklist_passing / checklist_total) * 80
           + (dry_run_passed ? 15 : 0)
           + (no_blockers ? 5 : 0)
```

- **100** = fully ready to ship
- **80-99** = ready with minor items (can ship with `--force`)
- **<80** = not ready, continue preparing

---

## Rollback Protocol

If `--rollback` is specified or post-ship verification fails:

| Type | Rollback Action |
|------|----------------|
| code-pr | Close PR: `gh pr close` |
| code-release | Delete tag: `git tag -d` + `git push --delete origin` |
| deployment | Revert deploy: `git revert` or `kubectl rollback` |
| content | Unpublish/revert to draft |
| marketing-email | Cannot rollback (flag as "sent") |
| marketing-campaign | Pause campaign in ad platform |
| sales | Send correction/follow-up |
| research | Request retraction or update |
| design | Revert to previous version in asset library |

**Non-reversible actions** (email, some publications) are flagged before Phase 6 ship action.

---

## 9 Shipment Types Reference

| Type | Domain | Key Phases |
|------|--------|-----------|
| `code-pr` | Software | Tests, lint, PR description, review |
| `code-release` | Software | Version bump, changelog, tag, dependency audit |
| `deployment` | DevOps | Build, env vars, health check, rollback plan |
| `content` | Publishing | Links, metadata, grammar, images |
| `marketing-email` | Marketing | Subject, links, CAN-SPAM, responsive |
| `marketing-campaign` | Marketing | Assets, tracking, audience, budget |
| `sales` | Sales | Pricing, branding, CTA, format |
| `research` | Academic | Citations, methodology, data sources |
| `design` | Design | Formats, responsive, accessibility, brand |

---

## Output Directory

Creates `ship/{YYMMDD}-{HHMM}-{ship-slug}/` with:
- `checklist.md` — full checklist with pass/fail status
- `ship-log.tsv` — iteration log (if preparation loop ran)
- `summary.md` — final ship report

---

## Error Recovery

| Error | Recovery |
|-------|----------|
| Can't detect shipment type | Ask user explicitly |
| Tests fail during preparation | Log failure, try to fix, skip after 3 attempts |
| Dry-run fails | Report failure, do not proceed to ship |
| Ship action fails | Log error, suggest manual steps |
| Post-ship verification fails | Alert user, suggest rollback |
| Obsidian unavailable | Continue without knowledge queries — local TSV + report still generated |

## Anti-Patterns

- **Do NOT ship without a checklist** — even `--force` enforces blockers
- **Do NOT skip dry-run silently** — user must opt in with `--auto` or acknowledge results
- **Do NOT ship non-reversible actions without explicit confirmation** — always flag emails, publications
- **Do NOT block the loop on Obsidian failures** — use retry-then-queue protocol for all MCP calls
- **Do NOT repeat Obsidian warnings** — one warning at Phase 0 is sufficient
