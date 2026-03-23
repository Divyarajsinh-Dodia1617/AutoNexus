# /autonexus:govern — Governance Control Plane

Manages safety guardrails, permissions, and audit trails for agent actions. Define policies that restrict what agents can do — block secret commits, limit file scope, enforce review requirements — and audit what they did. Essential for running agents on production codebases.

## Syntax

```
# Apply a built-in template
/autonexus:govern --template secret-protection

# List all active policies
/autonexus:govern --list

# Add a custom policy
/autonexus:govern --add "no-force-push: block git push --force on main"

# View audit trail
/autonexus:govern --audit

# Check for policy violations
/autonexus:govern --violations

# View agent permission claims
/autonexus:govern --claims

# Disable a policy temporarily
/autonexus:govern --disable secret-protection

# Re-enable a policy
/autonexus:govern --enable secret-protection

# Remove a policy permanently
/autonexus:govern --remove no-force-push
```

## What It Does

1. **Policy management** — Define rules that constrain agent behavior. Policies are checked before every agent action.
2. **Templates** — Pre-built policy bundles for common safety scenarios (secret protection, scope guarding, production safety).
3. **Audit trail** — Every agent action is logged with timestamp, agent name, action type, affected files, and policy evaluation result.
4. **Violation tracking** — When an agent attempts a blocked action, it's logged as a violation with full context for review.
5. **Claims system** — Agents declare what permissions they need; the governance plane grants or denies based on active policies.

## Built-in Templates

| Template | Policies Included | Use Case |
|----------|-------------------|----------|
| `secret-protection` | Block commits containing API keys, tokens, passwords; scan for .env files | Any project with secrets |
| `scope-guard` | Restrict agents to specified directories; block modifications outside scope | Monorepos, shared codebases |
| `production-safety` | Require review before deploy; block direct-to-main commits; enforce test pass | Production environments |
| `minimal` | Log all actions but block nothing | Monitoring without restriction |
| `strict` | All of the above combined | High-security projects |

## Flags

| Flag | Purpose |
|------|---------|
| `--list` | List all active policies with their status |
| `--add <policy>` | Add a custom policy rule |
| `--disable <name>` | Temporarily disable a policy |
| `--enable <name>` | Re-enable a disabled policy |
| `--remove <name>` | Permanently remove a policy |
| `--audit` | Show the full audit trail for recent sessions |
| `--claims` | Show agent permission claims and their grant/deny status |
| `--violations` | Show all policy violations |
| `--template <name>` | Apply a built-in policy template |

## Examples

```bash
# Set up secret protection before running agents
/autonexus:govern --template secret-protection

# Output:
# Applied template: secret-protection
# Active policies:
#   ✓ block-secret-commits    Block commits containing API keys, tokens, passwords
#   ✓ scan-env-files          Warn on .env file modifications
#   ✓ scan-credentials        Block files matching *.pem, *.key, credentials.*

# Set up scope guard for a monorepo
/autonexus:govern --add "scope-limit: restrict agents to packages/auth/** only"

# View the audit trail after a session
/autonexus:govern --audit

# Output:
# ┌──────────────────────────────────────────────────────────────────────┐
# │ Audit Trail — Last 24h                                              │
# ├───────────┬──────────────┬──────────────┬───────────┬───────────────┤
# │ Timestamp │ Agent        │ Action       │ Files     │ Policy Result │
# ├───────────┼──────────────┼──────────────┼───────────┼───────────────┤
# │ 14:32:01  │ Code Agent   │ file:write   │ auth.ts   │ ✓ allowed     │
# │ 14:32:15  │ Code Agent   │ file:write   │ .env      │ ✗ BLOCKED     │
# │ 14:33:22  │ QA Agent     │ file:write   │ auth.test │ ✓ allowed     │
# │ 14:34:10  │ Ship Agent   │ git:commit   │ 3 files   │ ✓ allowed     │
# └───────────┴──────────────┴──────────────┴───────────┴───────────────┘

# Check violations after a sprint
/autonexus:govern --violations

# Output:
# Violations (2 found):
#   1. [14:32:15] Code Agent attempted to write .env — blocked by scan-env-files
#   2. [15:01:44] Ship Agent attempted force-push to main — blocked by no-force-push
```

## Obsidian Integration

### What Gets Written

| What | Obsidian Path | When |
|------|---------------|------|
| Active policies | `Projects/{ProjectName}/Governance/policies.md` | On policy add/remove/template |
| Audit log | `Projects/{ProjectName}/Governance/audit-log/{date}.md` | After every agent action |
| Violations report | `Projects/{ProjectName}/Governance/violations/{date}.md` | On each policy violation |

### What Gets Read
- Active policies before every agent action
- Previous violations for trend analysis in `--violations`

## Without Obsidian

- Policy enforcement works fully without Obsidian — policies are stored in session state
- `--audit` and `--violations` only show current session data (no persistence)
- `--template` and `--add` work normally, but policies reset between sessions
- For persistent governance across sessions, Obsidian is required

## Tips

- **Set up templates before running agents on production code.** `--template secret-protection` takes 5 seconds and prevents costly mistakes.
- **Check `--violations` after sprints** to catch patterns — if agents keep hitting the same policy, the underlying task might need restructuring.
- **Use `scope-guard` in monorepos** to prevent agents from accidentally modifying unrelated packages.
- **`--audit` is your friend for debugging** — if an agent did something unexpected, the audit trail shows exactly what happened and why it was allowed.
- **Combine with `/autonexus:hook`** — set up a hook that auto-runs `--violations` after every session for continuous monitoring.
- **Start with `minimal` template** if you want visibility without restrictions, then tighten policies as you learn what guardrails your project needs.
