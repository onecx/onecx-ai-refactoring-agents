# UI Migration Hard Rules (PrimeNG → Angular Material)

<!--
  Hard rules for the PrimeNG → Angular Material UI swap.
  This file is the shared rulebook the ui-migration-* agents read.
  Progress file: UI_MIGRATION_PROGRESS.md (NOT MIGRATION_PROGRESS.md).
-->

## State Management
- `UI_MIGRATION_PROGRESS.md` is the ONLY source of truth for the UI migration — read it before every action.
- NEVER touch `MIGRATION_PROGRESS.md` — the Angular version-migration progress file (a separate scope). This migration's only progress file is `UI_MIGRATION_PROGRESS.md`.
- Use only 3 markers: `[ ]` not started, `[x]` completed, `[-]` not applicable.
- Update in-place (find the exact task entry, replace that section) — never append to the end.
- Read the ENTIRE file before writing to it.
- Task numbering must be contiguous (A.1, A.2, A.3 — no gaps) within a phase.
- Parent tasks stay `[ ]` while any child sub-task is `[ ]` — mark a parent `[x]` ONLY when ALL children are complete.
- If the target Material major conflicts with repo state (e.g. Material major ≠ `@angular/core` major) → STOP, ask the user which state is correct — never guess.

## Initialization Gates (MANDATORY order — HALT on any failure)

Run these BEFORE any planning or execution. The order is fixed and non-negotiable.

### Gate 0 — Progress check
- Check whether `UI_MIGRATION_PROGRESS.md` exists in the workspace root.
- If it does NOT exist → gates were never passed; run Gate 1 → Gate 2 → Gate 3, then Discovery/Planning (create the file).
- If it EXISTS → read it, show status, resume at the first `[ ]` task (planning already passed). Do NOT re-run Discovery from scratch.

### Gate 1 — Baseline (HALT)
- `npm run build`, `npm run lint`, AND `npm run test` must ALL pass.
- Record all three results (plus lint warning count + test coverage as baselines) in the Baseline section.
- If ANY fails → **HALT** and report the exact error to the developer. Do NOT proceed to Gate 2, Discovery, or any task. Do NOT attempt the fix yourself.

### Gate 2 — Branch (HALT)
- `git branch --show-current` must NOT be `main`, `master`, `dev`, or `develop`.
- If it is one of those → **HALT** and ask the developer to create a feature branch. Never work on a protected branch.

### Gate 3 — MCP (HALT-or-bootstrap)
- Verify the **PrimeNG MCP** and a **Material-side MCP** are configured (see the data file's "MCP Gate" section).
- If EITHER is missing → **HALT** and either (a) ask the developer to configure it, or (b) with the developer's OK, add the config to the target repo's MCP file (`mcp.json` / `.mcp.json` / `.vscode/mcp.json`) from the `mcp.example.json` template, then re-verify.
- Never proceed to Discovery with a missing MCP that the flow depends on.

- If any gate fails, stop and report to the developer — do not attempt fixes yourself.

## Task Execution
- Executor handles exactly ONE `[ ]` task per spawn. The orchestrator decides continue-or-stop via stop conditions (see orchestrator agent).
- At the START of every executor invocation: STRICTLY read the active data file (`.claude/instructions/ui-migration-prime-to-material.instructions.md`) — it carries the MCP source list, the component-pair starter table, the PrimeFlex→Tailwind mapping, and repo-specific rules that are NOT auto-injected.
- One task per PrimeNG component family (and per PrimeFlex→Tailwind surface) + one final cleanup task. Contiguous numbering per phase.
- **Search scope**: ALWAYS search the ENTIRE repository (all directories, all file types: `.ts`, `.html`, `.scss`, `.css`, `.json`, `.js`) — NEVER limit to a single folder. Use `grep -r` across the whole project root.
- **Element context is mandatory**: before transforming a component, understand its use-case in the repo (where it's used, what it drives) and check the relevant CSS (component overrides, global styles, `primeng-overwrites`/theme tokens) so the Material result is maximally similar — not just a mechanical tag swap.
- Check repository evidence (grep/search) for EVERY pattern before deciding applicability.
- Mark `[-]` not applicable ONLY when ALL patterns for the task are confirmed absent from the **entire codebase** (not just one folder).
- If ANY pattern applies → execute the task (do the applicable parts, note the rest). "One condition not met" ≠ "whole task not applicable."
- Each pattern must be independently checked — an `@primeng/x` import being absent does NOT mean `<p-x>` template tags or `pX` attribute usages are absent. Check every pattern separately.
- Component replacement must be DOC-DRIVEN: NEVER use hardcoded "never replace" lists or the starter table alone — re-derive each pair from the fetched docs per task and defer to what they say.
- If docs are silent or contradictory about replacing a component → stop and ask, do not guess from memory or a prior run.
- PrimeFlex→Tailwind is DOC/CONFIG-DRIVEN: map from the PrimeFlex utility set to Tailwind utilities using their corresponding values and measurements; re-derive the mapping per task; record source (MCP/node_modules/URL) for each.
- When in doubt about applicability, default to must-have (safer than skipping).

## Validation Rules (SCOPED — do not run the whole suite)
- **Per change (default)**: run the test for the SPECIFIC changed file(s) only — `npm run test -- --include='**/<changed>.spec.ts'` (or the `--config`/`ng test --include=` equivalent for the repo's runner) — **plus** `npm run build`.
- **Do NOT run the full test suite in the normal per-task flow** — it halts/destabilizes the system and is slow.
- **Full suite runs ONLY at**: (a) Gate 1 baseline, (b) phase-gate validations (A→B), (c) final verification (end of Phase B).
- Read COMPLETE output — return codes alone are insufficient.
- If build fails: capture the error (last 20+ lines), fix in the same invocation, rerun before validating lint.
- If lint fails: capture, fix, rerun — 0 errors, 0 NEW warnings vs baseline.
- If the scoped test fails: capture, fix, rerun until passing.
- Never mark `[x]` without all 8 evidence fields filled and validation passed.
- Lint must ALWAYS pass, including in Phase B (cleanup).

## Phase Boundaries
- Gate 0 → Planning: only after Gates 1–3 all pass.
- Phase 1 (planner) never executes code changes — discovery and planning only.
- Planning → Phase A: requires explicit developer approval of the generated `UI_MIGRATION_PROGRESS.md` (mandatory gate).
- Phase A → Phase B: requires explicit developer approval (mandatory gate). Phase B = cleanup + error-recovery (structural consolidation, leftover-import sweep, final verification). **No package removal in any phase** — `primeng`/`primeflex`/`primeicons` stay in `package.json` (required by the OneCX v8 libraries; removed only at the OneCX v9 / Angular 22 upgrade).

## No-Defer Rule
- You CANNOT defer any sub-step to a later phase or a future invocation.
- If a task needs a package not yet installed (e.g. `@angular/material-icons` for icon ligatures): confirm the major matches `@angular/core`, resolve an explicit stable version (`npm view <pkg>@<major> dist-tags` — never `@latest`), and install it in the same invocation. Installing a design-system package is an architectural decision — get developer confirmation first.
- Do NOT remove `primeng`/`primeflex`/`primeicons` at any point — they are KEPT in `package.json` for the whole migration (required by the OneCX v8 libraries). Their removal is out of scope and happens at the OneCX v9 / Angular 22 upgrade.

## Error Handling
- Errors during execution are YOUR job to fix in the SAME invocation — never defer.
- Error retry limit: attempt up to 3 different approaches per error — after 3 failures, escalate to the developer with the full attempt log.
- Log-based debugging: capture the **last 20 lines** of error output — never summarize, copy the actual output.
- Before declaring an import/API unresolvable: (1) `grep -r "Symbol" node_modules/@angular/material/ node_modules/@angular/cdk/ -l`, (2) read `node_modules/@angular/material/<x>.d.ts`, (3) check the package's public API / CHANGELOG. Only after all return nothing → escalate to the user with full evidence.
- Never silently drop functionality to silence an error. If a behavioral difference (PrimeNG vs Material) cannot be resolved from docs, record it in the task's Edge Cases field.

## Verification Checklist (Before Marking [x])
- Before marking ANY task `[x]`, verify ALL sub-steps are complete.
- For component migration: old import removed (grep proof), new import added, template updated, API mapping matches, CSS updated where needed, no duplicate module/provider in bootstrap.
- For PrimeFlex→Tailwind: old utility classes replaced (grep proof), Tailwind utilities present with matching measurements, no leftover PrimeFlex classes in scope.
- After migration, run `grep -r` for the OLD import/tag/class across the entire codebase — if any remain, the task is NOT complete.
- Provider/module deduplication: if a Material module/provider belongs in the root bootstrap, keep it ONLY there — remove it from the component.
- If ANY sub-step is unchecked → the task stays `[ ]` — note which parts are incomplete.
- Compare changes against a working Material example in the repo if one exists (e.g. an already-migrated component).
- ONLY mark `[x]` when ALL sub-steps verified AND validation passed.

## CSS and File Scope
- NEVER modify files in other repositories (e.g. shell-ui, workspace-ui) or `node_modules` to fix issues in YOUR application.
- CSS fixes belong in YOUR application's component styles or global styles — not in library files or other microfrontends.
- `pre_loaders/**` (auto-generated, version-pinned stubs) and `node_modules` are OUT of scope — record any grep hits there as "excluded — pre_loaders/legacy scope", never edited.
- If a CSS issue seems to require library changes, document it and escalate to the developer.

## Test Strategy
- During component migration: focus on getting the component working first, then fix its scoped test.
- Do not let a scoped test failure block the migration work — record it, complete the migration, then fix the scoped test.
- The full suite runs only at baseline / phase gates / final verification.

## Decision Protocol
- ASK the user for: gate failures, ambiguous component-pair candidates (two plausible Material components for one PrimeNG one), behavioral differences with no clean equivalent, phase gates, package installs, risky adaptations.
- DO NOT ask for: routine searches, applicability checks, next-task routing, whether to read docs.
- If docs are ambiguous: stop, document the ambiguity, ask ONE concise question with the concrete candidates and the repo grep evidence.
- Only the orchestrator routes to other agents — agents never call each other directly.
