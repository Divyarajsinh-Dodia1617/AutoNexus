# Governance Control Plane — AutoNexus

Policy enforcement and compliance framework constraining agent behavior.

## Policy Types

| Policy | Enforcement |
|--------|-------------|
| File Protection | Glob patterns no agent can modify |
| Command Restriction | Commands requiring approval |
| Branch Protection | Restricted git operations |
| Secret Detection | Patterns that must never appear in commits |
| Scope Enforcement | Max files per atomic change |
| Time Limits | Max agent task duration |
| Cost Limits | Token budget per session/sprint |

## Built-in Policies (Always Active)
1. Never modify .env, *.key, *.pem, credentials.*
2. Never git push --force to main/master
3. Never rm -rf outside project
4. Max 10 files per atomic change
5. Agent tasks timeout after 5 minutes

## Governance Events
policy-violation, policy-warning, approval-required, cost-threshold.

## Audit Trail
Projects/{ProjectName}/Governance/ -- policies/, audit-log.md, violations.md, approvals.md

## Integration Points
Evaluated at every operation. Loaded at session start. Managed via /autonexus:govern.
