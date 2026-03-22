# Status Workflow — /autonexus:status

Read-only dashboard that shows the current state of AutoNexus for a project. Queries Obsidian and local data to display session history, iteration stats, strategy effectiveness, knowledge freshness, and pending sync items. **Makes NO modifications** — no loop, no commits, no Obsidian writes.

**Core idea:** At-a-glance project health check. How many sessions have been run? What's the keep rate? Which strategies work best? Is the Knowledge Center fresh? Are there unresolved predict findings?

## Trigger

- User invokes `/autonexus:status`
- User says "show autonexus status", "how's the project doing", "what's my keep rate"

## Architecture

```
/autonexus:status
  ├── Step 1: Resolve ProjectName
  ├── Step 2: Query Obsidian (or local fallback)
  ├── Step 3: Calculate statistics
  └── Step 4: Print dashboard to terminal
```

No loop. No phases. Single-pass read-only operation.

## Step 1: Project Detection

Resolve the ProjectName using the standard method:

```
1. git remote get-url origin → extract repo name
2. Fallback: package.json "name" field
3. Fallback: basename of current working directory
```

If `--project <name>` flag is provided, use that name instead.

## Step 2: Query Obsidian

### If OBSIDIAN_AVAILABLE:

Execute the following queries (all use retry-then-queue protocol, but since this is read-only, failures simply result in "unavailable" fields in the dashboard):

**Query 1 — Session count + recent sessions:**
```
obsidian_global_search({
  query: "autonexus/session",
  searchInPath: "Projects/{ProjectName}/Iterations/",
  contextLength: 400,
  pageSize: 20
})
```
Parse results to extract:
- Total session count
- Recent sessions (last 5) with date, goal, baseline, final metric, keeps/discards
- Sessions this week (filter by date)

**Query 2 — Knowledge Center freshness:**
```
obsidian_manage_frontmatter({
  filePath: "Projects/{ProjectName}/Knowledge/Architecture.md",
  operation: "get",
  key: "commit_hash"
})
```
Compare the stored `commit_hash` against `git rev-parse HEAD` to determine staleness:
- If hashes match → "fresh"
- If they differ → count commits between them: `git rev-list --count {stored_hash}..HEAD`
- If Architecture.md doesn't exist → "not initialized"

**Query 3 — Recent decisions:**
```
obsidian_global_search({
  query: "autonexus/decision",
  searchInPath: "Projects/{ProjectName}/Decisions/",
  contextLength: 200,
  pageSize: 5
})
```
Parse results to extract decision titles and dates.

**Query 4 — Unresolved predict findings:**
```
obsidian_global_search({
  query: "autonexus/predict",
  searchInPath: "Projects/{ProjectName}/Predictions/",
  contextLength: 300,
  pageSize: 5
})
```
Parse results to extract finding titles, severities, and persona names. Filter for unresolved findings (no linked fix decision).

**Query 5 — Sync queue status:**
Check if local `obsidian-sync-queue.md` exists and count pending items:
```bash
if [ -f obsidian-sync-queue.md ]; then
  wc -l < obsidian-sync-queue.md
fi
```

### If OBSIDIAN unavailable (fallback):

```
1. Read autonexus-results.tsv — calculate total iterations, keeps, discards, crashes
2. Read git log --oneline -50 — count experiment commits, extract session boundaries
3. Knowledge Center: skip (requires Obsidian)
4. Decisions: skip (requires Obsidian)
5. Predictions: skip (requires Obsidian)
6. Sync queue: check local file
```

## Step 3: Calculate Statistics

From the gathered data, compute:

```
total_iterations = sum of all iteration counts across sessions
total_keeps = sum of all keeps
total_discards = sum of all discards
total_crashes = sum of all crashes
keep_rate = (total_keeps / total_iterations) * 100

# Strategy analysis (if enough iteration data)
IF total_iterations >= 10:
  Group iterations by change category (from iteration note "Change:" descriptions)
  Calculate keep rate per category
  Sort by keep rate descending
  top_strategies = top 3 categories by keep rate
```

## Step 4: Print Dashboard

Print the dashboard to the terminal. This is a READ-ONLY operation — nothing is written to Obsidian.

```
=== AutoNexus Status: {ProjectName} ===

Knowledge Center:  {fresh|stale|not initialized} ({N} commits behind, updated {date})
Sessions:          {total} total, {recent} this week
Iterations:        {total} ({keeps} keeps, {discards} discards, {crashes} crashes)
Keep Rate:         {rate}%
Sync Queue:        {N} pending items | empty

Recent Sessions:
  {date} — {goal} — {baseline}->{final} ({keeps}/{total})
  {date} — {goal} — {baseline}->{final} ({keeps}/{total})
  {date} — {goal} — {baseline}->{final} ({keeps}/{total})

Recent Decisions:
  {date} — {title}
  {date} — {title}

Unresolved Findings:
  {severity} — {title} ({persona})
  {severity} — {title} ({persona})

Top Strategies: {category} ({rate}%), {category} ({rate}%), {category} ({rate}%)
```

### Dashboard field descriptions:

| Field | Source | Fallback |
|-------|--------|----------|
| Knowledge Center | Obsidian frontmatter `commit_hash` vs current HEAD | "unavailable (no Obsidian)" |
| Sessions | Obsidian session summary count | TSV session count estimate |
| Iterations | Obsidian iteration notes or TSV row count | TSV row count |
| Keep Rate | Calculated from iteration outcomes | Calculated from TSV |
| Sync Queue | Local `obsidian-sync-queue.md` line count | "no queue file" |
| Recent Sessions | Obsidian session summaries | TSV + git log reconstruction |
| Recent Decisions | Obsidian decision notes | "unavailable (no Obsidian)" |
| Unresolved Findings | Obsidian predict notes | "unavailable (no Obsidian)" |
| Top Strategies | Obsidian iteration note analysis | "not enough data" |

## Fallback Output (Obsidian Unavailable)

When Obsidian is not available, print a reduced dashboard:

```
=== AutoNexus Status: {ProjectName} (local data only) ===

Sessions:          ~{estimated} (from TSV gaps)
Iterations:        {total} ({keeps} keeps, {discards} discards, {crashes} crashes)
Keep Rate:         {rate}%
Sync Queue:        {N} pending items | empty | no queue file

Recent Activity (from git log):
  {date} — {commit message}
  {date} — {commit message}

Note: Connect Obsidian for full status (Knowledge Center, Decisions, Predictions, Strategy Analysis).
```

## Flags

| Flag | Purpose |
|------|---------|
| `--project <name>` | Override auto-detected project name |

## Constraints

- **Read-only:** This workflow makes NO modifications to the codebase, git history, or Obsidian vault.
- **No loop:** Single-pass operation. Gathers data, computes stats, prints, and exits.
- **Graceful degradation:** Every Obsidian query has a fallback. The dashboard always prints something, even with zero Obsidian access.
- **No AskUserQuestion:** This is a non-interactive display command. It prints and exits.
