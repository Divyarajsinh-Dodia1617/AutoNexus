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
| jr-qa-eng | Jr. QA Engineer | sonnet | agents/quality/jr-qa-engineer.md |

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

### Swarm Intelligence (Tier 2-7) — Multi-Agent Coordination

| ID | Name | Model | Definition |
|----|------|-------|------------|
| strategic-queen | Strategic Queen | opus | agents/swarm/strategic-queen.md |
| tactical-queen | Tactical Queen | sonnet | agents/swarm/tactical-queen.md |
| adaptive-queen | Adaptive Queen | sonnet | agents/swarm/adaptive-queen.md |
| scout-explorer | Scout Explorer | sonnet | agents/swarm/scout-explorer.md |
| swarm-memory-mgr | Swarm Memory Manager | sonnet | agents/swarm/swarm-memory-manager.md |
| collective-intel | Collective Intelligence Coordinator | sonnet | agents/swarm/collective-intelligence.md |

### Consensus (Tier 6) — Decision Algorithms

| ID | Name | Model | Definition |
|----|------|-------|------------|
| raft-manager | Raft Consensus Manager | sonnet | agents/consensus/raft-manager.md |
| bft-coordinator | Byzantine Fault Tolerance Coordinator | sonnet | agents/consensus/byzantine-coordinator.md |

### SPARC Methodology (Tier 5-6) — Structured Development

| ID | Name | Model | Definition |
|----|------|-------|------------|
| sparc-spec | SPARC Specification Agent | sonnet | agents/sparc/specification-agent.md |
| sparc-pseudo | SPARC Pseudocode Agent | sonnet | agents/sparc/pseudocode-agent.md |
| sparc-arch | SPARC Architecture Agent | opus | agents/sparc/architecture-agent.md |
| sparc-refine | SPARC Refinement Agent | sonnet | agents/sparc/refinement-agent.md |
| sparc-complete | SPARC Completion Agent | sonnet | agents/sparc/completion-agent.md |

### GitHub Operations (Tier 5-8) — Repository Management

| ID | Name | Model | Definition |
|----|------|-------|------------|
| pr-manager | PR Manager | sonnet | agents/github/pr-manager.md |
| issue-tracker | Issue Tracker | sonnet | agents/github/issue-tracker.md |
| release-mgr | Release Manager | sonnet | agents/github/release-manager.md |
| code-review-swarm | Code Review Swarm | sonnet | agents/github/code-review-swarm.md |
| multi-repo-coord | Multi-Repo Coordinator | sonnet | agents/github/multi-repo-coordinator.md |

### Learning & Reasoning (Tier 5-7) — Self-Improvement

| ID | Name | Model | Definition |
|----|------|-------|------------|
| reasoning-bank-mgr | ReasoningBank Manager | sonnet | agents/learning/reasoning-bank-manager.md |
| pattern-distiller | Pattern Distiller | sonnet | agents/learning/pattern-distiller.md |
| meta-learner | Meta-Learning Analyst | opus | agents/learning/meta-learning-agent.md |

### Governance (Tier 3-5) — Policy & Compliance

| ID | Name | Model | Definition |
|----|------|-------|------------|
| governance-ctrl | Governance Controller | sonnet | agents/governance/governance-controller.md |
| claims-mgr | Claims Manager | sonnet | agents/governance/claims-manager.md |
| compliance-auditor | Compliance Auditor | sonnet | agents/governance/compliance-auditor.md |

### Optimization (Tier 6-8) — Performance & Cost

| ID | Name | Model | Definition |
|----|------|-------|------------|
| topology-optimizer | Topology Optimizer | sonnet | agents/optimization/topology-optimizer.md |
| perf-monitor | Performance Monitor | haiku | agents/optimization/performance-monitor.md |
| resource-allocator | Resource Allocator | sonnet | agents/optimization/resource-allocator.md |
| benchmark-runner | Benchmark Runner | haiku | agents/optimization/benchmark-runner.md |

### Security (Enhanced) (Tier 6-7) — Advanced Security

| ID | Name | Model | Definition |
|----|------|-------|------------|
| security-analyst | Security Analyst | sonnet | agents/security/security-analyst.md |
| cve-remediation | CVE Remediation Specialist | sonnet | agents/security/cve-remediation.md |
| ai-defense | AI Defense Specialist | opus | agents/security/ai-defense.md |

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
| Quality | qa-manager, sr-qa-eng, jr-qa-eng |
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
| Additional Quality | jr-qa-eng (×2 — multiple Jr. QA Engineers for parallel story testing) |
| Operations | sr-devops-eng, sr-sre |
| Design | ux-designer, ui-designer |
| Database | dba |
| Documentation | tech-writer |

Sprint capacity: ~80 story points

### Swarm Team (8-15 agents)

Best for: Complex multi-domain tasks requiring parallel exploration

| Role | Agent |
|------|-------|
| Strategic Coordination | strategic-queen |
| Tactical Coordination | tactical-queen |
| Adaptive Optimization | adaptive-queen |
| Reconnaissance | scout-explorer (×2) |
| Knowledge Management | swarm-memory-mgr |
| Consensus | collective-intel, raft-manager |
| Builders | sr-backend-eng, sr-frontend-eng, fullstack-eng |
| Quality | sr-qa-eng, qa-eng |
| Security | sr-security-eng |

Swarm capacity: Task-dependent (queen decomposes into subtasks)

### SPARC Team (7-12 agents)

Best for: Feature development following structured SPARC methodology

| Role | Agent |
|------|-------|
| Specification | sparc-spec, pm |
| Pseudocode | sparc-pseudo |
| Architecture | sparc-arch, architect |
| Refinement | sparc-refine, sr-backend-eng, sr-frontend-eng |
| Completion | sparc-complete, sr-qa-eng, tech-writer |
| Oversight | cto |

SPARC capacity: 1 feature at a time, phased execution

### Governance Team (5 agents)

Best for: Policy enforcement, compliance audits, security governance

| Role | Agent |
|------|-------|
| Control Plane | governance-ctrl |
| Permissions | claims-mgr |
| Compliance | compliance-auditor |
| Security Review | security-architect, ai-defense |

## Escalation Chain

| Tier | Agent | Escalates To |
|------|-------|-------------|
| Jr. Engineer | jr-backend-eng, jr-frontend-eng | sr-backend-eng / sr-frontend-eng |
| Jr. QA Engineer | jr-qa-eng | sr-qa-eng |
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
| Swarm Worker | scout-explorer, builders | tactical-queen |
| Tactical Queen | tactical-queen | strategic-queen |
| Strategic Queen | strategic-queen | cto |
| Adaptive Queen | adaptive-queen | strategic-queen |
| Consensus | raft-manager, bft-coordinator | collective-intel |
| Collective Intel | collective-intel | strategic-queen |
| SPARC Agents | sparc-spec, sparc-pseudo, sparc-refine, sparc-complete | sparc-arch |
| SPARC Architect | sparc-arch | cto |
| GitHub Agents | pr-manager, issue-tracker, release-mgr | em |
| Code Review Swarm | code-review-swarm | principal-eng |
| Multi-Repo | multi-repo-coord | cto |
| Learning | pattern-distiller | reasoning-bank-mgr |
| ReasoningBank | reasoning-bank-mgr | principal-eng |
| Meta-Learner | meta-learner | cto |
| Governance | claims-mgr, compliance-auditor | governance-ctrl |
| Governance Ctrl | governance-ctrl | cto |
| Security (Advanced) | security-analyst, cve-remediation | security-architect |
| AI Defense | ai-defense | security-architect |
| Optimization | perf-monitor, benchmark-runner | resource-allocator |
| Topology Optimizer | topology-optimizer | adaptive-queen |

## Model Assignment

All agents use Sonnet by default. Exceptions:

| Agent | Model | Rationale |
|-------|-------|-----------|
| CEO | opus | Strategic decisions, complex tradeoffs |
| CTO | opus | Technical strategy, architecture review |
| CPO | opus | Product strategy, user experience vision |
| Principal Engineer | opus | Org-wide technical decisions, complex problem solving |
| Strategic Queen | opus | Complex goal decomposition, multi-domain coordination |
| SPARC Architecture Agent | opus | System design, interface definitions |
| Meta-Learning Analyst | opus | Meta-analysis of learning effectiveness |
| AI Defense Specialist | opus | AI safety, prompt injection defense |
| Performance Monitor | haiku | Lightweight background monitoring |
| Benchmark Runner | haiku | Fast metric collection |
