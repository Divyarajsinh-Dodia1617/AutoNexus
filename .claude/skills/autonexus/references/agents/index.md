# Agent Roster — /autonexus:agile

Defines all available agents, their categories, and team configurations. The orchestrator reads this file to determine which agents to activate for each story based on team config and story complexity.

## Agent Registry

### Executive (Tier 1) — Strategic Direction

| ID | Name | Model | Definition |
|----|------|-------|------------|
| ceo | CEO | opus | agents/executive/ceo.md |
| cto | CTO | opus | agents/executive/cto.md |
| cpo | CPO | opus | agents/executive/cpo.md |

### Management (Tier 4) — Team Level

| ID | Name | Model | Definition |
|----|------|-------|------------|
| em | Engineering Manager | sonnet | agents/management/em.md |
| pm | Product Manager | sonnet | agents/management/pm.md |
| tpm | Technical Program Manager | sonnet | agents/management/tpm.md |
| scrum-master | Scrum Master | sonnet | agents/management/scrum-master.md |

### Technical Leadership (Tier 5) — Pure Technical Authority

| ID | Name | Model | Definition |
|----|------|-------|------------|
| principal-eng | Principal Engineer | opus | agents/technical-leadership/principal-engineer.md |
| sr-staff-eng | Sr. Staff Engineer | sonnet | agents/technical-leadership/sr-staff-engineer.md |
| staff-eng | Staff Engineer | sonnet | agents/technical-leadership/staff-engineer.md |

### Architecture (Tier 6)

| ID | Name | Model | Definition |
|----|------|-------|------------|
| architect | Solutions Architect | sonnet | agents/architecture/solutions-architect.md |
| security-architect | Security Architect | sonnet | agents/architecture/security-architect.md |

### Senior Engineering (Tier 7) — Builders + Technical Owners

| ID | Name | Model | Definition |
|----|------|-------|------------|
| sr-backend-eng | Sr. Backend Engineer | sonnet | agents/engineering/sr-backend-engineer.md |
| sr-frontend-eng | Sr. Frontend Engineer | sonnet | agents/engineering/sr-frontend-engineer.md |
| sr-fullstack-eng | Sr. Full Stack Engineer | sonnet | agents/engineering/sr-fullstack-engineer.md |
| sr-devops-eng | Sr. DevOps Engineer | sonnet | agents/operations/sr-devops-engineer.md |
| sr-sre | Sr. SRE | sonnet | agents/operations/sr-sre.md |
| sr-security-eng | Sr. Security Engineer | sonnet | agents/operations/security-engineer.md |
| sr-qa-eng | Sr. QA Engineer (SDET) | sonnet | agents/quality/sr-qa-engineer.md |

### Mid-Level Engineering (Tier 8) — Feature Builders

| ID | Name | Model | Definition |
|----|------|-------|------------|
| backend-eng | Backend Engineer | sonnet | agents/engineering/backend-engineer.md |
| frontend-eng | Frontend Engineer | sonnet | agents/engineering/frontend-engineer.md |
| fullstack-eng | Full Stack Engineer | sonnet | agents/engineering/fullstack-engineer.md |
| qa-eng | QA Engineer | sonnet | agents/quality/qa-engineer.md |

### Junior Engineering (Tier 9) — Learning + Executing

| ID | Name | Model | Definition |
|----|------|-------|------------|
| jr-backend-eng | Jr. Backend Engineer | sonnet | agents/engineering/jr-backend-engineer.md |
| jr-frontend-eng | Jr. Frontend Engineer | sonnet | agents/engineering/jr-frontend-engineer.md |

### Quality Management (Tier 7)

| ID | Name | Model | Definition |
|----|------|-------|------------|
| qa-manager | QA Manager | sonnet | agents/quality/qa-manager.md |

### Specialized

| ID | Name | Model | Definition |
|----|------|-------|------------|
| ux-designer | UX Designer | sonnet | agents/specialized/ux-designer.md |
| ui-designer | UI Designer | sonnet | agents/specialized/ui-designer.md |
| tech-writer | Technical Writer | sonnet | agents/specialized/technical-writer.md |
| dba | DBA | sonnet | agents/specialized/dba.md |

## Team Configurations

### Auto (default)

The EM Agent selects team size per story during sprint planning:
- 1-3 point stories → Lean Team
- 5-8 point stories → Standard Team
- 13+ point stories → Full Team

### Lean Team (5 core agents)

Best for: simple stories, bug fixes, small features

| Role | Agent |
|------|-------|
| Product | pm |
| Architecture | architect |
| Implementation | sr-fullstack-eng |
| Quality | qa-eng |
| Coordination | em |

Sprint capacity: ~25 story points

### Standard Team (12 agents)

Best for: typical feature development

| Role | Agent |
|------|-------|
| Product | pm |
| Architecture | architect |
| Strategic Review | cto |
| Coordination | em, scrum-master |
| Backend | sr-backend-eng, backend-eng |
| Frontend | sr-frontend-eng, frontend-eng |
| Quality | qa-manager, sr-qa-eng |
| Security | sr-security-eng |

Sprint capacity: ~50 story points

### Full Team (20+ agents)

Best for: complex features, new systems, major refactors

All Standard Team agents, plus:

| Role | Agent |
|------|-------|
| Executive | ceo, cpo |
| Tech Leadership | principal-eng, sr-staff-eng |
| Additional Engineering | jr-backend-eng, jr-frontend-eng, fullstack-eng |
| Operations | sr-devops-eng, sr-sre |
| Design | ux-designer, ui-designer |
| Database | dba |
| Documentation | tech-writer |

Sprint capacity: ~80 story points

## Escalation Chain

| Tier | Agent | Escalates To |
|------|-------|-------------|
| Jr. Engineer | jr-backend-eng, jr-frontend-eng | sr-backend-eng / sr-frontend-eng |
| Mid Engineer | backend-eng, frontend-eng, qa-eng | sr-backend-eng / sr-frontend-eng / sr-qa-eng |
| Sr. Engineer | sr-backend-eng, sr-frontend-eng | staff-eng |
| Staff Engineer | staff-eng | sr-staff-eng |
| Sr. Staff Engineer | sr-staff-eng | principal-eng |
| Principal Engineer | principal-eng | cto |
| QA | qa-eng, sr-qa-eng | qa-manager |
| QA Manager | qa-manager | em |
| Architect | architect | principal-eng |
| Security | sr-security-eng | security-architect |
| Security Architect | security-architect | cto |
| EM | em | cto |
| PM | pm | cpo |
| CTO vs CPO | cto, cpo | ceo |

## Model Assignment

All agents use Sonnet by default. Exceptions:

| Agent | Model | Rationale |
|-------|-------|-----------|
| CEO | opus | Strategic decisions, complex tradeoffs |
| CTO | opus | Technical strategy, architecture review |
| CPO | opus | Product strategy, user experience vision |
| Principal Engineer | opus | Org-wide technical decisions, complex problem solving |
