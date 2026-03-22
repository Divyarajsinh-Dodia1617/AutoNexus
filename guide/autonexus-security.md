# Guide: /autonexus:security

Autonomous security auditing with STRIDE threat modeling, OWASP Top 10 coverage, 4 red-team adversarial personas, and Obsidian-backed knowledge persistence.

## What It Does

Runs a structured 7-phase security audit against your codebase:

1. **Recon** — Scans codebase and queries Obsidian Knowledge Center for architecture, components, and past security audits
2. **Assets** — Catalogs data stores, authentication, APIs, external services, user input surfaces
3. **Trust Boundaries** — Maps where trust levels change (browser/server, public/authenticated, user/admin)
4. **STRIDE** — Generates a threat model for every asset + boundary combination
5. **Attack Surface** — Maps entry points, data flows, and abuse paths
6. **Loop** — Iteratively tests attack vectors, validates findings with code evidence, writes each finding to Obsidian
7. **Report** — Produces a consolidated report and writes a threat model summary to Obsidian Decisions

Each iteration picks an untested attack vector, traces code paths, constructs proof, classifies by severity/OWASP/STRIDE, and logs the result.

## Syntax

```
/autonexus:security [flags]
```

## Flags

| Flag | Purpose |
|------|---------|
| `--diff` | Only audit files changed since last audit |
| `--fix` | Auto-fix confirmed Critical/High findings after audit |
| `--fail-on <severity>` | Exit non-zero for CI/CD gating (`critical`, `high`, `medium`) |
| `--scope <glob>` | Restrict audit to specific files/directories |
| `--depth <level>` | `shallow` (5 iter), `standard` (15), `deep` (30+) |
| `--iterations N` | Run exactly N iterations then stop |

## Examples

**Full audit, standard depth:**
```
/autonexus:security
```
Prompts for scope, depth, and action mode interactively.

**Quick API-focused audit:**
```
/autonexus:security --scope "src/api/**/*.ts" --iterations 10
Focus: authentication and authorization flows
```

**CI/CD gate — fail on critical findings:**
```
/autonexus:security --diff --fail-on critical --iterations 5
```
Only audits changed files. Exits non-zero if any Critical finding is confirmed.

**Deep audit with auto-fix:**
```
/autonexus:security --fix --depth deep
```
Runs 30+ iterations, then auto-fixes confirmed Critical and High findings.

**Delta audit comparing to previous run:**
```
/autonexus:security --diff
```
Scopes to files changed since last audit. Marks findings as New, Fixed, or Recurring.

## STRIDE Coverage

Every audit aims for 6/6 STRIDE coverage:

| Category | Looks For |
|----------|-----------|
| **S**poofing | Weak auth, missing CSRF, forged JWTs |
| **T**ampering | Missing validation, SQL injection, prototype pollution |
| **R**epudiation | Missing audit logs, unsigned transactions |
| **I**nfo Disclosure | PII in logs, debug endpoints, error message leaks |
| **D**enial of Service | Missing rate limits, regex DoS, resource exhaustion |
| **E**levation of Privilege | IDOR, broken access control, path traversal |

## OWASP Top 10 Coverage

Every audit aims for 10/10 OWASP category coverage:

A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design, A05 Security Misconfiguration, A06 Vulnerable Components, A07 Auth Failures, A08 Data Integrity Failures, A09 Logging Failures, A10 SSRF.

## 4 Red-Team Personas

1. **Security Adversary** — External hacker targeting auth bypass, injection, data exposure
2. **Supply Chain Attacker** — Targeting dependencies, CI/CD pipeline, unsigned artifacts
3. **Insider Threat** — Malicious employee testing privilege escalation, data exfiltration
4. **Infrastructure Attacker** — Targeting deployment configs, container escape, exposed services

## Composite Metric

```
metric = (owasp_tested/10)*50 + (stride_tested/6)*30 + min(findings, 20)
```
Maximum = 100. Higher = more thorough coverage.

## Obsidian Integration

**What gets read:**
- `Projects/{ProjectName}/Knowledge/Architecture.md` — system overview for context
- `Projects/{ProjectName}/Knowledge/Components.md` — component risk areas
- Past security audit decisions — recurring vulnerability patterns

**What gets written:**
- Per-finding notes at `Projects/{ProjectName}/Iterations/{date}/{session}-security-finding-{N}.md` with severity, OWASP category, STRIDE category, code evidence
- Threat model summary at `Projects/{ProjectName}/Decisions/{date}-security-{slug}.md` with top findings, coverage stats, recommendations
- All writes use retry-then-queue protocol — Obsidian failures never block the audit

## Local Output

Creates `security/{YYMMDD}-{HHMM}-{slug}/` with:
- `overview.md` — executive summary
- `threat-model.md` — STRIDE analysis
- `attack-surface-map.md` — entry points and abuse paths
- `findings.md` — all findings ranked by severity
- `owasp-coverage.md` — per-category test results
- `dependency-audit.md` — known CVEs
- `recommendations.md` — prioritized mitigations with code
- `security-audit-results.tsv` — iteration log

## Chain Patterns

**security then fix then ship:**
```
/autonexus:security --fix --iterations 15
/autonexus:ship --type code-pr
```
Audit, auto-fix critical/high, then ship the fixes as a PR.

**predict then security:**
```
/autonexus:predict
Focus: security implications of recent changes
/autonexus:security --diff
```
Get multi-persona predictions about security risks, then validate with a targeted audit.

**security as CI gate:**
```
claude -p "/autonexus:security --diff --fail-on critical --iterations 5"
```
Run in CI pipeline. Blocks merge if critical findings exist.

## Tips

- Start with `--iterations 10` for a quick first pass, then `--depth deep` for thorough coverage.
- Use `--diff` in CI to only audit changed files — faster and focused on new risk.
- The audit never reports findings without code evidence. Every finding has a `file:line` reference.
- Past Obsidian security records help detect recurring vulnerabilities — issues that keep appearing across audits get flagged and elevated.
- Combine `--fix --fail-on critical` for automated remediation with a safety gate.
