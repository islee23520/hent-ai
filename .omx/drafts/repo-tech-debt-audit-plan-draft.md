# Draft Consensus Plan — Hent-ai Technical Gap Inventory

## Requirements Summary
Use `.omx/specs/deep-interview-repo-tech-debt-audit.md` as source of truth. Produce a comprehensive, evidence-dense audit inventory across `generate`, `openclaw`, `cursor`, `hermes`, root package/config, assets, and `.github/workflows`. Do not fix source code during the audit. Include low-severity issues and separate observed evidence from inferred risks.

## RALPLAN-DR Summary

### Principles
1. Evidence first: every finding must cite file/line, command output, or be explicitly marked inferred.
2. Release readiness: prioritize defects that can break install, build, packaging, runtime integration, or CI trust.
3. Whole-repo coverage: audit all shipped/runtime surfaces, not only currently tested TypeScript packages.
4. Actionability: every finding needs severity, category, impact, fix difficulty, and recommended next step.
5. No implementation during audit: this plan produces an inventory, not fixes.

### Decision Drivers
1. Completeness across all Hent-ai runtimes (`generate`, `openclaw`, `cursor`, `hermes`).
2. Trustworthy evidence trail for later fixing and release decisions.
3. Clear prioritization by severity and repair effort.

### Viable Options
#### Option A — Evidence-first local verification audit (favored)
Pros: produces reproducible failures, validates CI/build/test reality, minimizes speculative noise. Cons: may miss defects requiring live Discord/OpenClaw/Hermes/Codex environments.
#### Option B — Static architecture/code-review inventory only
Pros: faster, safer, no environment sensitivity. Cons: weaker release confidence and fewer reproducible findings.
#### Option C — Fix-as-you-audit execution
Pros: immediate cleanup. Cons: violates user’s requested inventory-first output and risks losing complete issue coverage.

Favored option: Option A, with static review and inference buckets added after verification.

## Acceptance Criteria
1. Final audit report exists under `.omx/reports/` or `.omx/specs/` and covers: root configs, `.github/workflows`, `generate`, `openclaw`, `cursor`, `hermes`, `assets`, and tests.
2. Each finding records: ID, title, severity (`critical/high/medium/low`), category, evidence type (`observed` or `inferred`), evidence path/line or command output, impacted files, likely impact, fix difficulty (`S/M/L`), and recommended fix direction.
3. Observed failures are backed by command output from at least: package test/build/typecheck where available, Python tests for Hermes, and CI workflow inspection.
4. Inferred risks are explicitly separated from observed failures and include the rationale for why they matter.
5. Report includes prioritized fix order and “release blockers” subset.
6. Report lists commands run, commands skipped, and audit blind spots.
7. No source code fixes are made during audit execution.

## Implementation / Audit Steps

### 1. Establish repo map and audit matrix
- Read and cite package/workflow surfaces: root `package.json:1-6`, `.github/workflows/ci.yml:14-203`, `.github/workflows/pr-checks.yml:7-82`, `generate/package.json:1-37`, `openclaw/package.json`, `cursor/package.json:1-31`, `hermes/plugin.yaml:1-5`.
- Build an audit matrix with rows for runtime area and columns for build, test, packaging, runtime integration, docs parity, security/reliability, and release readiness.

### 2. Run non-mutating verification commands and capture output
- Root: inspect lockfiles/node_modules hygiene and run any available root-level scripts if present; root `package.json:1-6` currently has devDependencies only and no scripts.
- `generate`: `npm run build`, `npm test`; package scripts are at `generate/package.json:15-18`.
- `cursor`: `npm run build`, `npm test`; scripts are at `cursor/package.json:15-19`.
- `openclaw`: `npm test` and typecheck via `tsc --noEmit` if install state permits; CI currently weakens type/lint failures with warnings in `.github/workflows/ci.yml:52-56`.
- `hermes`: run `pytest tests/hermes/ -v` if test path exists; CI expects it at `.github/workflows/ci.yml:136-138`.
- Record environment blockers separately from code defects.

### 3. Audit packaging and release contracts
- Verify `generate` exports only `dist` (`generate/package.json:6-13`) while source imports OpenClaw internals in `generate/src/sets.ts:3-12`; determine whether built package includes/depends on unavailable files.
- Check Cursor bin/files consistency: `cursor/package.json:7-13` references `bin/hent-ai.js`, `dist-cli/install.js`, and `assets/optimized/*.png`; verify those files exist after build/package.
- Check OpenClaw package contract: `openclaw/package.json`, `openclaw/openclaw.plugin.json`, and CI mutation of package deps in `.github/workflows/ci.yml:36-50`.
- Check generated `dist/`, committed `node_modules`, per-package lockfiles, and root package manager mismatch implications.

### 4. Audit runtime error handling and reliability
- `generate/src/codex.ts`: verify auth parsing around `JSON.parse` (`rg` showed line 110) and response parsing around non-SSE `JSON.parse(responseBody)` (`rg` showed line 199); classify malformed auth/HTTP body behavior.
- `openclaw/onboarding/session.ts:75-92` and `110-112`: silent catches may hide corrupt sessions or filesystem failures.
- `openclaw/index.ts`: inspect Discord fetch paths around `994-1013`, `1043-1062`, `1249-1306`; classify API error handling, attachment preservation, token handling, logging sensitivity, and retry behavior.
- `generate/src/generator.ts:79-123`: inspect sequential generation, file writes, partial-output behavior, and reference image assumptions.

### 5. Audit classifier correctness and parity
- Compare emotion rules in `cursor/src/classifier/ruleClassifier.ts:3-65`, `hermes/__init__.py:35-118`, and `openclaw/index.ts:638-667` for divergence.
- Verify Korean/English coverage parity and false-positive risks; `openclaw/onboarding/parsers.ts:10-15` includes an encoding-corrupt-looking token and broad regex ordering.
- Check tests cover mixed-language, ambiguous, multi-emotion, and substring cases.

### 6. Audit onboarding, channel override, and asset manifest behavior
- Inspect `openclaw/onboarding/*`, `openclaw/assets/*`, `openclaw/test/*`, and docs for channel/workspace asset isolation.
- Confirm manifest migration/corruption handling, channel mode persistence, private-mode behavior, and default asset fallback.
- Classify missing tests for corrupt files, permissions, multi-channel concurrent sessions, and Discord API failures.

### 7. Audit CI quality gates
- `.github/workflows/ci.yml:52-56` converts typecheck/lint failures to warnings; decide severity because CI may pass with type errors or absent lint config.
- `.github/workflows/ci.yml:61-62` makes coverage non-blocking.
- `.github/workflows/ci.yml:144-203` integration job only depends on `openclaw` and `generate`, not `hermes`; evaluate whether Cursor is absent from CI.
- `.github/workflows/pr-checks.yml:20-23` labeler is `continue-on-error`; PR size is warning-only; changeset checks are shallow heuristics.

### 8. Produce inventory and prioritized fix roadmap
- Write final report with sections: Executive summary, release blockers, all findings, inferred risks, verification log, blind spots, recommended fix sequence.
- Use IDs like `HENT-AUDIT-001` and include fix difficulty.
- Do not edit implementation files.

## Risks and Mitigations
- Risk: local dependency state differs from CI. Mitigation: record exact command and environment; compare with workflow commands.
- Risk: live provider/Discord behavior cannot be fully tested locally. Mitigation: mark as inferred or blocked, propose e2e/manual verification.
- Risk: audit becomes too broad. Mitigation: enforce matrix coverage and finding schema; include low severity but prioritize release blockers.
- Risk: false positives from static review. Mitigation: keep evidence/inference separation.

## Verification Steps for the Plan Itself
- Confirm referenced files exist.
- Confirm final plan includes ADR, staffing guidance, and team/Ralph handoff hints.
- During audit execution, verify no implementation source files were modified.

## ADR
### Decision
Run an evidence-first comprehensive audit inventory before any fixes.
### Drivers
Completeness, release-readiness confidence, and actionable prioritization.
### Alternatives considered
- Static-only audit: rejected as insufficient for release confidence.
- Fix-as-you-audit: rejected because it obscures full inventory and violates planning boundary.
### Why chosen
It balances reproducible failures with whole-repo static review and preserves a clean handoff to later fixing.
### Consequences
Audit takes longer than a quick review but yields a more trustworthy backlog.
### Follow-ups
Use `$ralph` or `$team` to execute fixes after report approval.

## Available-Agent-Types Roster
- `explore`: fast file/symbol mapping.
- `planner`: sequencing and risk planning.
- `architect`: architecture/package boundary review.
- `debugger`: reproduce failures and root cause.
- `executor`: implement later fixes, not during audit planning.
- `test-engineer`: test coverage and verification design.
- `security-reviewer`: token/path/API trust boundary review.
- `code-reviewer`: comprehensive static review.
- `verifier`: evidence validation and completion proof.
- `writer`: final report polish.

## Follow-up Staffing Guidance
### `$ralph` sequential audit/fix path
Use one persistent owner plus targeted subagents: `debugger` for verification failures, `architect` for package/CI contracts, `test-engineer` for coverage gaps, `verifier` for final evidence. Suggested reasoning: high for architect/debugger/verifier, medium for writer.

### `$team` parallel audit path
Recommended lanes:
1. `debugger` — run builds/tests/typechecks and collect observed failures.
2. `architect` — packaging/monorepo/CI/release contracts.
3. `code-reviewer` — OpenClaw runtime/onboarding/Discord reliability.
4. `test-engineer` — test coverage matrix and missing regression cases.
5. `security-reviewer` — token handling, path traversal, external API boundaries.
6. `writer`/`verifier` — consolidate report and validate evidence.

## Launch Hints
```text
$ralph .omx/plans/repo-tech-debt-audit-plan.md
$team .omx/plans/repo-tech-debt-audit-plan.md
omx team --task "Execute repo technical gap audit from .omx/plans/repo-tech-debt-audit-plan.md"
```

## Team Verification Path
Team proves: all lanes report findings in schema, commands/output are captured, and no source fixes occurred. Ralph/verifier after handoff proves: final report covers every required repo area, evidence paths are valid, observed/inferred separation is maintained, and prioritized fix order is complete.
