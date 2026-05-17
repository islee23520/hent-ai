# Execution-Ready Spec — Repository Technical Gap Audit

## Metadata
- Source: `$deep-interview`
- Profile: standard
- Context type: brownfield
- Final ambiguity: 15.7%
- Threshold: 20%
- Context snapshot: `.omx/context/repo-tech-debt-audit-20260517T120634Z.md`
- Transcript: `.omx/interviews/repo-tech-debt-audit-<timestamp>.md`

## Intent
Find every technically insufficient implementation area in the Hent-ai repository so the project can move toward release readiness and subsequent fixes with a complete, prioritized defect/technical-debt inventory.

## Desired Outcome
Produce a comprehensive audit report covering immediately fixable bugs/defects, release-blocking quality/stability problems, and broader architecture/packaging/testing/CI technical debt.

## In Scope
- Runtime bugs and reproducible defects
- Type/build/test failures
- CI workflow gaps
- Packaging and release structure issues
- Cross-package architecture and coupling risks
- OpenClaw plugin behavior and Discord integration surfaces
- Onboarding/session/asset manifest behavior
- Image generation CLI behavior
- Cursor integration/classifier behavior
- Hermes integration/classifier behavior
- Documentation/implementation divergence where it affects technical correctness or release quality
- Security, performance, reliability, maintainability, DX, and test coverage gaps
- Low-severity items as well as high/critical issues
- Evidence-backed findings and separately marked inference/speculative risks

## Out of Scope / Non-goals
No exclusions. Everything technically relevant may be audited.

## Decision Boundaries
OMX may decide without additional confirmation:
- Severity classification: critical / high / medium / low
- Whether an item is evidence-backed or inferred/speculative
- Whether to run tests, builds, typechecks, and CI-equivalent local checks
- Whether packaging/release/docs divergence count as defects
- Fix priority and recommended repair ordering

Required boundary: separate evidence from inference/speculation.

## Constraints
- Deep-interview itself must not implement fixes.
- Audit should preserve evidence paths and commands/results where possible.
- Findings should be actionable for later fix execution.
- No dependency additions are implied by this spec.

## Testable Acceptance Criteria
The next audit/planning step is successful if it produces a report that:
1. Covers all major repo areas: `generate`, `openclaw`, `cursor`, `hermes`, root package/config, and `.github/workflows`.
2. Includes low-severity items, not only release blockers.
3. For each finding includes: severity, category, evidence, evidence type (`observed` vs `inferred`), impacted files, likely user/release impact, and fix difficulty.
4. Separates reproducible failures from code-review risks.
5. Includes a prioritized fix order.
6. Lists verification commands run and their results.
7. Clearly marks any unverified areas or audit blind spots.

## Brownfield Evidence Collected During Intake
- Root README defines Hent-ai as an emotion-image project for OpenClaw, Hermes Agent, Cursor, and image generation.
- TypeScript packages exist under `generate`, `openclaw`, and `cursor`.
- Hermes plugin files exist under `hermes`.
- Vitest tests exist across `generate`, `openclaw`, and `cursor`; Hermes test presence was reported by `omx explore` as `tests/hermes/test_hent_ai_plugin.py` and should be verified directly in the audit.
- `.github/workflows/ci.yml` and `.github/workflows/pr-checks.yml` exist.
- `openclaw/node_modules` and per-package lockfiles are present in the working tree and should be checked for repo hygiene/release implications.

## Initial Recon Risk Leads
These are not final findings; verify them in the audit:
1. Cross-package coupling: `generate/src/sets.ts` reportedly imports OpenClaw asset internals.
2. Error-handling fragility: non-SSE response parsing in `generate/src/codex.ts` may assume valid JSON.
3. Silent failure handling: onboarding session cleanup/restore may swallow errors.
4. Parser/regex brittleness: onboarding parsers may have broad/encoding-sensitive regexes.
5. Type-safety gap: OpenClaw registration surface may use `any`.
6. Classifier duplication: Cursor and Hermes may duplicate rule-based emotion detection without a shared source of truth.
7. Documentation/implementation divergence potential across runtimes.

## Recommended Handoff
Use `$ralplan` or direct audit execution. Recommended planning invocation:

```text
$plan --consensus --direct .omx/specs/deep-interview-repo-tech-debt-audit.md
```

If skipping consensus planning, execute an audit-only pass using this spec as binding requirements; do not start fixes until the inventory exists unless the user explicitly requests fixing.
