# Autonomous Loop Protocol — AutoNexus

Detailed protocol for the AutoNexus iteration loop. SKILL.md has the summary; this file has the full rules.

Adapted from autoresearch with Obsidian MCP integration at Phase 0, Phase 1, Phase 7, and session end.

## Loop Modes

- **Unbounded (default):** Loop forever until manually interrupted (`Ctrl+C`)
- **Bounded:** Loop exactly N times when `Iterations: N` is set

When bounded, track `current_iteration` against `max_iterations`. After the final iteration, print a summary and stop.

## Phase 0: Precondition Checks + Obsidian Init

**MUST complete ALL checks before entering the loop. Fail fast if any check fails.**

### Git Checks

```bash
# 1. Verify git repo exists
git rev-parse --git-dir 2>/dev/null || echo "FAIL: not a git repo"

# 2. Check for dirty working tree
git status --porcelain
# → If dirty: warn user and ask to stash or commit first

# 3. Check for stale lock files
ls .git/index.lock 2>/dev/null && echo "WARN: stale lock"

# 4. Check for detached HEAD
git symbolic-ref HEAD 2>/dev/null || echo "WARN: detached HEAD"

# 5. Check for git hooks
ls .git/hooks/pre-commit .git/hooks/commit-msg 2>/dev/null && echo "INFO: git hook detected"
ls .husky/pre-commit .husky/commit-msg 2>/dev/null && echo "INFO: husky hook detected"
```

**If any FAIL:** Stop and inform user.
**If any WARN:** Log the warning, proceed with caution.

### Obsidian Initialization

After git checks pass, initialize Obsidian. Load `references/obsidian-integration.md` and execute:

```
6. Resolve ProjectName (git remote → package.json → basename cwd)
7. Obsidian connectivity check:
   TRY: obsidian_list_notes({ dirPath: "Projects/", recursionDepth: 0 })
   SUCCESS → SET OBSIDIAN_AVAILABLE = true
   FAILURE → SET OBSIDIAN_AVAILABLE = false, WARN once

8. If OBSIDIAN_AVAILABLE:
   a. Drain any pending obsidian-sync-queue.md from previous session
   b. Archive old sessions (>30 days, max 5 folders) per obsidian-integration.md
   c. Generate session-id: {YYYYMMDD}-{HHMM}-{4-char-random}
   d. Create session folder marker in Obsidian
```

## Phase 1: Review (30 seconds)

Before each iteration, build situational awareness. **Complete ALL steps.**

### Standard Review (always)

```
1. Read current state of in-scope files (full context)
2. Read last 10-20 entries from autonexus-results.tsv
3. MUST run: git log --oneline -20 to see recent changes
4. MUST run: git diff HEAD~1 (if last iteration was "keep") to review what worked
5. Identify: what worked, what failed, what's untried
6. If bounded: check current_iteration vs max_iterations
```

### Obsidian Context Reads (if OBSIDIAN_AVAILABLE = true)

```
7. Identify files likely to be modified this iteration
8. Query 1 — Failed approaches:
   obsidian_global_search({
     query: "{target_filename}",
     searchInPath: "Projects/{ProjectName}/Iterations/",
     contextLength: 200,
     pageSize: 10
   })
   → Filter for notes with status: discard or status: crash in frontmatter
   → Extract "Change" and "Rationale" lines
   → Feed into ideation: "Avoid: {description} — previously failed"

9. Query 2 — Predict persona warnings:
   obsidian_global_search({
     query: "{target_filename}",
     searchInPath: "Projects/{ProjectName}/Predictions/",
     contextLength: 300,
     pageSize: 5
   })
   → Extract persona name, severity, recommendation
   → Feed into ideation: "Warning from {persona}: {severity} — {recommendation}"

Both queries use retry-then-queue protocol. If both fail, proceed with git + TSV context only.
```

**Why read git history every time?** Git IS the code memory. After rollbacks, state may differ from what you expect. The git log shows which experiments were kept vs reverted.

**Why read Obsidian every time?** Obsidian IS the knowledge memory. It captures WHY things failed (not just THAT they failed) and warnings from predict personas about the files you're about to touch.

## Git as Memory

Git as Memory is **always enabled**. The agent reads its own git history every iteration.

```bash
# Step 1: Recent experiment history
git log --oneline -20

# Step 2: Last successful change
git diff HEAD~1

# Step 3: Avoid repeating failures
git log --oneline -20 | grep "experiment"

# Step 4: Deep-dive a specific success
git show <hash> --stat
```

### Error Handling for Git Operations

```bash
safe_git_log() {
  git log --oneline -"${1:-20}" 2>/dev/null || echo "WARN: git log failed — empty repo?"
}

safe_git_diff() {
  git diff HEAD~1 --stat 2>/dev/null || echo "INFO: no previous commit to diff"
}

ensure_on_branch() {
  if ! git symbolic-ref HEAD 2>/dev/null; then
    echo "WARN: detached HEAD — creating recovery branch"
    git checkout -b autonexus-recovery-$(date +%s)
  fi
}
```

## Phase 2: Ideate (Strategic)

Pick the NEXT change. **MUST consult git history, results log, AND Obsidian context before deciding.**

**Priority order:**

1. **Fix crashes/failures** from previous iteration first
2. **Exploit successes** — run `git diff` on last kept commit, try variants
3. **Explore new approaches** — cross-reference results log, git history, AND Obsidian failed approaches
4. **Combine near-misses** — two changes that individually didn't help might work together
5. **Simplify** — remove code while maintaining metric
6. **Radical experiments** — when incremental changes stall

**Using Obsidian context in ideation:**
- If failed approach search returned results for target files → avoid those strategies, try fundamentally different angles
- If predict persona warning found → factor severity into risk assessment, prefer conservative changes for HIGH/CRITICAL warnings
- Mention in ideation reasoning: "Avoided X (previously discarded)" or "Heeded Security Analyst warning about Y"

**Anti-patterns:**
- Don't repeat exact same change that was already discarded — CHECK git log AND Obsidian first
- Don't make multiple unrelated changes at once
- Don't chase marginal gains with ugly complexity
- Don't ignore git history or Obsidian context

**Bounded mode:** If <3 iterations remain, prioritize exploiting successes over exploration.

## Phase 3: Modify (One Atomic Change)

- Make ONE focused change to in-scope files
- The change should be explainable in one sentence
- Write the description BEFORE making the change

**The one-sentence test:** If you need "and" to describe it, it's two changes. Split them.

| One Change (OK) | Two Changes (Split) |
|-----------------|---------------------|
| Change port 3000→8080 in Dockerfile + compose + nginx | Change port AND add new service |
| Add Redis in compose + app config + env vars | Add Redis AND refactor auth module |

### Enforcing Atomicity

```bash
FILES_CHANGED=$(git diff --name-only | wc -l)
if [ "$FILES_CHANGED" -gt 5 ]; then
  echo "WARN: ${FILES_CHANGED} files changed — verify single intent"
fi
```

## Phase 4: Commit (Before Verification)

**Commit before running verification.** This enables clean rollback.

```bash
# Stage ONLY in-scope files — NEVER use git add -A
git add <file1> <file2> ...

# Check if there's actually something to commit
git diff --cached --quiet
# Exit 0 (no changes): log as "no-op", skip verification
# Exit 1 (changes exist): proceed with commit

# Commit with descriptive experiment message
git commit -m "experiment(<scope>): <description>"
```

**Nothing to commit:** If no actual diff was produced, log as `status=no-op`, skip verification, proceed to next iteration.

**Hook failure handling:**
1. Read hook error output to understand WHY
2. If fixable (lint, formatting): fix, re-stage, retry — do NOT use `--no-verify`
3. If not fixable in 2 attempts: log as `status=hook-blocked`, revert changes, move on
4. NEVER bypass hooks with `--no-verify`

**Rollback strategy:**
```bash
# Preferred: git revert (preserves history)
git revert HEAD --no-edit

# Fallback: git reset (if revert conflicts)
git revert --abort && git reset --hard HEAD~1
```

## Phase 5: Verify (Mechanical Only)

Run the agreed-upon verification command. Capture output.

**Timeout rule:** If verification exceeds 2x normal time, kill and treat as crash.

**Extract metric:** Parse verification output for the specific metric number.

## Phase 5.1: Noise Handling

For volatile metrics (benchmarks, ML accuracy, Lighthouse):

| Strategy | When to Use |
|----------|-------------|
| Multi-run median (3-5 runs) | Medium noise metrics (benchmark times) |
| Minimum improvement threshold | Noisy metrics where small deltas are unreliable |
| Confirmation run | High-value keep decisions on noisy metrics |
| Environment pinning | ML/statistical workloads |

## Phase 5.5: Guard (Regression Check)

If a guard command was defined, run it after verification.

- **Verify** = "Did the metric improve?" (the goal)
- **Guard** = "Did anything else break?" (the safety net)

**Guard rules:**
- Only run if defined (optional)
- Run AFTER verify — no point checking guard if metric didn't improve
- Pass/fail only (exit code 0 = pass)
- If guard fails: revert and rework (max 2 attempts)
- NEVER modify guard/test files — adapt the implementation instead

**Guard failure recovery:**
1. Revert change (`safe_revert()`)
2. Read guard output → understand WHAT broke
3. Rework optimization to avoid regression
4. Commit reworked version → re-run verify + guard
5. If passes → keep. If fails → one more attempt, then discard.

## Phase 6: Decide (No Ambiguity)

```bash
safe_revert() {
  echo "Reverting: $(git log --oneline -1)"
  if git revert HEAD --no-edit 2>/dev/null; then
    echo "Reverted via git revert (experiment preserved in history)"
    return 0
  fi
  git revert --abort 2>/dev/null
  git reset --hard HEAD~1
  echo "Reverted via reset (experiment removed from history)"
  return 0
}
```

```
IF metric_improved AND (no guard OR guard_passed):
    STATUS = "keep"
ELIF metric_improved AND guard_failed:
    safe_revert()
    FOR attempt IN 1..2:
        Rework implementation (NOT tests)
        Commit → verify → guard
        IF both pass:
            STATUS = "keep (reworked)"
            BREAK
        safe_revert()
    IF still failing:
        STATUS = "discard"
ELIF metric_same_or_worse:
    STATUS = "discard"
    safe_revert()
ELIF crashed:
    IF fixable (max 3 tries):
        Fix → re-commit → re-verify → re-guard
    ELSE:
        STATUS = "crash"
        safe_revert()
```

**Simplicity override:** If metric barely improved (+<0.1%) but change adds significant complexity → discard. If metric unchanged but code is simpler → keep.

## Phase 7: Log Results

### Step 1: TSV Log (ALWAYS)

Append to `autonexus-results.tsv`:

```
iteration  commit   metric   status        description
42         a1b2c3d  0.9821   keep          increase attention heads from 8 to 12
43         -        0.9845   discard       switch optimizer to SGD
44         -        0.0000   crash         double batch size (OOM)
45         -        -        no-op         attempted to modify read-only config
46         -        -        hook-blocked  pre-commit lint rejected formatting
```

**Valid statuses:** `keep`, `keep (reworked)`, `discard`, `crash`, `no-op`, `hook-blocked`

### Step 2: Obsidian Iteration Note (if OBSIDIAN_AVAILABLE = true)

Write a concise 5-8 line note per `obsidian-integration.md`:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{session-id}-iter-{N}.md",
  content: "{rendered iteration note template}",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Uses retry-then-queue protocol. If fails, the TSV still has the data — no information is lost.

### Step 3: Decision Note (selective)

If this iteration represents a significant architectural decision (not every keep — only meaningful choices with alternatives considered), write a Decision note per `obsidian-integration.md`.

## Phase 8: Repeat or Complete

### Unbounded Mode (default)

Go to Phase 1. **NEVER STOP. NEVER ASK "should I continue?"**

### Bounded Mode (with Iterations: N)

```
IF current_iteration < max_iterations:
    Go to Phase 1
ELIF goal_achieved:
    Print: "Goal achieved at iteration {N}!"
    → Session End Protocol
ELSE:
    Print final summary
    → Session End Protocol
```

### Session End Protocol

When the loop terminates (interrupt, bounded limit, or goal achieved):

```
1. Print final summary:
   === AutoNexus Complete (N iterations) ===
   Baseline: {baseline} → Final: {current} ({delta})
   Keeps: X | Discards: Y | Crashes: Z
   Best iteration: #{n} — {description}

2. If OBSIDIAN_AVAILABLE:
   a. Write session summary note (per obsidian-integration.md)
   b. Append daily note mini-summary (per obsidian-integration.md)
   c. Drain obsidian-sync-queue.md

3. If user intends to push:
   Run pre-push knowledge rebuild (per obsidian-integration.md)
```

### When Stuck (>5 consecutive discards)

Applies to both modes:
1. Re-read ALL in-scope files from scratch
2. Re-read the original goal/direction
3. Review entire results log for patterns
4. Query Obsidian for failed approaches across ALL sessions (not just current)
5. Try combining 2-3 previously successful changes
6. Try the OPPOSITE of what hasn't been working
7. Try a radical architectural change

## Crash Recovery

| Failure | Response |
|---------|----------|
| Syntax error | Fix immediately, don't count as iteration |
| Runtime error | Attempt fix (max 3 tries), then move on |
| Resource exhaustion (OOM) | Revert, try smaller variant |
| Infinite loop/hang | Kill after timeout, revert |
| External dependency | Skip, log, try different approach |
| Obsidian MCP failure | Queue operation, continue loop (NEVER crash for Obsidian) |

## Communication

- **DO NOT** ask "should I keep going?" — in unbounded mode, YES. ALWAYS.
- **DO NOT** summarize after each iteration — just log and continue
- **DO** print a brief one-line status every ~5 iterations
- **DO** alert if you discover something surprising
- **DO** print a final summary when loop completes
