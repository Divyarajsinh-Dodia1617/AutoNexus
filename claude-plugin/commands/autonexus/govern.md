---
name: autonexus:govern
description: Governance control plane — manage policies, view audit trail, handle approvals, and configure agent permissions.
argument-hint: "[--list] [--add] [--disable <policy>] [--enable <policy>] [--audit] [--claims <agent-id>] [--violations] [--template <name>]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--list` — list all active policies with status
- `--add` — interactive policy creation wizard
- `--disable <name>` or `Disable:` — disable a policy (keeps definition, stops enforcement)
- `--enable <name>` or `Enable:` — re-enable a disabled policy
- `--remove <name>` or `Remove:` — permanently delete a policy
- `--audit` — show compliance audit trail (recent evaluations, violations, approvals)
- `--claims <agent-id>` or `Claims:` — show current claims for a specific agent
- `--violations` — show recent policy violations with resolution status
- `--template <name>` or `Template:` — create policy from built-in template
- `--report` — generate full governance compliance report

Remaining unmatched text = policy name or search query.

## Execution

1. Read governance control plane: `.claude/skills/autonexus/references/governance-control.md`
2. Read claims authorization: `.claude/skills/autonexus/references/claims-authorization.md`
3. Read Obsidian integration: `.claude/skills/autonexus/references/obsidian-integration.md`
4. Execute Obsidian connectivity check (SET OBSIDIAN_AVAILABLE)
5. Route based on flags:
   - `--list` → Query `Projects/{ProjectName}/Governance/policies/`, display table:
     | Name | Type | Severity | Status | Last Evaluated |
   - `--add` → use `AskUserQuestion`:
     - Q1 (Type): "What type of policy?" with [File Protection, Command Restriction, Scope Enforcement, Time Limit, Cost Limit, Branch Protection, Secret Detection]
     - Q2 (Severity): "What enforcement level?" with [Block (hard stop), Warn (log + continue), Log (record only)]
     - Q3 (Pattern): "What pattern/condition triggers this policy?" with examples for selected type
     Then write policy note to Obsidian
   - `--disable/--enable/--remove` → Find policy, update status, log action
   - `--audit` → Read `Projects/{ProjectName}/Governance/audit-log.md`, display recent entries
   - `--claims` → Look up agent in roster, compute claims based on tier + active tasks, display:
     | Claim Type | Scope | Source | Expires |
   - `--violations` → Read `Projects/{ProjectName}/Governance/violations.md`, display with status
   - `--template` → Built-in templates:
     - `secret-protection` — Block commits containing secrets (.env, *.key, *.pem)
     - `branch-protection` — Block force-push to main/master
     - `scope-guard` — Max 10 files per atomic change
     - `cost-limit` — Token budget per session
     - `time-limit` — Agent task timeout (5 min default)
   - `--report` → Spawn compliance-auditor agent for full report
   - No flags → Display governance overview: active policies, recent violations, pending approvals
6. Print formatted results

Stream all output live — never run in background.
