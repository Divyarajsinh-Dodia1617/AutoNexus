# Security Workflow — /autonexus:security

Autonomous security auditing that uses the autonexus loop to iteratively discover, validate, and report vulnerabilities. Combines STRIDE threat modeling, OWASP Top 10 sweeps, and red-team adversarial analysis into a single autonomous loop — with Obsidian as a persistent knowledge backbone.

**Output:** A severity-ranked security report with threat model, findings, mitigations, iteration log, and Obsidian knowledge trail.

## Trigger

- User invokes `/autonexus:security`
- User says "security audit", "run a security sweep", "threat model this codebase", "find vulnerabilities"
- User says "red-team this app", "OWASP audit", "STRIDE analysis"

## Loop Support

Works with both unbounded and bounded modes:

```
# Unlimited — keep finding vulnerabilities until interrupted
/autonexus:security

# Bounded — run exactly N security sweep iterations
/autonexus:security
Iterations: 10

# With target scope
/autonexus:security
Scope: src/api/**/*.ts, src/middleware/**/*.ts
Focus: authentication and authorization flows
```

## PREREQUISITE: Interactive Setup (when invoked without flags)

**CRITICAL — BLOCKING PREREQUISITE:** If `/autonexus:security` is invoked without `--diff`, scope, or focus, you MUST scan the codebase first, then use `AskUserQuestion` to gather user input BEFORE proceeding to ANY phase. DO NOT skip this step.

**Single batched call — all 3 questions at once:**

You MUST call `AskUserQuestion` with all 3 questions in ONE call:

| # | Header | Question | Options (from codebase scan) |
|---|--------|----------|------------------------------|
| 1 | `Scope` | "What should I audit?" | "Entire codebase (comprehensive)", "API routes + middleware only", "Authentication + authorization", "External-facing code only" |
| 2 | `Depth` | "How thorough?" | "Quick scan (5 iterations)", "Standard audit (15 iterations)", "Deep audit (30+ iterations)", "Unlimited" |
| 3 | `Action` | "What should I do with confirmed vulnerabilities?" | "Report only (read-only)", "Report + auto-fix Critical/High", "Report + CI gate (fail on critical)" |

**IMPORTANT:** Always ask all questions in a single call — never one at a time.

If flags are provided inline, skip interactive setup and proceed directly.

## Architecture

```
/autonexus:security
  ├── Phase 1: Recon (scan codebase + query Obsidian Knowledge Center)
  ├── Phase 2: Assets (identify data stores, auth, APIs, services)
  ├── Phase 3: Trust Boundaries (map trust level transitions)
  ├── Phase 4: STRIDE (generate threat model per asset × boundary)
  ├── Phase 5: Attack Surface (entry points, data flows, abuse paths)
  ├── Phase 6: Loop (iterative vulnerability discovery + Obsidian write per finding)
  └── Phase 7: Report (consolidated report + Obsidian threat model summary)
```

---

## Phase 1: Recon — Codebase Reconnaissance + Obsidian Query

Scan the project to build context. Before scanning files, query Obsidian for existing knowledge.

### Step 1a: Query Obsidian Knowledge Center

If `OBSIDIAN_AVAILABLE = true` (set at Phase 0 per obsidian-integration.md):

```
TRY (retry-then-queue):
  # Read architecture overview
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Knowledge/Architecture.md",
    format: "markdown"
  })

  # Read component details
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Knowledge/Components.md",
    format: "markdown"
  })

  # Read dependency graph
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Knowledge/Dependencies.md",
    format: "markdown"
  })

IF Knowledge Center notes exist:
  - Use architecture overview to understand system boundaries
  - Use component details to identify risk areas and fragile points
  - Use dependency info to seed the dependency audit
  - Skip redundant file scanning for areas already documented
  LOG: "Loaded architecture context from Obsidian Knowledge Center"

IF Knowledge Center notes do NOT exist:
  - Proceed with full file-based reconnaissance (Step 1b)
  LOG: "No Obsidian knowledge found — running full codebase scan"
```

### Step 1b: Query Obsidian for Past Security Audits

```
TRY (retry-then-queue):
  obsidian_global_search({
    query: "security audit",
    searchInPath: "Projects/{ProjectName}/Decisions/",
    contextLength: 300,
    pageSize: 10
  })

IF past audit decisions found:
  - Extract recurring vulnerability patterns
  - Note previously identified risk areas
  - Flag any findings that were marked "Recurring" in past audits
  - Feed into threat model: "Previously found: {vulnerability} in {location}"
  LOG: "Found {N} past security audit records — checking for recurring patterns"
```

### Step 1c: File-Based Reconnaissance

Scan the project to build context (supplementing or replacing Obsidian data):

```
READ:
  - package.json / requirements.txt / go.mod (dependencies)
  - .env.example / config files (secrets handling)
  - Dockerfile / docker-compose.yml (infrastructure)
  - API route files (attack surface)
  - Auth/middleware files (trust boundaries)
  - Database schemas/models (data assets)
  - CI/CD configs (supply chain)
```

## Phase 2: Assets — Asset Identification

Catalog every asset that has security relevance:

| Asset Type | Examples | Priority |
|------------|----------|----------|
| **Data stores** | Database, Redis, file storage, cookies, localStorage | Critical |
| **Authentication** | Login, OAuth, JWT, sessions, API keys | Critical |
| **API endpoints** | REST routes, GraphQL resolvers, webhooks | High |
| **External services** | Payment APIs, email providers, CDN, analytics | High |
| **User input surfaces** | Forms, URL params, headers, file uploads | High |
| **Configuration** | Environment variables, feature flags, CORS settings | Medium |
| **Static assets** | Public files, uploaded content, generated files | Low |

## Phase 3: Trust Boundaries — Trust Boundary Mapping

Identify where trust levels change:

```
Trust Boundaries:
  ├── Browser ←→ Server (client-side vs server-side)
  ├── Server ←→ Database (application vs data layer)
  ├── Server ←→ External APIs (internal vs third-party)
  ├── Public routes ←→ Authenticated routes
  ├── User role ←→ Admin role (privilege levels)
  ├── CI/CD ←→ Production (deployment boundary)
  └── Container ←→ Host (infrastructure boundary)
```

## Phase 4: STRIDE — Threat Model Generation

For each asset + trust boundary combination, analyze threats using STRIDE:

| Threat | Question | Example Findings |
|--------|----------|------------------|
| **S**poofing | Can an attacker impersonate a user/service? | Weak auth, missing CSRF, forged JWTs |
| **T**ampering | Can data be modified in transit or at rest? | Missing input validation, SQL injection, prototype pollution |
| **R**epudiation | Can actions be denied without evidence? | Missing audit logs, unsigned transactions |
| **I**nformation Disclosure | Can sensitive data leak? | Error messages expose internals, PII in logs, debug endpoints |
| **D**enial of Service | Can the service be disrupted? | Missing rate limiting, regex DoS, resource exhaustion |
| **E**levation of Privilege | Can a user gain unauthorized access? | IDOR, broken access control, path traversal |

Output the threat model as a structured table in the security report.

## Phase 5: Attack Surface — Attack Surface Map

Generate an attack surface map showing:

```
Attack Surface:
  ├── Entry Points
  │   ├── GET /api/users/:id          → IDOR risk (user enumeration)
  │   ├── POST /api/auth/login        → Brute force, credential stuffing
  │   ├── POST /api/upload            → File upload, path traversal
  │   ├── WebSocket /ws               → Auth bypass, injection
  │   └── Webhook /api/webhooks/*     → Signature verification
  ├── Data Flows
  │   ├── User input → DB query       → Injection risk
  │   ├── JWT → route handler          → Token validation
  │   └── File upload → storage        → Malicious file execution
  └── Abuse Paths
      ├── Rate limit bypass → account takeover
      ├── IDOR chain → data exfiltration
      └── SSRF → internal service access
```

## Phase 6: Loop — Iterative Vulnerability Discovery

### Iteration Protocol

Each iteration follows the autonexus pattern adapted for security:

#### Step 1: Review (Select Attack Vector)

Priority order for selecting the next vector to test:

1. **Critical STRIDE threats** not yet tested
2. **OWASP Top 10 categories** not yet covered
3. **High-severity attack paths** from the surface map
4. **Dependency vulnerabilities** (supply chain)
5. **Configuration weaknesses** (headers, CORS, CSP)
6. **Business logic flaws** (race conditions, state manipulation)
7. **Information disclosure** (error handling, debug modes)

Track coverage in the results log. The goal is comprehensive coverage.

#### Step 2: Analyze (Deep Dive)

For the selected vector:
1. Read all relevant code files
2. Trace data flow from entry point to data store
3. Identify missing validation, sanitization, or access checks
4. Look for known vulnerability patterns

#### Step 3: Validate (Proof Construction)

For each potential finding, construct proof:

```
Finding Proof Structure:
  ├── Vulnerable code location (file:line)
  ├── Attack scenario (step-by-step)
  ├── Input that triggers the vulnerability
  ├── Expected vs actual behavior
  ├── Impact assessment
  └── Confidence level (Confirmed / Likely / Possible)
```

**Validation Rules:**
- **Confirmed** — Code path clearly allows the attack, no guards present
- **Likely** — Guards exist but are bypassable or incomplete
- **Possible** — Theoretical risk, depends on configuration or runtime conditions

Do NOT report findings without supporting code evidence.

#### Step 4: Classify

Assign severity and categories:

**Severity (CVSS-inspired):**

| Severity | Criteria |
|----------|----------|
| **Critical** | RCE, auth bypass, SQL injection, data breach, admin takeover |
| **High** | XSS (stored), SSRF, privilege escalation, mass data exposure |
| **Medium** | CSRF, open redirect, info disclosure, missing rate limits |
| **Low** | Missing headers, verbose errors, weak session config |
| **Info** | Best practice suggestions, hardening recommendations |

**OWASP Top 10 (2021) mapping:**

| ID | Category |
|----|----------|
| A01 | Broken Access Control |
| A02 | Cryptographic Failures |
| A03 | Injection |
| A04 | Insecure Design |
| A05 | Security Misconfiguration |
| A06 | Vulnerable Components |
| A07 | Auth & Identification Failures |
| A08 | Software & Data Integrity Failures |
| A09 | Security Logging & Monitoring Failures |
| A10 | Server-Side Request Forgery |

**STRIDE mapping:** Tag each finding with the applicable STRIDE category.

#### Step 5: Log to TSV + Write Finding to Obsidian

Append to security-audit-results.tsv:

```tsv
iteration	vector	severity	owasp	stride	confidence	location	description
0	-	-	-	-	-	-	baseline — 3 npm audit warnings
1	IDOR	High	A01	EoP	Confirmed	src/api/users.ts:42	GET /api/users/:id returns any user data without ownership check
2	XSS	Medium	A03	Tampering	Likely	src/components/comment.tsx:18	User input rendered via dangerouslySetInnerHTML
3	rate-limit	Medium	A05	DoS	Confirmed	src/api/auth.ts:15	POST /login has no rate limiting — brute force possible
```

**Obsidian Write (per finding):**

If `OBSIDIAN_AVAILABLE = true`, write each confirmed/likely finding to Obsidian using retry-then-queue:

```
obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "Projects/{ProjectName}/Iterations/{YYYY-MM-DD}/{session-id}-security-finding-{N}.md",
  content: "---
tags: [autonexus/security, autonexus/finding, autonexus/{project}]
date: {YYYY-MM-DD}
iteration: {N}
severity: {Critical|High|Medium|Low|Info}
owasp: {A01-A10}
stride: {S|T|R|I|D|E}
confidence: {Confirmed|Likely|Possible}
location: {file:line}
session: {session-id}
---

# Finding {N}: {title}

**Severity:** {severity} | **OWASP:** {category} | **STRIDE:** {category}
**Location:** `{file}:{line}`
**Confidence:** {confidence}

## Description
{what is wrong}

## Code Evidence
```{lang}
{vulnerable code snippet}
```

## Attack Scenario
{step-by-step exploitation}

## Mitigation
```{lang}
{fixed code snippet}
```
",
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

This creates a searchable, tagged knowledge trail of every security finding across all audits.

#### Step 6: Repeat

- **Unbounded:** Keep finding vulnerabilities. Never stop. Never ask.
- **Bounded (Iterations: N):** After N iterations, generate final report and stop.
- **Coverage tracking:** Every 5 iterations, print coverage summary.

### Coverage Summary Format

```
=== Security Audit Progress (iteration 10) ===
STRIDE Coverage: S[✓] T[✓] R[✗] I[✓] D[✓] E[✓] — 5/6
OWASP Coverage: A01[✓] A02[✗] A03[✓] A04[✗] A05[✓] A06[✓] A07[✓] A08[✗] A09[✗] A10[✗] — 5/10
Findings: 4 Critical, 2 High, 3 Medium, 1 Low
Confirmed: 7 | Likely: 2 | Possible: 1
```

---

## Phase 7: Report — Final Report + Obsidian Summary

Generated at loop completion (bounded) or on interrupt (unbounded).

### Local Report Structure

```markdown
# Security Audit Report

## Executive Summary
- **Date:** {date}
- **Scope:** {files/directories scanned}
- **Iterations:** {N}
- **Total Findings:** {count} ({critical} Critical, {high} High, {medium} Medium, {low} Low)

## Threat Model

### Assets
{table of identified assets}

### Trust Boundaries
{diagram of trust boundaries}

### STRIDE Analysis
{threat model table}

### Attack Surface Map
{entry points, data flows, abuse paths}

## Findings (Descending Severity)

### [CRITICAL] Finding 1: {title}
- **OWASP:** {category}
- **STRIDE:** {category}
- **Location:** `{file}:{line}`
- **Confidence:** Confirmed / Likely / Possible
- **Description:** {what's wrong}
- **Attack Scenario:** {step-by-step exploitation}
- **Code Evidence:**
  ```{lang}
  {vulnerable code snippet}
  ```
- **Mitigation:**
  ```{lang}
  {fixed code snippet}
  ```
- **References:** {CWE, CVE if applicable}

### [HIGH] Finding 2: ...
...

## Coverage Matrix

| OWASP Category | Tested | Findings |
|----------------|--------|----------|
| A01 Broken Access Control | ✓ | 2 |
| A02 Cryptographic Failures | ✓ | 0 |
| ... | ... | ... |

| STRIDE Category | Tested | Findings |
|-----------------|--------|----------|
| Spoofing | ✓ | 1 |
| Tampering | ✓ | 2 |
| ... | ... | ... |

## Dependency Audit
{npm audit / pip audit / go vulnerabilities}

## Security Headers Check
{CSP, HSTS, X-Frame-Options, etc.}

## Recommendations (Priority Order)
1. {Critical fix 1}
2. {Critical fix 2}
...

## Iteration Log
{full TSV content}
```

### Obsidian: Write Threat Model Summary to Decisions

After generating the local report, write a consolidated security decision record to Obsidian:

```
TRY (retry-then-queue):
  obsidian_update_note({
    targetType: "filePath",
    targetIdentifier: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-security-{slug}.md",
    content: "---
tags: [autonexus/decision, autonexus/security, autonexus/{project}]
date: {YYYY-MM-DD}
status: accepted
type: security-audit
iterations: {N}
findings_critical: {count}
findings_high: {count}
findings_medium: {count}
findings_low: {count}
stride_coverage: {n}/6
owasp_coverage: {n}/10
session: {session-id}
commit_hash: {git rev-parse HEAD}
---

# Security Audit: {slug}

## Summary
- **Scope:** {scope description}
- **Iterations:** {N}
- **Total Findings:** {count} ({critical} Critical, {high} High, {medium} Medium, {low} Low)
- **STRIDE Coverage:** {n}/6 | **OWASP Coverage:** {n}/10

## Top Findings

### 1. [{severity}] {title}
- **OWASP:** {category} | **STRIDE:** {category}
- **Location:** `{file}:{line}`
- **Evidence:** {one-line code evidence}
- **Status:** Open

### 2. [{severity}] {title}
...

{Include top 5-10 findings by severity}

## Threat Model Summary
{Condensed threat model: key assets, critical trust boundaries, highest-risk STRIDE categories}

## Recurring Vulnerabilities
{List any findings that match patterns from past Obsidian security records}

## Recommendations
1. {Priority 1 action}
2. {Priority 2 action}
3. {Priority 3 action}

## Links
- Local report: `security/{folder-name}/overview.md`
- Iteration log: `security/{folder-name}/security-audit-results.tsv`
",
    modificationType: "wholeFile",
    wholeFileMode: "overwrite",
    createIfNeeded: true
  })
```

### Obsidian: Tag and Cross-Link

After writing the decision note:

```
obsidian_manage_tags({
  filePath: "Projects/{ProjectName}/Decisions/{YYYY-MM-DD}-security-{slug}.md",
  operation: "add",
  tags: ["autonexus/security-audit"]
})
```

---

## OWASP Checks Reference

Detailed checks to perform for each OWASP category:

### A01 — Broken Access Control
- [ ] IDOR on all parameterized routes (`:id`, `:slug`)
- [ ] Missing authorization middleware on protected routes
- [ ] Horizontal privilege escalation (user A accessing user B's data)
- [ ] Vertical privilege escalation (user accessing admin functions)
- [ ] Directory traversal on file operations
- [ ] CORS misconfiguration allowing unauthorized origins
- [ ] Missing function-level access control

### A02 — Cryptographic Failures
- [ ] Sensitive data in plaintext (passwords, tokens, PII)
- [ ] Weak hashing algorithms (MD5, SHA1 for passwords)
- [ ] Hardcoded secrets/API keys in source
- [ ] Missing encryption at rest / in transit
- [ ] Weak random number generation for security tokens
- [ ] Exposed .env files or config with secrets

### A03 — Injection
- [ ] SQL/NoSQL injection in database queries
- [ ] Command injection in shell executions (exec, spawn)
- [ ] XSS (stored, reflected, DOM-based)
- [ ] Template injection (SSTI)
- [ ] LDAP injection
- [ ] Path injection in file operations
- [ ] Header injection (CRLF)

### A04 — Insecure Design
- [ ] Missing rate limiting on sensitive endpoints
- [ ] No account lockout after failed login attempts
- [ ] Predictable resource identifiers
- [ ] Race conditions in critical operations
- [ ] Missing CSRF protection on state-changing operations
- [ ] Insecure direct object references in design

### A05 — Security Misconfiguration
- [ ] Debug mode enabled in production
- [ ] Default credentials / admin pages exposed
- [ ] Verbose error messages exposing internals
- [ ] Missing security headers (CSP, HSTS, X-Content-Type-Options)
- [ ] Unnecessary HTTP methods enabled
- [ ] Directory listing enabled
- [ ] Stack traces in error responses

### A06 — Vulnerable and Outdated Components
- [ ] Known CVEs in dependencies (npm audit, pip audit)
- [ ] Outdated frameworks with security patches available
- [ ] Unmaintained dependencies
- [ ] Dependencies with known prototype pollution

### A07 — Identification and Authentication Failures
- [ ] Weak password policies
- [ ] Missing multi-factor authentication for admin
- [ ] Session fixation vulnerabilities
- [ ] JWT vulnerabilities (none algorithm, weak secret, no expiry)
- [ ] Insecure password reset flows
- [ ] Missing session invalidation on logout/password change

### A08 — Software and Data Integrity Failures
- [ ] Missing integrity checks on CI/CD pipelines
- [ ] Unsigned or unverified updates/dependencies
- [ ] Insecure deserialization
- [ ] Missing CSP or SRI for external scripts
- [ ] Unsigned webhooks / API callbacks

### A09 — Security Logging and Monitoring Failures
- [ ] Missing audit logs for security events
- [ ] No logging of failed authentication attempts
- [ ] Sensitive data in logs (passwords, tokens)
- [ ] Missing alerting on suspicious activity
- [ ] Log injection vulnerabilities

### A10 — Server-Side Request Forgery (SSRF)
- [ ] Unvalidated URLs in server-side requests
- [ ] DNS rebinding vulnerabilities
- [ ] Missing allowlist for external service calls
- [ ] Proxy/redirect endpoints without validation

---

## Red-Team Adversarial Lenses

Four adversarial personas for comprehensive coverage:

### 1. Security Adversary (Primary)
**Mindset:** "I'm a hacker trying to breach this system"
- Focus: auth bypass, injection, data exposure, privilege escalation
- Method: trace every input to its sink, find missing guards
- Priority: exploitable findings over theoretical risks

### 2. Supply Chain Attacker
**Mindset:** "I'm compromising dependencies or build pipeline"
- Focus: dependency vulnerabilities, CI/CD weaknesses, unsigned artifacts
- Method: audit dependency tree, check for typosquatting, verify integrity
- Priority: dependencies with known CVEs, build pipeline access

### 3. Insider Threat
**Mindset:** "I'm a malicious employee or compromised account"
- Focus: privilege escalation, data exfiltration, access control gaps
- Method: check what a low-privilege user can access, find horizontal movement
- Priority: admin bypass, bulk data export, missing audit trails

### 4. Infrastructure Attacker
**Mindset:** "I'm attacking the deployment, not the code"
- Focus: container escape, exposed services, network segmentation
- Method: check Docker configs, K8s manifests, exposed ports, env vars
- Priority: secrets in environment, overly permissive configs

---

## Composite Metric

The security audit uses a **coverage + finding count** composite metric:

```
metric = (owasp_categories_tested / 10) * 50 + (stride_categories_tested / 6) * 30 + min(finding_count, 20)
```

- **Direction:** higher is better (more coverage + more findings = more thorough)
- **Maximum theoretical:** 50 + 30 + 20 = 100
- **Baseline:** 0 (nothing tested yet)

This incentivizes the loop to cover ALL categories before going deep on any one.

---

## Flags & Modes

### `--diff` — Delta Mode

Only audit files changed since the last audit. Reads the most recent `security/` subfolder to establish what was already tested.

```
/autonexus:security --diff
```

**How it works:**

1. Find the latest `security/*/overview.md` by timestamp in folder name
2. Parse `findings.md` from that folder to get previously tested files
3. Run `git diff --name-only {last_audit_commit}..HEAD` to find changed files
4. Scope the current audit to ONLY those changed files
5. In the final report, mark findings as:
   - **New** — found in changed files, not in previous audit
   - **Fixed** — was in previous audit, no longer present in changed code
   - **Recurring** — still present from previous audit (unchanged)

**Delta report additions:**

```markdown
## Delta Summary (vs {previous_audit_folder})

| Status | Count | Details |
|--------|-------|---------|
| New findings | 3 | Found in changed files |
| Fixed | 2 | No longer present |
| Recurring | 5 | Still present from last audit |
| Files changed | 12 | Since last audit |
| Files audited | 8 | (security-relevant subset) |
```

If no previous audit folder exists, `--diff` falls back to full audit with a warning.

### `--fail-on` — Severity Threshold Gate

Exit with non-zero code if findings meet or exceed a severity threshold. Designed for CI/CD blocking.

```
/autonexus:security --fail-on critical
/autonexus:security --fail-on high
/autonexus:security --fail-on medium
```

| Flag Value | Blocks on |
|------------|-----------|
| `critical` | Any Critical finding |
| `high` | Any Critical or High finding |
| `medium` | Any Critical, High, or Medium finding |

**Behavior:**
- Runs the full audit normally
- After generating the report, checks findings against threshold
- If threshold met: prints `SECURITY GATE FAILED: {N} findings at {severity} or above` and exits non-zero
- If threshold not met: prints `SECURITY GATE PASSED` and exits 0

### `--fix` — Auto-Remediation Mode

After completing the audit, switches to autonexus modify-then-verify loop to fix confirmed findings.

```
/autonexus:security --fix
/autonexus:security --fix
Iterations: 10
```

**How it works:**

1. Run the full security audit (setup + loop + report)
2. Filter findings: only **Confirmed** severity **Critical** and **High**
3. For each fix iteration:
   - Pick the highest-severity unfixed finding
   - Apply the mitigation from `recommendations.md`
   - Commit the fix
   - Re-verify: does the vulnerability still exist?
   - If fixed: keep commit, mark finding as "Fixed" in report
   - If still vulnerable: revert, try different approach
   - If new findings introduced: revert immediately

**Safety rules:**
- NEVER fix Low or Info findings automatically (too subjective)
- NEVER modify test files (fixes must not break existing tests)
- Run existing tests after each fix — revert if any test fails
- Maximum 3 fix attempts per finding, then skip

### Combining Flags

Flags can be combined:

```
# Delta audit + auto-fix critical/high + block on remaining criticals
/autonexus:security --diff --fix --fail-on critical
Iterations: 15

# Quick delta check in CI
/autonexus:security --diff --fail-on high
Iterations: 5
```

**Execution order when combined:**
1. `--diff` narrows scope
2. Security audit runs (with narrowed scope if `--diff`)
3. `--fix` runs remediation loop on confirmed Critical/High
4. `--fail-on` checks remaining (unfixed) findings against threshold

---

## Obsidian: Recurring Vulnerability Detection

At Phase 1 (Recon), after querying past security audit decisions, perform pattern matching:

```
TRY (retry-then-queue):
  obsidian_global_search({
    query: "autonexus/finding severity",
    searchInPath: "Projects/{ProjectName}/",
    contextLength: 200,
    pageSize: 20
  })

Parse results:
  GROUP findings by:
    - OWASP category (e.g., A01 appears in 3 past audits)
    - Location pattern (e.g., same file flagged repeatedly)
    - STRIDE category frequency

  IF a pattern appears in 2+ past audits:
    FLAG as "Recurring Pattern" in the current audit report
    ELEVATE priority for testing that vector
    LOG: "Recurring pattern detected: {owasp_category} in {file} — flagged in {N} past audits"
```

This ensures systemic issues do not get repeatedly flagged without resolution.

---

## Report Output — Structured Folder

Every `/autonexus:security` run creates a dedicated folder inside a `security/` directory at the project root.

### Folder Structure

```
{project_root}/
└── security/
    ├── 260315-0945-stride-owasp-full-audit/
    │   ├── overview.md
    │   ├── threat-model.md
    │   ├── attack-surface-map.md
    │   ├── findings.md
    │   ├── owasp-coverage.md
    │   ├── dependency-audit.md
    │   ├── recommendations.md
    │   └── security-audit-results.tsv
    └── ...
```

### Folder Naming Convention

```
security/{YYMMDD}-{HHMM}-{audit-type-slug}/
```

**Slug generation rules:**
- If no scope/focus specified: `stride-owasp-full-audit`
- If scope is auth-related: `auth-authorization-audit`
- If scope is API-related: `api-security-audit`
- If scope is infra-related: `infrastructure-security-audit`
- If user provides a focus string: kebab-case it (e.g., "payment flow" -> `payment-flow-audit`)

---

## Historical Comparison

When a previous audit exists in `security/`, the current run automatically generates a comparison section.

```markdown
## Historical Comparison

**Previous audit:** security/260310-1430-stride-owasp-full-audit/ (5 days ago)

### Trend
| Metric | Previous | Current | Change |
|--------|----------|---------|--------|
| Critical | 3 | 1 | improved |
| High | 4 | 5 | regressed |
| Medium | 2 | 3 | +1 |
| Total | 9 | 9 | 0 |
| OWASP coverage | 6/10 | 8/10 | +2 |
| STRIDE coverage | 4/6 | 5/6 | +1 |
```

---

## Error Recovery

| Error | Recovery |
|-------|----------|
| Can't determine tech stack | Ask user for framework/language |
| No API routes found | Scan for all exported functions with HTTP-like patterns |
| Dependency audit fails | Skip, note in report, continue with code analysis |
| Code too large for context | Focus on files matching attack surface (API, auth, DB) |
| False positive suspected | Mark as "Possible" confidence, include caveats |
| Obsidian unavailable | Continue without knowledge queries — local TSV + report still generated |

## Anti-Patterns

- **Do NOT report theoretical risks without code evidence** — every finding needs a file:line reference
- **Do NOT skip categories** — the loop should aim for 100% OWASP + STRIDE coverage
- **Do NOT auto-fix vulnerabilities unless --fix is set** — report only by default
- **Do NOT test against live production** — analyze code statically, suggest dynamic tests
- **Do NOT report the same finding twice** — check results log for duplicates before logging
- **Do NOT prioritize quantity over quality** — 5 confirmed critical > 50 theoretical lows
- **Do NOT block the loop on Obsidian failures** — use retry-then-queue protocol for all MCP calls
