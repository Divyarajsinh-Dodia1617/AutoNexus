# Claims-Based Authorization — AutoNexus

Glob-scoped permission system. Every agent operates under least privilege with auto-revoking claims.

## Claim Types (7)

| Claim | Description | Example |
|-------|-------------|---------|
| read | File patterns agent can read | src/**/*.ts |
| write | File patterns agent can modify | src/auth/** |
| execute | Shell commands allowed | npm test |
| spawn | Agent types this agent can spawn | jr-*, qa-* |
| memory | Obsidian paths for read/write | Projects/{project}/Iterations/** |
| network | External access | none, npm-registry |
| admin | System-level ops | git-force, deploy, none |

## Security Levels (4)

| Level | Agents | Claims |
|-------|--------|--------|
| Minimal | Junior Tier 9 | Read in-scope, no write, verify only |
| Standard | Mid/Senior Tier 7-8 | Project read, in-scope write, standard cmds |
| Elevated | Leads/Architects Tier 5-6 | All read/write, all cmds, any spawn |
| Admin | Executive Tier 1-3 | Unrestricted, time-limited 30 min |

## Auto-Revocation
Claims granted at task assignment. Revoked on completion. Admin expires after 30 minutes.

## Violation Handling
Block action, log to claims-log.md, fire claim-violation event. 3+ violations: escalate.

## Integration Points
Applied at agent spawn. Evaluated at every operation. Logged to Security/claims-log.md.
