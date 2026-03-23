---
name: autonexus:auto-agent
description: Intelligent auto-scaling agent spawning — automatically determines optimal team size and composition based on task complexity.
argument-hint: "[task description] [--complexity <low|medium|high|enterprise>] [--budget <token-limit>] [--dry-run]"
---

EXECUTE IMMEDIATELY — do not deliberate, do not ask clarifying questions before reading the protocol.

## Argument Parsing (do this FIRST, before reading any files)

Extract from $ARGUMENTS:

- `--complexity <level>` or `Complexity:` — override complexity assessment (low|medium|high|enterprise)
- `--budget <limit>` or `Budget:` — token budget limit for the task
- `--dry-run` — show what agents WOULD be spawned without executing
- `--scope <glob>` or `Scope:` — file scope for the task
- `--prefer <tier>` or `Prefer:` — prefer agents from a specific tier (junior|mid|senior|lead)

Remaining unmatched text = task description.

## Execution

1. Read three-tier routing: `.claude/skills/autonexus/references/three-tier-routing.md`
2. Read agent roster: `.claude/skills/autonexus/references/agents/index.md`
3. Read claims authorization: `.claude/skills/autonexus/references/claims-authorization.md`
4. If no task → use `AskUserQuestion`:
   - Question 1 (Task): "What task should be auto-assigned?" with custom input
   - Question 2 (Scope): "Which files are in scope?" with detected project file options
5. Assess complexity across 5 dimensions (0-10 each):
   - **File breadth:** How many files involved? (1=1 file, 10=entire codebase)
   - **Domain depth:** How much domain knowledge needed? (1=generic, 10=deep specialization)
   - **Novelty:** Has this type of task been done before? (1=routine, 10=first time)
   - **Risk:** Blast radius of a mistake? (1=trivial, 10=production outage)
   - **Reasoning depth:** How many logical steps needed? (1=direct, 10=complex chain)
6. Determine team size based on total score (0-50):
   - **Low (≤ 10):** 2 agents — 1 coordinator (sonnet) + 1 developer (sonnet)
   - **Medium (11-25):** 4 agents — 1 coordinator + 2 developers + 1 reviewer
   - **High (26-40):** 8 agents — 2 coordinators + 3 developers + 2 reviewers + 1 QA
   - **Enterprise (> 40):** Full swarm — queen + 8-15 specialized agents (redirect to /autonexus:swarm)
7. Select specific agents from roster based on task domain:
   - Backend-heavy → sr-backend-eng, backend-eng
   - Frontend-heavy → sr-frontend-eng, frontend-eng
   - Full-stack → sr-fullstack-eng
   - Security-sensitive → sr-security-eng
   - Data-heavy → dba
8. Assign claims based on agent tiers and task scope
9. Display planned team:
   | Agent | Role | Model | Claims | Task Assignment |
10. If `--dry-run` → STOP after displaying plan
11. If `--budget` specified → verify team cost within budget, downgrade tiers if needed
12. Spawn agents and execute task
13. Collect results, revoke claims, report

Stream all output live — never run in background.
