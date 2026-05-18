# Revised Consensus Plan — Hent-ai Technical Gap Inventory

## Requirements Summary
Source of truth: `.omx/specs/deep-interview-repo-tech-debt-audit.md`. Produce a comprehensive, evidence-dense inventory of technically insufficient implementation areas across the Hent-ai repository. The audit covers `generate`, `openclaw`, `cursor`, `hermes`, root/package configuration, assets, tests, docs, and `.github/workflows`. It must include low-severity items, classify severity and fix difficulty, and separate evidence buckets. It must not implement fixes.

## RALPLAN-DR Summary

### Principles
1. **Discovery before hotspots**: start with a repo-wide census so unknown gaps are not masked by known leads.
2. **Evidence taxonomy**: every finding must be bucketed as `observed-local`, `observed-workflow-config`, or `inferred-external`.
3. **Release contract realism**: verify what users/CI/package managers actually consume, not only source intent.
4. **Harness skepticism**: treat test aliases, stubs, coverage scopes, and CI warning gates as first-class audit surfaces.
5. **Inventory-only discipline**: produce the complete issue inventory before any code fix.

### Decision Drivers
1. Completeness across all shipped/runtime surfaces.
2. Reproducible or clearly classified evidence for later fixing.
3. Prioritized release-readiness decisions with low-to-critical coverage.

### Viable Options
#### Option A — Three-pass evidence-first comprehensive audit (favored)
Approach: repo-wide static census → non-mutating verification runs → blind-spot/parity pass for docs, publish/install contracts, CI/config, and external-runtime risks.
- Pros: balances completeness with reproducibility; surfaces unknown gaps; supports clean observed/inferred separation.
- Cons: slower and more process-heavy than a quick hotspot audit.

#### Option B — Static-first exhaustive taxonomy, limited command execution
Approach: broad static review of all code/config/docs first, then run only minimal smoke commands.
- Pros: strongest breadth and lowest environment dependency.
- Cons: more findings remain inferred; weaker confidence for build/test/package defects.

#### Option C — Verification-first release gate audit
Approach: run build/test/type/package/pack checks first, then inspect code only around failures and release blockers.
- Pros: fastest path to high-confidence release blockers.
- Cons: likely misses low-severity and architectural/test-harness debt required by the spec.

Rejected non-option: fix-as-you-audit, because the source spec requires inventory before fixing.

## ADR
### Decision
Use Option A: a three-pass evidence-first comprehensive audit.
### Drivers
Completeness, evidence quality, release-contract realism, and no-fix audit discipline.
### Alternatives considered
- Static-first taxonomy: broader but weaker at proving reproducible failures.
- Verification-first release gate: strong for blockers but incomplete for low/architecture debt.
- Fix-as-you-audit: invalid for this task because it violates the inventory-first boundary.
### Why chosen
Option A synthesizes Architect feedback: it preserves reproducible local verification while adding a repo-wide discovery census and explicit blind-spot/parity pass.
### Consequences
The audit will take longer than a quick review, but produces a report that can drive later `$ralph`/`$team` fixes without losing low-severity or config/doc debt.
### Follow-ups
After the audit report is approved, run a separate fix-planning/execution workflow. Do not fix during this audit.

## Acceptance Criteria
The plan is complete when the audit report:
1. Covers root config, `.github/workflows`, `generate`, `openclaw`, `cursor`, `hermes`, `assets`, tests, docs, lockfiles, committed build artifacts, stubs/mocks, and package manifests.
2. Includes findings from all severity levels: `critical`, `high`, `medium`, `low`.
3. Uses the required finding schema: ID, title, severity, category, evidence bucket, evidence, impacted files, likely impact, fix difficulty (`S/M/L`), confidence, and recommended fix direction.
4. Uses exactly these evidence buckets: `observed-local`, `observed-workflow-config`, `inferred-external`.
5. Separates release blockers from non-blocking technical debt.
6. Includes commands run, command outputs/summaries, skipped commands with reasons, and blind spots.
7. Includes a prioritized fix order but does not change source implementation files.
8. Explicitly audits docs/runtime parity, package tarball/install surfaces, test harness masking, CI soft-fail behavior, and external integration risks.

## Finding Schema
```md
### HENT-AUDIT-### — <title>
- Severity: critical | high | medium | low
- Category: build | test | ci | packaging | runtime | security | reliability | performance | maintainability | docs-parity | dx
- Evidence bucket: observed-local | observed-workflow-config | inferred-external
- Evidence: <file:line or command output summary>
- Impacted files: <paths>
- User/release impact: <concrete impact>
- Fix difficulty: S | M | L
- Confidence: high | medium | low
- Recommended fix direction: <not implementation, just direction>
```

## Execution Plan

### Phase 0 — Repo-wide discovery census
Create an audit matrix before hotspot analysis. Census:
- Entrypoints, package manifests, exports, bins, plugin manifests, hooks, scripts, workflows, test configs, TypeScript configs, Python plugin files, assets, docs, dist outputs, lockfiles, committed `node_modules`, stubs/mocks, TODO/FIXME, `any`, `@ts-ignore`, silent catches, `JSON.parse`, filesystem boundaries, network/fetch boundaries, auth/token handling, path handling, and generated artifacts.
- Required file anchors include root `package.json:1-6`, `.github/workflows/ci.yml:14-203`, `.github/workflows/pr-checks.yml:7-82`, `generate/package.json:6-18`, `cursor/package.json:7-19`, `hermes/plugin.yaml:1-5`, `openclaw/tsconfig.json:15-22`, `openclaw/vitest.config.ts:7-27`, and `generate/vitest.config.ts:3-18`.

### Phase 1 — Non-mutating verification runs
Run commands that do not intentionally modify source. Capture exact output and environment notes.
- Root: inspect scripts; root `package.json:1-6` currently has no scripts.
- `generate`: `npm run build`, `npm test` from scripts at `generate/package.json:15-18`.
- `cursor`: `npm run build`, `npm test` from scripts at `cursor/package.json:15-19`.
- `openclaw`: `npm test`; run `tsc --noEmit` or `npm exec tsc --noEmit` if dependency state permits; compare against CI’s warning-only type/lint gates at `.github/workflows/ci.yml:52-56`.
- `hermes`: `pytest tests/hermes/ -v` because CI expects it at `.github/workflows/ci.yml:136-138`; if path missing or failing, classify separately.
- Record failures as `observed-local`; record CI semantics/config problems as `observed-workflow-config`.

### Phase 2 — Config and harness masking audit
Audit whether tests/builds can give false confidence.
- `openclaw/tsconfig.json:15-19` aliases `@hent-ai/generate` to source and SDK entry to `test/stubs/plugin-entry.ts`; inspect release-vs-test implications.
- `openclaw/tsconfig.json:21-22` includes only `index.ts` and onboarding TS; confirm whether assets/scripts/tests are typechecked elsewhere.
- `openclaw/vitest.config.ts:8-19` aliases generated package to `test/stubs/generate.ts` and coverage to only `onboarding/parsers.ts`; classify blind spots.
- `generate/vitest.config.ts:7-15` limits coverage to `src/generator.ts`; identify unmeasured `codex.ts`, CLI, sets, and package coupling.
- Check all `test/stubs/*` and fixture assumptions.

### Phase 3 — Publish/install contract validation
Validate what would actually ship and install.
- For `generate`, `cursor`, and `openclaw`, run or plan `npm pack --dry-run` with a local cache; capture tarball file lists.
- Compare package `files`, `bin`, `exports`, `main`, plugin manifests, and docs.
- Required checks:
  - `generate/package.json:6-13` exports `dist` and bin `./dist/main.js`; inspect whether cross-package import from `generate/src/sets.ts:3-12` leaks OpenClaw internals into built output or package contents.
  - `cursor/package.json:7-13` declares `bin/hent-ai.js`, `dist-cli/install.js`, and `assets/optimized/*.png`; verify those files exist in tarball and after build.
  - `openclaw/package.json` / `openclaw/openclaw.plugin.json`: verify extension package includes every runtime file it references, not only `index.ts`.
- Findings from package manifests/tarballs are `observed-local` if from local `npm pack`; `observed-workflow-config` if from package/CI contract analysis.

### Phase 4 — Docs/runtime parity audit
Compare install/run claims with actual package and runtime surfaces.
- Root README `README.md:23-29` describes supported platforms and setup paths.
- Root README `README.md:47-53` documents `generate` commands.
- OpenClaw docs `openclaw/README.md:25-37` clone/install, `87-105` config, and `126-139` build/restart claims.
- Cursor docs `cursor/README.md:7-41` advertise `npx hent-ai-cursor install/status/uninstall` flows; compare against `cursor/package.json:7-13` and built files.
- Hermes docs `hermes/README.md:7-28` copy/symlink install; compare with `hermes/__init__.py:95-108` asset resolution and `hermes/plugin.yaml:1-5` hooks.

### Phase 5 — Runtime reliability and error-handling audit
Inspect source for actionable reliability defects.
- `generate/src/codex.ts`: auth JSON parse around line 110, fetch/timeout behavior around lines 154-179, non-SSE `JSON.parse(responseBody)` around line 199, and safety rephrase loop around lines 236-256.
- `generate/src/generator.ts`: output directory creation and partial output behavior around lines 79-123.
- `openclaw/onboarding/session.ts:75-92` and `110-112`: silent catches and persistence corruption behavior.
- `openclaw/onboarding/parsers.ts:10-15`: encoding-corrupt-looking token and regex ordering/false positives.
- `openclaw/index.ts`: Discord edit/send/append fetch paths around `994-1013`, `1043-1062`, `1249-1306`; token/logging/retry/attachment preservation concerns.
- `hermes/__init__.py:131-158`: path/media directive behavior, platform gating, missing assets behavior.

### Phase 6 — Classifier parity, maintainability, and UX/DX audit
- Compare rule definitions across `cursor/src/classifier/ruleClassifier.ts:3-65`, `hermes/__init__.py:35-118`, and `openclaw/index.ts:638-667`.
- Identify divergence in Korean/English support, substring matching, default emotion behavior, and rule ordering.
- Audit CLI ergonomics and validation in `generate/src/cli.ts` and `cursor/bin/install.ts`.
- Track duplication as maintainability/DX findings when it creates inconsistent behavior.

### Phase 7 — Security, performance, and external integration risk pass
Classify risks that cannot be fully observed locally as `inferred-external` unless backed by config/workflow evidence.
- Security: token sourcing/logging, path traversal or filename sanitization, attachment path construction, env var handling, external API base URLs.
- Performance/reliability: sequential image generation, rate limiting, Discord API retries, timeout defaults, large image handling, memory pressure from buffers.
- External integrations: OpenClaw SDK availability, Discord API behavior, Hermes hook contract, Codex/OpenAI image generation behavior.

### Phase 8 — CI and release gate audit
- `.github/workflows/ci.yml:36-50`: package mutation before install.
- `.github/workflows/ci.yml:52-56`: typecheck/lint downgrade to warning.
- `.github/workflows/ci.yml:61-62`: coverage downgrade to warning.
- `.github/workflows/ci.yml:106-138`: Hermes pytest path and dependency assumptions.
- `.github/workflows/ci.yml:144-203`: integration job dependencies and whether Cursor is omitted.
- `.github/workflows/pr-checks.yml:20-23`, `35-48`, `60-82`: labeler continue-on-error, PR size warning-only, shallow breaking-change heuristics.

### Phase 9 — Final report assembly and validation
Write a final audit report, preferably `.omx/reports/repo-tech-debt-audit.md`, with:
- Executive summary
- Release blockers
- Full findings table
- Findings by area
- Inferred external risks
- Verification log
- Blind spots
- Prioritized fix sequence
- Suggested next workflow (`$ralph` or `$team`) after report approval
Validate the report against all acceptance criteria and the finding schema. Confirm no source implementation files were modified.

## Risks and Mitigations
- **Risk: known-lead bias**. Mitigation: Phase 0 census before hotspot review.
- **Risk: local environment differs from CI or user installs**. Mitigation: separate `observed-local` from `observed-workflow-config`, and inspect workflow/package contracts.
- **Risk: tests are misleading because of stubs/aliases**. Mitigation: dedicated config/harness audit.
- **Risk: live Discord/OpenClaw/Hermes/Codex behavior cannot be fully reproduced**. Mitigation: mark `inferred-external`, include manual/e2e verification recommendations.
- **Risk: audit drifts into fixing**. Mitigation: no implementation edits; final report only includes recommended fix direction.

## Verification Steps for This Plan
- It includes repo-wide census, verification, config/harness, publish/install, docs parity, runtime, classifier, security/performance, CI, and report assembly phases.
- It uses the three evidence buckets requested by reviewers.
- It contains testable acceptance criteria and concrete file anchors.
- Post-audit orchestration is separated into the appendix below and is not part of the audit execution core.

## Applied Consensus Improvements
- Added Phase 0 repo-wide discovery census.
- Added first-class config/harness audit.
- Added explicit `npm pack --dry-run` publish/install contract validation.
- Added docs/runtime parity phase.
- Replaced two evidence buckets with `observed-local`, `observed-workflow-config`, `inferred-external`.
- Moved staffing/launch guidance into an appendix to reduce execution drift.

---

# Appendix — Post-Audit Handoff Guidance

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

## `$ralph` Follow-up Staffing Guidance
Use after the audit report exists and the user wants sequential fix execution. Suggested lanes: `debugger` for reproducible failures, `architect` for package/CI contracts, `test-engineer` for regression coverage, `security-reviewer` for trust boundaries, `verifier` for final proof. Use high reasoning for architecture/debugging/security/verification.

## `$team` Follow-up Staffing Guidance
Use if the report has many independent fix lanes. Suggested team lanes:
1. Build/package lane — `debugger`/`executor`.
2. OpenClaw runtime/onboarding lane — `code-reviewer`/`executor`.
3. Cursor/Hermes parity lane — `executor`/`test-engineer`.
4. CI/test harness lane — `test-engineer`/`build-fixer`.
5. Security/reliability lane — `security-reviewer`.
6. Final validation lane — `verifier`/`writer`.

## Launch Hints
```text
$ralph .omx/reports/repo-tech-debt-audit.md
$team .omx/reports/repo-tech-debt-audit.md
omx team --task "Fix prioritized findings from .omx/reports/repo-tech-debt-audit.md"
```

## Team Verification Path
Team proves each lane fixed assigned findings with tests and no scope creep. Ralph/verifier then proves the release-blocker subset is closed, regression tests pass, and remaining risks are explicitly accepted or deferred.
