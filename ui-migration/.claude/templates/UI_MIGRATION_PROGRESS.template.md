# UI Migration Progress: PrimeNG → Angular Material

Date Started: **[YYYY-MM-DD]**
Repository: **[path/to/repo]**
Git Branch: **[branch name]**
Status: **[Gate 1 (baseline) / Gate 2 (branch) / Gate 3 (MCP) / Planning / In Progress (Phase A) / Phase B (cleanup) / Completed]**
Active Data File: **`.claude/instructions/ui-migration-prime-to-material.instructions.md`**
Active Rules File: **`.claude/instructions/ui-migration-rules.instructions.md`**
Migration identifier: **prime-to-material**

---

## Initialization Gates (HALT on any failure — fixed order)

| Gate | Check | Status | Evidence |
|------|-------|--------|----------|
| Gate 1 — Baseline: `npm run build` | `[ ]` | | |
| Gate 1 — Baseline: `npm run lint` | `[ ]` | record warning count | |
| Gate 1 — Baseline: `npm run test` | `[ ]` | record coverage % | |
| Gate 2 — Branch | `[ ]` | NOT main/master/dev/develop | |
| Gate 3 — MCP: PrimeNG MCP configured | `[ ]` | | |
| Gate 3 — MCP: Material-side MCP configured | `[ ]` | | |

**Baseline Records (all three must pass to unlock planning):**
- npm run build: [pass/fail — if fail: HALTED]
- npm run lint: [pass/fail — warnings: N (baseline frozen)]
- npm run test: [pass/fail — coverage: X%]
- Lint warning baseline: [N warnings — frozen for all comparisons]
- Test coverage baseline: [X%]

---

## Version Reality Check

| Package | Installed | Notes |
|---------|-----------|-------|
| `@angular/core` | [X] | current framework major |
| `@angular/material` | [X] | major MUST equal core major; note if already installed |
| `@angular/cdk` | [X] | same major |
| `primeng` | [X] | source library — **KEPT** (required by OneCX v8 libraries; usage → Angular Material in Phase A) |
| `primeflex` | [X] | layout utilities → Tailwind in Phase A — **KEPT** in `package.json` (required by OneCX v8) |
| `primeicons` | [X] | icon font → Material icons in Phase A — **KEPT** in `package.json` (required by OneCX v8) |
| Tailwind CSS | [present / missing] | PrimeFlex→Tailwind target; if missing, configure first |

**Target Material major = [X]** (must equal `@angular/core` major). Note any install requirement (a design-system install requires developer confirmation).

---

## Documentation Sources

- **PrimeNG MCP** (`mcp__primeng__*`) — OLD-side API audit. Verify `mcp__primeng__version` matches the installed major before trusting data.
- **Material-side MCP**: `mcp__angular-cli__*` (docs + build/devserver validation) and `mcp__ngm-dev-blocks__*` (generate the new Material+Tailwind blocks).
- **Fallback:** `node_modules/primeng/`, `node_modules/@angular/material/`, `v{N}.primeng.org`, `material.angular.dev`, `tailwindcss.com`.
- Every component AND PrimeFlex→Tailwind decision MUST record which source (MCP / node_modules / URL) was used.

**Existing Material references in this repo** (already-migrated examples to learn from): [list or "none"]

---

## Discovery & Inventory (planner — full-repo grep)

> One task per PrimeNG component family + one task per PrimeFlex→Tailwind surface.
> Planner grep evidence below is **pre-collected** for planning; the executor re-runs it (Step 3) and records authoritative counts.

### PrimeNG component inventory

| # | PrimeNG component | OLD patterns grepped | Material candidate(s) | Ambiguous? | Task |
|---|-------------------|----------------------|-----------------------|------------|------|
| 1 | [component] | [import / tag / attr / service] | [candidate] | [yes/no] | [A.N] |

### PrimeFlex→Tailwind surface inventory

| # | PrimeFlex surface | OLD classes grepped | Tailwind mapping | Task |
|---|-------------------|---------------------|------------------|------|
| 1 | [layout/spacing/grid] | [classes + files] | [mapping] | [A.N] |

**Applicability note:** ALL patterns 0 hits (excluding `pre_loaders/**`) → `[-]`. ANY pattern with hits → task applies.

---

## Phase A: Pre-Removal Code Changes (component swaps + PrimeFlex→Tailwind)

> Execute ONE task per invocation. Markers: `[ ]` not started, `[x]` completed, `[-]` not applicable.
> Each task = full-repo grep for the OLD patterns → understand the element's use-case + relevant CSS → transform (max similarity) → **scoped validate** → 8 evidence fields.
> **Scoped validation** (per change): test for the specific changed file(s) only (`npm run test -- --include='**/<changed>.spec.ts'` or `--config`/`ng test --include=`) **plus** `npm run build`. Do NOT run the full suite here.

### Task Template (COPY for each task)

```
**[A.N]. [Task Name — component family or PrimeFlex→Tailwind surface]**
- [ ] Status: [ ] not started | [x] completed | [-] not applicable
- Source pages: [MCP tool calls / URLs actually fetched for this pair]
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
  * npm run build: [must pass]
  * scoped tests: [must pass]
  * npm run lint: [must pass — 0 errors, 0 NEW warnings vs baseline]
- Final outcome: [success only after all pass]
- Remaining issues: [list and how resolved]

> **⚠️ DEFERRED PrimeNG REMOVAL — notify the developer here:**
> `primeng`, `primeflex`, and `primeicons` have been **intentionally left in `package.json`** — they are NOT removed during this migration. The OneCX v8 libraries still depend on PrimeNG, so removing them now would break the build. When you upgrade Angular to **22** (the OneCX **v9** upgrade), add the v9 libraries — they are Angular Material compatible — and **then** you can remove `primeng` / `primeflex` / `primeicons`.
> - [ ] Developer notified of the deferred PrimeNG removal (Angular 22 / OneCX v9 upgrade)

---

## Phase B: Cleanup + Error Recovery (developer approval REQUIRED)

> Phase B is cleanup + error recovery. It is triggered after ALL Phase A tasks are `[x]`/`[-]`.
> The orchestrator presents a summary and asks for explicit developer approval before entering it.
> **No package removal here** — `primeng` / `primeflex` / `primeicons` stay in `package.json` (required by the OneCX v8 libraries).

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

- [ ] Zero `@primeng/*` imports in in-scope files (whole-repo grep, `pre_loaders/**` excluded)
- [ ] Zero `<p-...>` tags in in-scope templates
- [ ] Zero `pButton`/`pInputText`/`pTooltip`/`[pTooltip]` attribute usages
- [ ] No `ConfirmationService`/`MessageService`/`PrimeIcons`/`providePrimeNG`/`primeng/api`/`@primeuix/*` in scope
- [ ] Zero PrimeFlex utility classes (`.col-*`, `.g-*`, `.m-*`, `.p-*`, `.row`, `.flex`, ...) on app markup — all replaced by Tailwind
- [ ] Tailwind `content` config covers edited templates; utilities built
- [ ] `primeng`/`primeflex`/`primeicons` **still present** in `package.json` (intentionally kept — required by the OneCX v8 libraries; removal happens at the OneCX v9 / Angular 22 upgrade, not now)
- [ ] Module-federation shared lists reflect final `@angular/material` + `@angular/cdk` (verify, don't hand-edit if auto-derived)
- [ ] `npm run build` → `npm run lint` → `npm run test` all green (**full** output read)
- [ ] Devserver visual pass reported to the developer

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

| Timestamp | Phase | Error (last 20 lines) | Root Cause | Fix Applied | Result |
|-----------|-------|-----------------------|------------|-------------|--------|
| [time] | [A/B] | [error output] | [cause] | [fix] | [resolved/pending] |

---

## Decision Log

<!-- Track all non-trivial choices made during migration -->

| Decision | Rationale | Alternatives Considered | Date |
|----------|-----------|------------------------|------|
| [what] | [why] | [other options] | [when] |

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
