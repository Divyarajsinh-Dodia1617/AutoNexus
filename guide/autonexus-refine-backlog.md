# /autonexus:refine-backlog — Backlog Grooming

Interactively create, refine, reprioritize, split, or archive user stories in your Obsidian-backed product backlog. Every story is validated for testability before it can enter a sprint.

## What It Does

- **Add stories:** Describe what you want → PM + Architect agents structure it into proper user stories
- **Refine stories:** Make acceptance criteria more specific and testable
- **Reprioritize:** Reorder backlog by business value and dependencies
- **Split stories:** Break large stories into smaller, independently deliverable pieces
- **Archive:** Remove completed or cancelled stories
- **Review:** See full backlog status at a glance

## Quick Start

```bash
# Interactive mode
/autonexus:refine-backlog

# Quick-add a story
/autonexus:refine-backlog --new "User can reset their password via email"

# Refine a specific story
/autonexus:refine-backlog --story STORY-042

# Scope to an epic
/autonexus:refine-backlog --epic epic-auth

# Review full backlog
/autonexus:refine-backlog --review

# Archive a story
/autonexus:refine-backlog --archive STORY-039
```

## Prerequisites

1. **Obsidian MCP configured** — Run `/autonexus:setup` if not done

## How It Works

You provide the intent, agents do the structuring:

1. **You say:** "I need a password reset feature"
2. **PM Agent** writes proper user story + 3-6 testable acceptance criteria + verify command + test file path
3. **Architect Agent** assesses technical feasibility, dependencies, complexity
4. **You review** and approve or request changes
5. **Story saved** to Obsidian with status "ready" (if all validation passes)

### Testability Validation

Every story must pass these checks before it's marked "ready":

| Check | Required |
|-------|----------|
| At least 3 acceptance criteria | Yes |
| Each AC is testable | Yes |
| verify_command defined | Yes |
| test_file path defined | Yes |
| Architect technical notes | Yes |
| No unresolved dependencies | Yes |

## Story Format

```markdown
# STORY-042: User Preference API

## User Story
As a registered user, I want to save my dashboard preferences
so that my layout persists across sessions.

## Acceptance Criteria
- [ ] AC-1: GET /api/preferences returns saved preferences
- [ ] AC-2: PUT /api/preferences saves new preferences
- [ ] AC-3: New users get default configuration

## Test Contract
verify_command: "npm test -- --grep 'STORY-042'"
test_file: "tests/stories/story-042.test.ts"
coverage_target: 90
```

## Obsidian Structure

```
Projects/{ProjectName}/Backlog/
|- _index.md          <- Priority-ordered story list
|- epics/
|   |- epic-{slug}.md <- Epic definitions
|- stories/
    |- STORY-{id}.md  <- Individual story notes
```

## Flags

| Flag | Purpose |
|------|---------|
| `--new "title"` | Quick-add a story |
| `--story STORY-{id}` | Refine specific story |
| `--epic epic-{slug}` | Scope to an epic |
| `--review` | Review full backlog status |
| `--archive STORY-{id}` | Archive a story |

## Tips

- **Run this before `/autonexus:agile`** to ensure your backlog has enough ready stories
- **Be descriptive** when adding stories — more context means better acceptance criteria
- **Stories >13 points** should be split — they're too large for a single sprint
- **Check your backlog in Obsidian** — it's all human-readable markdown
