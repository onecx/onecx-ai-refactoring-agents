# UI Migration Hard Rules (PrimeNG → Material / Optimus)

<!--
  Hard rules for the PrimeNG → target UI swap. The target is Angular Material
  OR Optimus UI — chosen by the developer and recorded in the progress file's
  `Target:` field (NEVER defaulted). Tailwind (PrimeFlex→Tailwind) applies to
  BOTH targets.
  This file is the SHARED, target-agnostic rulebook the ui-migration-* agents
  read. The target-specific data file (prime-to-material / prime-to-optimus)
  carries the MCP/WebFetch source list, component mapping, and Tailwind details.
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
- If the **target** package major conflicts with repo state (e.g. Material major ≠ `@angular/core` major; for Optimus, the installation page's peer-dep Angular range) → STOP, ask the user which state is correct — never guess.
- For an Optimus migration, also read `.claude/instructions/ui-migration-onecx-version-compat.instructions.md` before planning/execution. Record installed OneCX, Optimus, Tailwind and PrimeFlex versions; API-compatible selectors do not imply OneCX theme/CSS compatibility.

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

### Gate 3 — Docs / MCP (TARGET-AWARE, HALT-or-bootstrap)

- Always verify the **PrimeNG MCP** (OLD side) and the **Angular CLI MCP** (`mcp__angular-cli__*` — build/devserver validation, either target).
- **Material target**: also verify a **Material-side MCP** (`ngm-dev-blocks` + Material MCP) is configured (see the Material data file's "MCP Gate" section).
- **Optimus UI target**: there is **NO MCP** — verify version-matched official sources via WebFetch (fall back to `curl -sSL` if policy-blocked): v1/Angular 21 uses `https://v1.optimus.openng.org/installation`, `/migration/primeng`, `/tailwind`, `/theming/styled`, and `/configuration`; v2/Angular 22 uses the current Optimus docs. If unavailable, use the **Doc-source Fallback** and inspect installed OneCX packages as well as Optimus; HALT only if package/source evidence is unavailable.
- If anything the flow depends on is missing → a **missing MCP** is bootstrap-able: (a) ask the developer to configure it, or (b) with the developer's OK, add the MCP config to the target repo's MCP file (`mcp.json` / `.mcp.json` / `.vscode/mcp.json`) from the `mcp.example.json` template, then re-verify. An **unreachable doc/URL source** (not a missing MCP) instead falls through to the **Doc-source Fallback** (node_modules trace) before any HALT.
- Never proceed to Discovery with a missing doc/MCP source that the flow depends on — and never proceed on a mapping that was neither doc-sourced **nor** node_modules-verified.

- If any gate fails, stop and report to the developer — do not attempt fixes yourself.

## Doc-source Fallback (both targets) — MCP/WebFetch → node_modules → then HALT

The primary API/doc source differs by target — **Material:** the MCPs; **Optimus:** WebFetch/`curl` of the two official pages; **PrimeFlex values:** `node_modules/primeflex/`; **Tailwind values:** the Tailwind MCP. When the primary source is **unreachable, policy-blocked, or returns nothing**, do **NOT** immediately HALT and do **NOT** guess. Fall back in this order until you have evidence you can VERIFY:

1. **Retry / alternate transport** — Optimus URLs: WebFetch → `curl -sSL`. An MCP: probe a second tool or the alternate MCP (e.g. `mcp__angular-cli__search_documentation` before giving up on the Material MCP).
2. **Trace `node_modules/` — the LAST-RESORT evidence source, and VERIFY it.** The installed packages ARE the authoritative API source; read them to gather AND verify the evidence in place of the web, and record `node_modules/<pkg>` as the source:
   - **OLD side (both targets):** `node_modules/primeng/` — `package.json` (subpath exports + installed major), the component's `.d.ts` (the real exported directives/classes/services + inputs/outputs), `CHANGELOG`; `node_modules/primeflex/primeflex.css` (measured utility values); `node_modules/primeicons/primeicons.css` (glyph classes).
   - **NEW side — Material:** `node_modules/@angular/material/<component>/…d.ts` (the real `MatXxx` export names, inputs/outputs) and `node_modules/@angular/cdk/`.

- **NEW side — Optimus:** `node_modules/@openng/optimus-ui/` (entry points/config/style hook), `node_modules/@openng/optimus-ui-themes/`, the matching Tailwind plugin, `node_modules/@openng/icons/openng-icons.css`; and `node_modules/@onecx/angular-utils/` + `@onecx/angular-accelerator/` (theme provider exports/source, peer deps, stylesheet imports). Verify both rename and version-specific integration.
- **Tailwind (size parity):** `node_modules/tailwindcss/` default theme/scale — confirm the exact value of the chosen utility (the authoritative source when the Tailwind MCP is down).
- The scoped `npm run build` (executor Step 6) is the **final verification**: a renamed/derived import that RESOLVES and BUILDS proves the node_modules-traced evidence was correct. Record the trace in the task's 8 evidence fields (which `.d.ts`/`package.json`/asset you read + what it confirmed).

3. **HALT only if `node_modules` also yields no evidence** — e.g. the target package is not yet installed, or its `.d.ts`/assets are absent. Then report exactly what is missing and HALT (do not proceed on a guessed mapping).

This replaces "unreachable → HALT" as the default: **HALT becomes the LAST step, after an honest `node_modules` trace has been attempted and recorded.**

## Task Execution

- Executor handles exactly ONE `[ ]` task per spawn. The orchestrator decides continue-or-stop via stop conditions (see orchestrator agent).
- At the START of every executor invocation: read the **active data file** for the target. For Optimus also read `.claude/instructions/ui-migration-onecx-version-compat.instructions.md`. Read `.claude/instructions/ui-migration-primeflex-to-tailwind.instructions.md` for any Tailwind task. These files are mandatory; do not assume same class names imply same CSS values or that a rename migrates OneCX theme integration.
- One task per PrimeNG component family (and per PrimeFlex→Tailwind surface) + one final cleanup task. Contiguous numbering per phase.
- **Search scope**: ALWAYS search the ENTIRE repository (all directories, all file types: `.ts`, `.html`, `.scss`, `.css`, `.json`, `.js`) — NEVER limit to a single folder. Use `grep -r` across the whole project root.
- **Element context is mandatory**: before transforming a component, understand its use-case in the repo (where it's used, what it drives) and check the relevant CSS (component overrides, global styles, `primeng-overwrites`/theme tokens) so the target result is maximally similar — not just a mechanical tag swap.
- Check repository evidence (grep/search) for EVERY pattern before deciding applicability.
- Mark `[-]` not applicable ONLY when ALL patterns for the task are confirmed absent from the **entire codebase** (not just one folder).
- If ANY pattern applies → execute the task (do the applicable parts, note the rest). "One condition not met" ≠ "whole task not applicable."
- Each pattern must be independently checked — an `@primeng/x` import being absent does NOT mean `<p-x>` template tags or `pX` attribute usages are absent. Check every pattern separately.
- Component mapping must be DOC-DRIVEN: NEVER use hardcoded "never replace" lists or the starter table alone. **Material** — re-derive each target **pair** from the fetched docs per task and defer to what they say. **Optimus v1** — component imports/identifiers are a documented rename and `p-*`/`pi-*` selectors stay unchanged, but also run the OneCX-version compatibility checks. A rename is not sufficient evidence for provider, theme, Tailwind cascade, token scoping, or overlay parity.
- If docs are silent or contradictory about replacing a component → stop and ask, do not guess from memory or a prior run.
- PrimeFlex→Tailwind is DOC/CONFIG-DRIVEN (full mapping table + Tailwind tooling: `.claude/instructions/ui-migration-primeflex-to-tailwind.instructions.md`): map from the PrimeFlex utility set to Tailwind utilities using their corresponding values and measurements; re-derive the mapping per task; record source (MCP/node_modules/URL) for each. **Size-difference check (MANDATORY — both Material and Optimus targets):** do NOT map class-name-to-class-name. Read the ACTUAL PrimeFlex CSS value (e.g. from `node_modules/primeflex/primeflex.css`), pick the Tailwind utility whose value matches, or an **arbitrary value** (e.g. `m-[0.75rem]`, `gap-1.75rem`) when the Tailwind scale has no exact match. Record the measured source value + the chosen Tailwind value for every mapped utility — the result must be size-parity, not "close".
- When in doubt about applicability, default to must-have (safer than skipping).

## Validation Rules (SCOPED — do not run the whole suite)

- **Per change (default)**: for each changed file, find its `.spec.ts` in the same folder and run only that spec: `npx ng test --watch=false --browsers=ChromeHeadless --include=<path/to/that.spec.ts>` — **plus** `npm run build`. Never use `npm run test` for this: it can be a no-op `echo`. A changed file with no spec is recorded as `no spec file for <path>`; do not run the full suite.
- **Do NOT run the full test suite in the normal per-task flow** — it halts/destabilizes the system and is slow.
- **Full suite runs ONLY at**: (a) Gate 1 baseline, (b) phase-gate validations (A→B), (c) final verification (end of Phase B).
- **Final verification also runs the browser check** in `.claude/instructions/ui-migration-browser-check.instructions.md` after build, lint and test are green, and reports its URLs to the developer.
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
- Phase A → Phase B: requires explicit developer approval (mandatory gate). Phase B = cleanup + error-recovery (structural consolidation, leftover-import sweep, final verification). For a verified OneCX v8 app, retain dependencies required by installed peers/styles; do not remove them based on the app-owned rename alone. At final verification, list exact future OneCX v9 / Angular 22 removals and conditions in the cleanup handoff.

## No-Defer Rule

- You CANNOT defer any sub-step to a later phase or a future invocation.
- If a task needs a package not yet installed (e.g. `@angular/material-icons` for icon ligatures, or the Optimus package): confirm the target major matches `@angular/core` (or the installation page's peer range), resolve an explicit stable version (`npm view <pkg>@<major> dist-tags` — never `@latest`), and install it in the same invocation. Installing a design-system package is an architectural decision — get developer confirmation first.
- Do not remove `primeng`/`primeflex`/`primeicons` solely because app code was renamed. Check actual OneCX peer dependencies, CSS imports and remaining source/runtime references; retain what the installed stack needs. At completion, record conditional v9 removals with evidence in the cleanup handoff.

## Error Handling

- Errors during execution are YOUR job to fix in the SAME invocation — never defer.
- Error retry limit: attempt up to 3 different approaches per error — after 3 failures, escalate to the developer with the full attempt log.
- Log-based debugging: capture the **last 20 lines** of error output — never summarize, copy the actual output.
- Before declaring an import/API unresolvable: (1) `grep -r "Symbol" node_modules/<target-package>/ -l` (Material: `@angular/material`/`@angular/cdk`; Optimus: the Optimus package), (2) read the target package's `.d.ts`, (3) check the package's public API / CHANGELOG (or the fetched docs for Optimus). Only after all return nothing → escalate to the user with full evidence.
- Never silently drop functionality to silence an error. If a behavioral difference (PrimeNG vs the target) cannot be resolved from docs, record it in the task's Edge Cases field.

## Verification Checklist (Before Marking [x])

- Before marking ANY task `[x]`, verify ALL sub-steps are complete.
- For component migration: old import removed (grep proof), new import added, template updated, API mapping matches, CSS updated where needed, no duplicate module/provider in bootstrap.
- For PrimeFlex→Tailwind: old utility classes replaced (grep proof), Tailwind utilities present with matching measurements, no leftover PrimeFlex classes in scope.
- After migration, run `grep -r` for the OLD import/tag/class across the entire codebase — if any remain, the task is NOT complete.
- Provider/module deduplication: if a target module/provider belongs in the root bootstrap, keep it ONLY there — remove it from the component.
- If ANY sub-step is unchecked → the task stays `[ ]` — note which parts are incomplete.
- Compare changes against a working target example in the repo if one exists (e.g. an already-migrated component).
- For Optimus, compare against the version-matched working app and record OneCX/Optimus/Tailwind majors. The Angular 22 / OneCX v9 app is a visual and cleanup reference, not proof its providers or stylesheet pipeline can be copied into v8.
- Require a final “v9 cleanup handoff” listing each temporary adapter, prefix, plugin/config, legacy CSS/dependency and federation entry as remove/replace/retain, with evidence/condition. A generic “remove PrimeNG at v9” note is insufficient.
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

- ASK the user for: gate failures, ambiguous **target mapping** (Material: a candidate **pair** — two plausible target components for one PrimeNG one; Optimus: a rename that drifts from the fetched page, or whether a file is app-owned vs `pre_loaders`/OneCX-v8 which stays on `primeng`), behavioral differences with no clean equivalent, phase gates, package installs, risky adaptations, and **the target-library choice on a fresh start** (Material vs Optimus UI — never default).
- DO NOT ask for: routine searches, applicability checks, next-task routing, whether to read docs.
- If docs are ambiguous: stop, document the ambiguity, ask ONE concise question with the concrete candidates and the repo grep evidence.
- Only the orchestrator routes to other agents — agents never call each other directly.
