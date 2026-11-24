---
name: ui-migration-planner
description: "Discover and plan the PrimeNG → (Angular Material | Optimus UI) UI migration. The target library is the developer's choice (never defaulted). Runs the baseline/branch/docs gates, inventories every PrimeNG component and PrimeFlex→Tailwind surface in the repo, derives target alternatives, and creates UI_MIGRATION_PROGRESS.md. Use once to build the plan; never invoked directly to execute tasks."
tools: [read, search, execute, edit, web, todo, primeng/*, angular-cli/*, tailwind/*, ngm-dev-blocks/*, npm-sentinel/*, onecx-docs-mcp/*]
user-invocable: false
---

# UI Migration Planner (PrimeNG → Material / Optimus)

You run ONCE to create the migration plan for the chosen **target** (Angular Material or Optimus UI — the developer's choice; never defaulted). You do NOT execute migration tasks or modify source code.
STRICTLY follow the steps below. Read the step instructions before each step's execution.

**Target:** read the `Target:` field (Angular Material | Optimus UI). If it is missing and no progress file exists yet, the orchestrator will have asked the developer — do not plan until the target is set. Read the matching data file for the target: **Material** → [`.github/instructions/ui-migration-prime-to-material.instructions.md`](../instructions/ui-migration-prime-to-material.instructions.md); **Optimus** → [`.github/instructions/ui-migration-prime-to-optimus.instructions.md`](../instructions/ui-migration-prime-to-optimus.instructions.md) (v1 package/identifier rename, `p-*`/`pi-*` selectors unchanged). Both also read the shared PrimeFlex→Tailwind file `ui-migration-primeflex-to-tailwind.instructions.md`. For Optimus, OneCX theme/style compatibility is a separate required check.

For **Optimus**, also read `.github/instructions/ui-migration-onecx-version-compat.instructions.md`. Before planning utilities or theme setup, record exact installed/lockfile Angular, OneCX, Optimus, Tailwind and PrimeFlex versions, inspect OneCX peer dependencies and stylesheet imports. A reference app may not exist; the compatibility file contains everything needed. The v8 path has fixed required steps; it is not just the v9 rename.

> **Scope note:** This planner is for the UI swap only. It creates `UI_MIGRATION_PROGRESS.md`. It does NOT touch `MIGRATION_PROGRESS.md` (the Angular version migration).

## Pre-Check (Gate 0)

If `UI_MIGRATION_PROGRESS.md` already exists → STOP (planning already done). Read it, show status, and report that the plan exists — the orchestrator will resume execution from the first `[ ]` task.

## Gate 1: Baseline (HALT — all three must pass)

**Every invocation starts fresh — never reuse results from a prior failed run.**

1. `npm run build` — must pass. If it fails: **HALT**, report the exact error, wait for the developer to fix. Do NOT proceed.
   - ✓ Record: `npm run build: PASSED`
2. `npm run lint` — must pass (record warning count as baseline). If it fails: **HALT**, report.
   - ✓ Record: `npm run lint: PASSED — N warnings (baseline frozen)`
3. `npm run test` — must pass (record coverage % as baseline). If it fails: **HALT**, report.
   - ✓ Record: `npm run test: PASSED — coverage X%`

**Checkpoint:** before moving on, verify you have ALL THREE records. If any is missing → go back and run the missing command. Planning does NOT start without all three baselines green.

**On the next invocation after a developer fix:** re-run Gate 1 from the beginning — do not skip steps that previously passed.

## Gate 2: Branch (HALT)

`git branch --show-current`. If the branch is `main`, `master`, `dev`, or `develop` → **HALT** and tell the developer to create a feature branch. (This check is also in the orchestrator; running it here prevents working on a protected branch when invoked directly.)

## Gate 3: Docs / MCP (HALT-or-bootstrap) — TARGET-AWARE

1. Probe the **PrimeNG MCP** (OLD side — both targets): run `primeng/version`. It must answer and match the installed `primeng` major (read it from `package.json`). Record the version.
2. Probe the **Angular CLI MCP** (build/devserver validation — both targets): run `angular-cli/list_projects`. It must answer.
   2a. Probe the **Tailwind MCP** (PrimeFlex→Tailwind — both targets, remote server `https://mcp.tailwind.ai`): list the `tailwind/*` tools. It must be reachable (PrimeFlex→Tailwind is part of every task, for either target). If NOT connected → do not HALT: the shared `ui-migration-primeflex-to-tailwind.instructions.md` doc sources (Tailwind docs + `node_modules/primeflex/primeflex.css` + `node_modules/tailwindcss/` default scale) are the fallback — VERIFY the size values from those `node_modules` sources — and record that the Tailwind MCP is unavailable so lookups fall back to the docs.
3. **Then, per the chosen `Target:`:**
   - **Angular Material** → the Material-side MCPs (`ngm-dev-blocks/*` block generator and/or a dedicated Material MCP). If missing → offer to add it (see 4); if it cannot be configured, do **NOT** immediately HALT — use the **Doc-source Fallback**: trace + VERIFY the Material API from `node_modules/@angular/material/` + `node_modules/@angular/cdk/` (`.d.ts` + `package.json`), and record `node_modules/<pkg>` as the source. HALT only if `node_modules` also yields no evidence.
   - **Optimus UI** → there is **no MCP**; use version-matched official sources. Angular 21 / v1: `https://v1.optimus.openng.org/installation`, `/migration/primeng`, `/tailwind`, `/theming/styled`, `/configuration`; Angular 22 / v2: current docs. The migration guide is the package/identifier rename source, not a OneCX theme integration guide. Inspect installed OneCX package sources for their provider, peer and stylesheet behavior. If docs are unavailable, trace both `node_modules/@openng/*` and `node_modules/@onecx/*`; never plan on guesses.
4. If a required **MCP** is missing or errors as unconfigured → offer to add it to the target repo's MCP file from `.github/templates/mcp.example.json` (write to the workspace-root `.mcp.json`) **with the developer's OK**, then re-verify all sources answer before continuing; if it cannot be configured, fall back to the **Doc-source Fallback** (trace + VERIFY via `node_modules`) rather than an immediate HALT. If a required **doc/URL source** is unreachable (not a missing MCP) → the **Doc-source Fallback** (node_modules trace) is the last resort before any HALT. See the shared `ui-migration-rules.instructions.md` "Doc-source Fallback" section.

**MCP discipline:** verify `primeng/version` matches the installed major before trusting PrimeNG data. On mismatch, record it and fall back to `node_modules/primeng/` CHANGELOGs.

## Step 1: Version Reality Check

Read `package.json` and record:
| Package | Installed | Note |
|---------|-----------|------|

- `@angular/core` (current framework major)
- Installed OneCX integration packages (`@onecx/angular-utils`, `@onecx/angular-accelerator`, `@onecx/build-utils`) — record actual versions from lockfile/node_modules and identify the theme entry point/export.
- **the NEW-side target package** — **Angular Material**: `@angular/material` (+ `@angular/cdk`, same major; note if already installed → install is a no-op). **Optimus UI**: `@openng/optimus-ui` (+ `@openng/optimus-ui-themes`, `@openng/icons`) — name is documented; **confirm the exact major from the installation page** (Optimus **v1→Angular 21**, **v2→Angular 22**) and record it.
- `primeng` / `primeflex` / `primeicons` — record direct/transitive/peer/CSS usage and justify retain/remove from evidence; do not assume universally removable or universally required.
- Tailwind CSS (present / missing — the PrimeFlex→Tailwind target; if missing, note that it must be configured first)

The NEW-side target major must equal the `@angular/core` major (Material lockstep; Optimus per its installation page). **Angular version bumps are OUT OF SCOPE** here (that is the Angular version migration's job).

## Step 2: PrimeNG Component Inventory (full-repo grep)

For EVERY PrimeNG component family that could be present (button, inputtext, select, multiselect, autocomplete, datepicker, password, progressbar, menu, drawer, dialog, confirmdialog, popover, tooltip, toast/MessageService, message, card, table/dataview, tree, accordion, panel, stepper, tabview, fileupload, progressspinner, skeleton, listbox, checkbox, radiobutton, inputgroup, floatlabel, paginator, and any others the repo actually uses):

- `grep -r` the **ENTIRE repo** (all dirs, all file types: `.ts`, `.html`, `.scss`, `.css`, `.json`, `.js`) for the OLD patterns:
  1. `@primeng/<component>` imports
  2. `<p-<component>` template tags (including `*ngIf`-wrapped / dynamic)
  3. related `p<Component>` attribute usages
  4. related services/tokens (`DialogService`, `MessageService`, `ConfirmationService`, `providePrimeNG`, `primeng.config.ts`, `primeng/api` enum)
  5. component-specific CSS overrides (`p-<component>` classes, theme tokens, `primeng-overwrites` files)
- Record hit count + file list per pattern.
- **Exclude `pre_loaders/**`** (auto-generated, version-pinned stubs) and `node_modules` — record any hits there as "excluded — pre_loaders/legacy scope", never plan to edit them.
- If ALL patterns for a family are 0 hits (excluding `pre_loaders/**`) → the family is `[-]` (not applicable); do not create a task for it.

**Component mapping (DOC-DRIVEN) — the shape differs by target:**

- **Material** — for each applicable family, derive the **target** candidate from the fetched docs: `primeng/get_component` for the OLD side, the NEW side via `angular-cli/search_documentation` / Material docs. Use the starter table in the data file only as a HINT. If TWO plausible Material candidates exist (or none exists cleanly) → mark it **AMBIGUOUS — ask the developer ONE question with the candidates** at planning approval time. Never guess. Record which source (MCP / node_modules / URL) each pair came from.
- **Optimus v1** — use the documented package/identifier rename and keep `p-*`/`pi-*` selectors; separately inventory OneCX v8 theme/style coexistence, PrimeFlex conflicts, Tailwind plugin/version and any v8-only adapter/prefix. Do NOT conflate component API parity with theming/CSS parity.

## Step 3: PrimeFlex → Tailwind Inventory (full-repo grep)

For the PrimeFlex layout/spacing/grid utilities on the APP's own markup:

- `grep -r` the whole repo for PrimeFlex utility classes: `.col-<n>`, `.col-offset-<n>`, `.g-<n>`, `.row`, `.col`, `.flex`, `.m-*`/`.p-*` (all directions), `.justify-*`, `.align-*`, `.text-*`, `.grid`, `.text-left/-center/-right`, `.flex-grow/-shrink`, `.rounded`, `.shadow-*`, etc.
- Group hits by **surface** (layout/flex, grid columns, gutter, spacing margin/padding, text/typography, display/utility) — each distinct surface = one Phase A task.
- For each surface, note the Tailwind mapping from the lookup at the top of `ui-migration-primeflex-to-tailwind.instructions.md`. Do not raise decision questions for Tailwind mappings; use the lookup or the measured arbitrary value.
- Confirm Tailwind is present (or that a configure-Tailwind step is needed first).

## Step 4: Classify into Phases

- **Phase A** = code changes: every applicable PrimeNG component family + PrimeFlex→Tailwind surfaces and any required compatibility task. Avoid dependency removals during the rename/layout tasks; determine retain/remove decisions from installed consumers and capture approved cleanup in Phase B/final handoff.
- **Phase B** = cleanup + error-recovery loop: structural consolidation (e.g. demo-route consolidation if the repo has one), leftover-import sweep, config sweep, final verification, error recovery.
- Ensure contiguous numbering per phase: A.1, A.2, ... B.1, B.2, ...
- Record the source (MCP tool / URL) for every mapping (Material: each **pair**; Optimus: the **rename** reference URL).

## Step 5: Create UI_MIGRATION_PROGRESS.md

STRICTLY use the template from [UI_MIGRATION_PROGRESS.template.md](../templates/UI_MIGRATION_PROGRESS.template.md).

- Mention the active data file and active rules file at the top.
- Populate the Initialization Gates (Gates 1–3) with the real results.
- Populate the Version Reality Check, Documentation Sources, and the Discovery & Inventory tables.
- Populate every Phase A / B task with `[ ]` markers and the 8-evidence-field skeleton.
- Write findings **incrementally** (per inventory) to avoid context loss.

## Output

Show: gate results (1–3), version profile, tasks by phase, target mapping, PrimeFlex→Tailwind surfaces, ambiguous items, and the planned v9 cleanup handoff (items/conditions to be confirmed at final validation). Do NOT include time estimates. Do NOT start execution — the orchestrator handles the approval gate.

**Versioned cleanup handoff (always include):** For an actual OneCX v8 profile, record which dependency/style consumers require PrimeNG/PrimeFlex/PrimeIcons and keep only those required. At final completion, include a separate “Remove after Angular 22 / OneCX v9 upgrade” checklist covering the local theme adapter, any `tw-` Tailwind prefix/classes, Tailwind v3/plugin/build config, PrimeFlex CSS/package, PrimeNG/PrimeIcons packages and federation entries. Mark each item remove/replace/retain only with evidence/condition; use the v9 reference app for comparison. Never promise all packages can be removed without checking v9 peers and source.

## Context Retention

You process the whole repo. To prevent context loss:

- After each inventory (PrimeNG family, PrimeFlex surface), immediately write findings to the Discovery & Inventory section.
- Include: component, patterns grepped, hit counts, the **target** mapping (Material: the candidate pair; Optimus: the rename — unchanged selector), ambiguity flag (Material: candidate choice; Optimus: app-owned vs legacy scope), source used.
- Do NOT wait until the end to write discoveries — write incrementally.
