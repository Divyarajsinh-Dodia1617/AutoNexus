# /autonexus:learn — Autonomous Codebase Documentation Engine

Scouts your codebase, learns its architecture and patterns, generates comprehensive documentation, then validates and iteratively fixes until docs match reality. Syncs knowledge to Obsidian Knowledge Center for cross-session persistence.

## Syntax

```
/autonexus:learn

/autonexus:learn --mode init --depth deep

/autonexus:learn --mode update --scope src/api/**

/autonexus:learn --mode check

/autonexus:learn --mode summarize --scan
```

## What It Does

8 phases:

1. **Scout** — parallel codebase reconnaissance; checks Obsidian Knowledge Center cache to skip redundant scanning
2. **Analyze** — detect project type, tech stack, existing doc structure, staleness gap
3. **Map** — determine which docs to create/update based on project signals; gap analysis
4. **Generate** — spawn docs-manager agent; write docs locally AND sync to Obsidian Knowledge Center
5. **Validate** — mechanical verification (code refs, internal links, config keys, size limits)
6. **Fix** — re-generate failed docs with validation feedback (up to 3 iterations)
7. **Finalize** — inventory, git diff summary; update Obsidian Knowledge Center commit_hash and updated_at
8. **Log** — record results to `learn-results.tsv`

## 4 Modes

| Mode | When to Use | What It Does |
|------|------------|-------------|
| **init** | No docs exist (or starting fresh) | Full scout, create all docs, write Knowledge Center |
| **update** | Docs exist but code changed | Incremental scout, refresh stale docs, sync Knowledge Center |
| **check** | Want read-only assessment | Inventory + validation health report, then STOP |
| **summarize** | Need quick overview | Create/update `codebase-summary.md` only |

Mode is auto-detected from `docs/` state if not specified.

## Flags

| Flag | Purpose | Default |
|------|---------|---------|
| `--mode <mode>` | Operation: init, update, check, summarize | Auto-detect |
| `--scope <glob>` | Limit learning to specific dirs/files | Everything |
| `--depth <level>` | Doc comprehensiveness: quick, standard, deep | standard |
| `--file <name>` | Target single doc file for update | all docs |
| `--scan` | Force fresh scout (summarize mode) | false |
| `--topics <list>` | Focus summarize on specific topics | all |
| `--no-fix` | Skip validation-fix loop | false |
| `--format <fmt>` | Output format: markdown (default) | markdown |

## Examples

```bash
# First time — learn everything from scratch
/autonexus:learn --mode init

# Quick: just update what changed
/autonexus:learn --mode update

# Deep documentation with API reference and deployment guide
/autonexus:learn --mode init --depth deep

# Update only the system architecture doc
/autonexus:learn --mode update --file system-architecture.md

# Health check — are docs stale?
/autonexus:learn --mode check
```

## Obsidian Integration

### Knowledge Center Sync

Learn is the primary **producer** of the Obsidian Knowledge Center. When learn runs:

1. **Phase 1 (Scout):** Checks if Knowledge Center has recent data (matching commit hash). If so, uses it as a scout cache to avoid redundant scanning. If stale or missing, runs full scout.

2. **Phase 4 (Generate):** After writing `docs/` locally, extracts and writes three Knowledge Center notes to Obsidian:
   - `Projects/{ProjectName}/Knowledge/Architecture.md` — system overview, tech stack, components, data flow
   - `Projects/{ProjectName}/Knowledge/Components.md` — per-component details, dependencies, risk areas
   - `Projects/{ProjectName}/Knowledge/Dependencies.md` — import graph, external deps, data flows

3. **Phase 7 (Finalize):** Updates `commit_hash` and `updated_at` frontmatter on all Knowledge Center notes so future sessions can check freshness.

### The Knowledge Cycle

```
/autonexus:learn → produces docs/ + Obsidian Knowledge Center
                       ↓
Pre-push rebuild → refreshes Knowledge Center at push time
                       ↓
/autonexus:predict → reads Knowledge Center as scout cache
                       ↓
/autonexus:learn (update) → incrementally refreshes everything
```

Learn generates the knowledge, pre-push keeps it current, predict and other commands consume it.

### Resilience

All Obsidian writes use retry-then-queue protocol. Local `docs/` files are always the authoritative output. If Obsidian is unavailable, documentation generation proceeds identically — only the Knowledge Center sync is skipped.

## Chain Patterns

```bash
# Learn codebase, then predict issues
/autonexus:learn --mode init
/autonexus:predict --scope src/**

# Learn changes, then ship docs as PR
/autonexus:learn --mode update
/autonexus:ship --type code-pr

# Health check → update if stale
/autonexus:learn --mode check
# If report says "Stale" →
/autonexus:learn --mode update

# Full quality pipeline
/autonexus:learn --mode init
/autonexus:scenario --domain software
/autonexus:predict --depth deep
/autonexus:ship
```

## Docs Created

### Always Created (init mode)
- `docs/project-overview-pdr.md` — project overview and PDR
- `docs/codebase-summary.md` — summary with file inventory and key dependencies
- `docs/code-standards.md` — structure and code standards
- `docs/system-architecture.md` — architecture with Mermaid diagrams
- `README.md` — root readme (max 300 lines)

### Conditionally Created
- `docs/deployment-guide.md` — if Dockerfile/CI config detected
- `docs/design-guidelines.md` — if UI components/frontend framework detected
- `docs/api-reference.md` — if API routes/controllers detected
- `docs/testing-guide.md` — if test dirs/config detected
- `docs/configuration-guide.md` — if .env.example/config/ detected
- `docs/changelog.md` — from git log with conventional commit parsing

## Tips

- **Start with `--mode check`** to see current doc health before deciding whether to init or update.
- **Use `--scope`** on large monorepos to avoid context overflow — learn will suggest this if it detects >10K files.
- **Use `--no-fix`** for a fast first pass when you want to review docs before iterative fixing.
- **The validation-fix loop** caps at 3 iterations. If docs still have warnings after 3 passes, review the warnings manually — they're usually edge cases in code references.
- **After `learn --mode init`**, run `learn --mode check` to verify the baseline. Then use `learn --mode update` going forward.
- **Check Obsidian Knowledge Center** after a learn run — the Architecture, Components, and Dependencies notes provide a cross-session persistent view of your codebase that other AutoNexus commands consume.
- **Pair with predict** — running `learn` before `predict` ensures predict has fresh Knowledge Center data, avoiding redundant codebase scanning.
