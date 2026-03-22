# /autonexus:plan — Goal to Config Wizard

The hardest part isn't the loop — it's defining Scope, Metric, and Verify correctly. `/autonexus:plan` converts your plain-language goal into a validated, ready-to-execute configuration.

## Syntax

```
/autonexus:plan
Goal: Make the API respond faster

/autonexus:plan Increase test coverage to 95%

/autonexus:plan Reduce bundle size below 200KB
```

## What It Does

7 steps:

1. **Capture Goal** — ask if not provided
2. **Analyze Context** — scan codebase + query Obsidian Knowledge Center
3. **Define Scope** — suggest file globs, validate they resolve to real files
4. **Define Metric** — suggest mechanical metrics, validate they output a number
5. **Define Guard** — optional regression prevention
6. **Define Direction** — higher or lower is better
7. **Define Verify** — construct command, **dry-run it**, confirm it works
8. **Confirm & Launch** — present config, write Decision note to Obsidian

## Critical Gates

- Metric MUST be mechanical (outputs a parseable number)
- Verify command MUST pass a dry run
- Scope MUST resolve to at least 1 file

## Obsidian Integration

- **Phase 2:** Reads existing Knowledge Center for project context (if available)
- **Phase 2:** Searches past plan decisions to suggest previously successful configs
- **Phase 7:** Writes a Decision note documenting the plan (goal, scope, metric, alternatives considered)

## Examples

```
/autonexus:plan
Goal: Make the API respond faster
# → Wizard suggests: Scope: src/api/**/*.ts, Metric: p95 ms, Verify: npm run bench

/autonexus:plan
Goal: Improve code quality
# → Wizard suggests: Scope: src/**/*.ts, Metric: lint errors, Verify: eslint . --format json

/autonexus:plan
Goal: Reduce Docker image size
# → Wizard suggests: Scope: Dockerfile, Metric: image MB, Verify: docker build . | grep "size"
```

## After the Wizard

You get a ready-to-paste `/autonexus` config, or launch directly.
