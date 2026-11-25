# UI Migration Progress: PrimeNG → [Angular Material | Optimus UI]

Date Started: **[YYYY-MM-DD]**
Repository: **[path/to/repo]**
Git Branch: **[branch name]**
Target: **[Angular Material | Optimus UI]** — chosen by the developer, NEVER defaulted. On resume, use this value (do not re-ask or switch — a different target starts a fresh progress file).
Status: **[Gate 1 (baseline) / Gate 2 (branch) / Gate 3 (docs/MCP) / Planning / In Progress (Phase A) / Phase B (cleanup) / Completed]**
Active Data File: **[Material → `.claude/instructions/ui-migration-prime-to-material.instructions.md` | Optimus → `.claude/instructions/ui-migration-prime-to-optimus.instructions.md`]** (set to match `Target:`)
Active Rules File: **`.claude/instructions/ui-migration-rules.instructions.md`** (shared, target-agnostic)
Active Compatibility File: **[Optimus → `.claude/instructions/ui-migration-onecx-version-compat.instructions.md` | Material → N/A unless OneCX v8/Tailwind coexistence requires it]**
Migration identifier: **[prime-to-material | prime-to-optimus]** (set to match `Target:`)
Working visual reference: **[path + installed Angular / OneCX / target / Tailwind versions, or N/A with reason]**

---

## Initialization Gates (HALT on any failure — fixed order)

| Gate                                                                       | Check | Status                                                      | Evidence |
| -------------------------------------------------------------------------- | ----- | ----------------------------------------------------------- | -------- |
| Gate 1 — Baseline: `npm run build`                                         | `[ ]` |                                                             |          |
| Gate 1 — Baseline: `npm run lint`                                          | `[ ]` | record warning count                                        |          |
| Gate 1 — Baseline: `npm run test`                                          | `[ ]` | record coverage %                                           |          |
| Gate 2 — Branch                                                            | `[ ]` | NOT main/master/dev/develop                                 |          |
| Gate 3 — Docs/MCP: PrimeNG MCP + Angular CLI MCP configured (both targets) | `[ ]` |                                                             |          |
| Gate 3 — (Material) `ngm-dev-blocks` + Material MCP configured             | `[ ]` | Material target only                                        |          |
| Gate 3 — (Optimus) the two Optimus doc URLs reachable via WebFetch         | `[ ]` | Optimus target only — no MCP; internal access may be needed |          |

**Baseline Records (all three must pass to unlock planning):**

- npm run build: [pass/fail — if fail: HALTED]
- npm run lint: [pass/fail — warnings: N (baseline frozen)]
- npm run test: [pass/fail — coverage: X%]
- Lint warning baseline: [N warnings — frozen for all comparisons]
- Test coverage baseline: [X%]

---

## Version Reality Check

| Package                     | Installed           | Notes                                                                                                                                                                                        |
| --------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@angular/core`             | [X]                 | current framework major                                                                                                                                                                      |
| Installed OneCX integration | [X]                 | Record actual `@onecx/angular-utils`, `@onecx/angular-accelerator`, `@onecx/build-utils` versions and theme integration entry points.                                                        |
| `<target package>`          | [X]                 | Material: `@angular/material` (major MUST equal core major); Optimus: `@openng/optimus-ui` (major from the installation page — **v1→Angular 21, v2→Angular 22**). Note if already installed. |
| `<target companion>`        | [X]                 | Material: `@angular/cdk` (same major); Optimus: `@openng/optimus-ui-themes` + `@openng/icons` (theme preset + icons)                                                                         |
| `primeng`                   | [X]                 | Record direct/transitive/peer/runtime usage and decision; do not assume globally required or removable.                                                                                      |
| `primeflex`                 | [X]                 | Record CSS imports and v8 library class usage; app templates may migrate while the library still needs it.                                                                                   |
| `primeicons`                | [X]                 | Record actual icon/font/package consumers and decision.                                                                                                                                      |
| Tailwind CSS                | [present / missing] | PrimeFlex→Tailwind target (BOTH Material and Optimus); if missing, configure first                                                                                                           |
| Tailwind utility namespace  | [X]                 | Record prefix decision (none / temporary `tw-` / other), collision evidence and cleanup plan.                                                                                                |

**Target version = [X]** — Material: `@angular/material` major MUST equal `@angular/core` major. Optimus: `@openng/optimus-ui` major per the installation page (v1→Angular 21, v2→Angular 22) — pin the major. Note any install requirement (a design-system install requires developer confirmation).

---

## Documentation Sources

- **PrimeNG MCP** (`mcp__primeng__*`) — OLD-side API audit. Verify `mcp__primeng__version` matches the installed major before trusting data. (Both targets.)
- **Angular CLI MCP** (`mcp__angular-cli__*`) — NEW-side **build/devserver validation** + general Angular docs. (Both targets.)
- **Target NEW-side source:**
  - **Material** → `mcp__ngm-dev-blocks__*` (generate the new Material+Tailwind blocks) + Material docs.
  - **Optimus UI** → **no MCP** — use version-matched docs. Angular 21/v1: `https://v1.optimus.openng.org/installation`, `/migration/primeng`, `/tailwind`, `/theming/styled`, `/configuration`, `/migration/update`. Angular 22/v2: current Optimus docs. The migration guide documents the rename; also verify OneCX theme integration from installed package sources. Component API parity does not prove theme/CSS parity.
- **Doc-source fallback (both targets — MCP/WebFetch → node_modules → then HALT):** when the primary source (MCP or doc URL) is unreachable/policy-blocked/returns nothing, do **NOT** immediately HALT or guess — trace the installed `node_modules/` as the **LAST-RESORT evidence source** and **VERIFY** it in place of the web (the installed `.d.ts` = the real export names + inputs/outputs, `package.json` subpath exports + major, `CHANGELOG`, and CSS/asset files). OLD side: `node_modules/primeng/`, `node_modules/primeflex/primeflex.css`, `node_modules/primeicons/primeicons.css`. NEW side — Material: `node_modules/@angular/material/<component>/…d.ts` + `node_modules/@angular/cdk/`; Optimus: `node_modules/@openng/optimus-ui/` + `@openng/icons/openng-icons.css`. Tailwind size values: `node_modules/tailwindcss/` default scale. **The scoped `npm run build` is the final verification** (a rename/choice that RESOLVES and BUILDS proves the traced evidence was correct). **HALT only if `node_modules` also yields no evidence** (target package not installed / `.d.ts` absent).
- Every component AND PrimeFlex→Tailwind decision MUST record which source (MCP / URL / **node_modules**) was used.

**Existing target references in this repo** (already-migrated examples to learn from): [list or "none"]

---

## Discovery & Inventory (planner — full-repo grep)

> One task per PrimeNG component family + one task per PrimeFlex→Tailwind surface.
> Planner grep evidence below is **pre-collected** for planning; the executor re-runs it (Step 3) and records authoritative counts.

### PrimeNG component inventory

| #   | PrimeNG component | OLD patterns grepped            | **Target** mapping (Material: candidate **pair**; Optimus: the **rename** — unchanged selector) | Ambiguous? | Task  |
| --- | ----------------- | ------------------------------- | ----------------------------------------------------------------------------------------------- | ---------- | ----- |
| 1   | [component]       | [import / tag / attr / service] | [pair (Material) / rename (Optimus)]                                                            | [yes/no]   | [A.N] |

### PrimeFlex→Tailwind surface inventory

| #   | PrimeFlex surface     | OLD classes grepped | Tailwind mapping | Size-parity? (measured value → chosen value; arbitrary value where scale differs) | Task  |
| --- | --------------------- | ------------------- | ---------------- | --------------------------------------------------------------------------------- | ----- |
| 1   | [layout/spacing/grid] | [classes + files]   | [mapping]        | [e.g. `.g-4` = `2.2rem` → `gap-[1.1rem]`]                                         | [A.N] |

**Applicability note:** ALL patterns 0 hits (excluding `pre_loaders/**`) → `[-]`. ANY pattern with hits → task applies.
**Size-difference check (MANDATORY):** do NOT map class-name-to-class-name — the PrimeFlex scale ≠ the Tailwind scale. Record the measured PrimeFlex value and the chosen Tailwind value (arbitrary value where no exact step exists) for every utility that carries a measurement.

---

## Phase A: Code Changes (Material: component swaps; Optimus: package/identifier rename) + PrimeFlex→Tailwind

> Execute ONE task per invocation. Markers: `[ ]` not started, `[x]` completed, `[-]` not applicable.
> Each task = full-repo grep for the OLD patterns → understand the element's use-case + relevant CSS → **Material: transform (max similarity)** / **Optimus: rename the import/identifier/asset and VERIFY the `<p-...>`/`pi-*` selector is untouched** → **scoped validate** → 8 evidence fields.
> **Scoped validation** (per change): run the `.spec.ts` that sits next to each changed file with `npx ng test --watch=false --browsers=ChromeHeadless --include=<spec path>` (not `npm run test`, which may be a no-op) **plus** `npm run build`. No spec next to a file → record `no spec file for <path>`. Do NOT run the full suite here.

### Task Template (COPY for each task)

```
**[A.N]. [Task Name — component family or PrimeFlex→Tailwind surface]**
- [ ] Status: [ ] not started | [x] completed | [-] not applicable
- Source pages: [MCP tool calls / URLs / `node_modules/<pkg>` (last-resort: the `.d.ts`/`package.json`/asset traced + verified when the MCP/URL was unreachable) used for this task (Material: the chosen pair; Optimus: the rename reference — there is no pair)]
- Applicability: must-have | nice-to-have | not applicable
- Repository evidence: [grep command + hit count + file list, per pattern — one line per pattern]
- Sub-steps executed: [each pattern / mapping — done | not-applicable | not-found]
- Files changed: [exact file list]
- Validation (scoped, in order):
  * npm run build: [✓ pass or full error output]
  * scoped test (changed files only): [✓ pass or full error output]
  * npm run lint: [✓ 0 errors, 0 NEW warnings vs baseline or full error output]
- Final outcome: success (after N fix rounds) | not applicable | escalated
- Edge cases: [behavioral differences, approximate mappings (e.g. grid gutter), out-of-scope hits (pre_loaders), decisions]
```

### Phase A Tasks (discovered by planner)

> PLANNER NOTE: List tasks below. Contiguous A.N numbering. One entry per component family and per PrimeFlex→Tailwind surface. No time estimates.

**[A.1]. [Task Name]**

- [ ] not started
- Source pages: [MCP/URL]
- Applicability: [TBD by executor]
- Repository evidence: [TBD by executor]
- Sub-steps executed: [TBD by executor]
- Files changed: [TBD]
- Validation: build [TBD] | scoped test [TBD] | lint [TBD]
- Final outcome: [TBD]
- Edge cases: [TBD]

**[A.N — last Phase A task]. End-of-Phase-A Build State Record**

- [ ] not started
- Source page: N/A — required by migration process
- **STRICT**: `npm run build` must pass and scoped tests for all Phase A changes must pass.
- Validation (MUST pass — fix failures before Phase B):
  - npm run build: [must pass]
  - scoped tests: [must pass]
  - npm run lint: [must pass — 0 errors, 0 NEW warnings vs baseline]
- Final outcome: [success only after all pass]
- Remaining issues: [list and how resolved]

> **Required final handoff — temporary v8 implementation cleanup:** record each temporary item and its remove/replace/retain condition based on the actual OneCX v9 package graph and working reference. Assess the local OneCX theme adapter/style hook, `tw-` prefix/classes, Tailwind v3/plugin/build configuration, PrimeFlex CSS/package, PrimeNG/PrimeIcons packages and federation entries. Do not promise all dependencies can be removed without checking peers, source, CSS imports, `npm ls`, build and Shell behavior.
>
> - [ ] Final “Remove after Angular 22 / OneCX v9 upgrade” checklist completed with evidence and shared with developer

---

## Phase B: Cleanup + Error Recovery (developer approval REQUIRED)

> Phase B is app-owned cleanup + error recovery. It is triggered after ALL Phase A tasks are `[x]`/`[-]`.
> The orchestrator presents a summary and asks for explicit developer approval before entering it.
> Do not remove dependencies without checking current OneCX peer/CSS/runtime consumers. Record conditional v9 removals in the final handoff.

**Orchestrator presents:**

- Files changed in Phase A: [git diff --stat]
- End-of-Phase-A build state: [pass/fail]

**Developer approval:**

- [ ] Approved to proceed with Phase B (cleanup)
- Approval recorded by: [Developer Name] on [Date]

### Phase B Tasks

**[B.1]. [Cleanup task — e.g. structural consolidation / route removal / leftover import sweep / module-federation config sweep]**

- [ ] not started
- Source page: N/A
- Repository evidence: [TBD by executor]
- Files changed: [TBD]
- Validation: build [TBD] | scoped test [TBD] | lint [TBD]
- Final outcome: [TBD]

### Error Recovery Loop (after all Phase B tasks)

- [ ] Rerun `npm run build` (should now pass)
- [ ] Rerun `npm run lint` (must pass — 0 errors, vs baseline warnings)
- [ ] Rerun `npm run test` **full suite** (must pass; coverage at or above baseline)
- [ ] For each transitional error recorded in Phase B: check if NOW fixed → update entry
- [ ] Remaining unfixed errors: document for manual fix, add to blockers

---

## Final Verification and Validation (run at/after Phase B)

> **Material and Optimus differ materially here.** Material is a component _swap_ — the old `<p-...>` tags and PrimeNG identifiers must disappear. Optimus is a _rename_ — the `<p-...>`/`pi-*` selectors **stay unchanged**; only the package/import/identifier/asset layer changes. Check the group that matches the active `Target:`.

**Shared (both targets):**

- [ ] Zero PrimeFlex utility classes (`.col-*`, `.g-*`, `.m-*`, `.p-*`, `.row`, `.flex`, ...) on app markup — all replaced by Tailwind
- [ ] **Tailwind size parity verified** — each replaced utility's measured value matches its PrimeFlex source (arbitrary values where the scale differs; see the size-difference rule)
- [ ] Tailwind `content` config covers edited templates; utilities built
- [ ] `primeng`/`primeflex`/`primeicons` decisions match installed peer dependencies, CSS imports and source/runtime usage; deferred v9 cleanup is documented with evidence
- [ ] `npm run build` → `npm run lint` → `npm run test` all green (**full** output read)
- [ ] Devserver visual pass reported to the developer
- [ ] Browser check done per `.claude/instructions/ui-migration-browser-check.instructions.md`: runner release tag + checksum OK, URLs, test page file and screenshot paths recorded (or `BLOCKED: <reason>`)

**Material only:**

- [ ] Zero `primeng/*` imports in in-scope files (whole-repo grep, `pre_loaders/**` excluded)
- [ ] Zero `<p-...>` tags in in-scope templates
- [ ] Zero `pButton`/`pInputText`/`pTooltip`/`[pTooltip]` directive-attribute usages
- [ ] No `ConfirmationService`/`MessageService`/`PrimeIcons`/`providePrimeNG`/`primeng/api`/`@primeuix/*` in scope
- [ ] Module-federation shared lists reflect `@angular/material` + `@angular/cdk` (verify, don't hand-edit if auto-derived)

**Optimus only:**

- [ ] The PrimeNG-facing surface is **still present and unchanged** in in-scope app-owned code: `<p-...>` component tags, `pButton`/`pTooltip`-style directive attributes, and `pi-*` icon classes (grep proves these were NOT renamed — the migration only touched the import/identifier/asset layer)
- [ ] Zero `primeng/*`, `primeicons`, and `@primeuix/*` imports in **app-owned** files (whole-app grep, `pre_loaders/**` AND the OneCX v8 libraries excluded — those legitimately stay on `primeng`)
- [ ] `provideOptimus` / `OptimusConfigType` in scope (not `providePrimeNG` / `PrimeNGConfigType`)
- [ ] Exactly one icon font stylesheet (`primeicons/primeicons.css`) is imported (no second `openng-icons.css`)
- [ ] Module-federation shared lists (app-owned) reflect `@openng/optimus-ui` (verify, don't hand-edit if auto-derived)

---

## Current Session Context

<!-- Updated continuously — helps resume across chat sessions -->

- Last executed step: [step name and outcome]
- Next planned step: [step name and dependencies]
- Open issues/blockers: [current blocking issues]
- Recent discoveries: [new information from documentation]

---

## Error Log

<!-- Captured during execution — each build/lint/test failure -->

| Timestamp | Phase | Error (last 20 lines) | Root Cause | Fix Applied | Result             |
| --------- | ----- | --------------------- | ---------- | ----------- | ------------------ |
| [time]    | [A/B] | [error output]        | [cause]    | [fix]       | [resolved/pending] |

---

## Decision Log

<!-- Track all non-trivial choices made during migration -->

| Decision | Rationale | Alternatives Considered | Date   |
| -------- | --------- | ----------------------- | ------ |
| [what]   | [why]     | [other options]         | [when] |

---

## Summary

**Start date:** [YYYY-MM-DD]
**End date:** [YYYY-MM-DD when complete]
**Total tasks:** [N] (Phase A: X, Phase B: Y)
**Completed:** [N]
**Skipped:** [N] (via Skip N by developer)

**Test coverage baseline:** [X%] → **Final coverage:** [X%]
**Lint warning baseline:** [N] → **Final lint warnings:** [N]

**Critical blockers:** [if any]

**Sign-off:** [Developer Name, Date]
