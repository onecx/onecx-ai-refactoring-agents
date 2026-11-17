# PrimeNG → Angular Material Migration Data

<!--
  This file contains ALL data for the PrimeNG → Angular Material UI swap.
  It is NOT auto-injected (no applyTo). ui-migration-executor and the
  /migrate-material command read it on-demand.
  Separate scope from the Angular version migration (MIGRATION_PROGRESS.md) —
  this migration is tracked in UI_MIGRATION_PROGRESS.md.
-->

## Scope

- **Source UI library**: PrimeNG (`p-*` template tags, `@primeng/*` imports, `providePrimeNG`, primeicons/primeflex)
- **Target UI library**: Angular Material (`mat-*` tags, `@angular/material/*` imports, `@angular/cdk`)
- **Progress file**: `UI_MIGRATION_PROGRESS.md` (workspace root) — same marker/evidence conventions as MIGRATION_PROGRESS.md
- **In scope**: root workspace application code (`src/`, styles, providers, specs)
- **OUT of scope (never modify)**: `pre_loaders/**` (auto-generated version-pinned stubs), `node_modules`, `@onecx/portal-layout-styles` (legacy v5 MFE styling intentionally retained per README)

## MCP Gate (Gate 3 — HALT-or-bootstrap)

Before Discovery or execution, confirm BOTH MCP servers are configured in the target repo. This is a hard gate (see `ui-migration-rules.instructions.md`).

| MCP | Tool prefix | Purpose | Config shape (see `templates/mcp.example.json`) |
|-----|-------------|---------|-----------------------------------------------|
| **PrimeNG MCP** | `mcp__primeng__*` | audit the OLD PrimeNG usage (component API, template syntax, behavior) | `npx -y @primeng/mcp` |
| **Material-side MCP (docs + validation)** | `mcp__angular-cli__*` | NEW-side Material API reference + build/devserver validation | `npx -y @angular/cli mcp` |
| **Material-side MCP (generator)** | `mcp__ngm-dev-blocks__*` | generate the new Angular Material + Tailwind blocks | `npx -y @ngm-dev/cli mcp <abs-project-dir>` |

**How to verify:** list the configured MCP tools (e.g. probe `mcp__primeng__version` and `mcp__angular-cli__list_projects`). If a server is missing or errors as unconfigured:
- **HALT** and tell the developer: "MCP gate failed — <server> is not configured."
- Offer to add it to the target repo's MCP file from `templates/mcp.example.json` (write to root `mcp.json` / `.mcp.json` / `.vscode/mcp.json` — whichever the target uses), **with the developer's OK**, then re-verify.
- Only proceed once BOTH servers answer.

> **Material-side identity is repo-dependent.** By default this is the Angular CLI MCP (Material has no separate MCP server). If the developer's stack ships a dedicated Angular Material MCP, Gate 3 validates that instead — record which one is authoritative in the progress file.

## Documentation Sources

### PrimeNG MCP
- **available**: true (`npx -y @primeng/mcp`)
- **key-tools**: `mcp__primeng__list`, `mcp__primeng__version`, `mcp__primeng__get_component` (`includeApi`, `includeExamples`), `mcp__primeng__search`, `mcp__primeng__get_example`, `mcp__primeng__validate_usage`
- **purpose**: audit the OLD PrimeNG usage — component API, template syntax, behavior. Verify `mcp__primeng__version` matches the installed PrimeNG major before trusting its data.
- **fallback-url**: `https://v{N}.primeng.org/` (version-pinned subdomains) and `node_modules/primeng/` source + CHANGELOG

### Angular CLI MCP
- **available**: true (`npx -y @angular/cli mcp`)
- **key-tools**:
  - `mcp__angular-cli__get_best_practices` (pass `workspacePath`) — read BEFORE writing any new Angular code
  - `mcp__angular-cli__search_documentation` (pass installed `@angular/core` major as `version`) — Angular.dev / Material **API reference** lookups
  - `mcp__angular-cli__list_projects` — project/test-framework discovery
  - `mcp__angular-cli__run_target` — build/lint/test execution
  - `mcp__angular-cli__devserver_start` / `devserver_wait_for_build` / `devserver_stop` — live build feedback + visual check for user-visible UI changes
- **purpose**: NEW-side API reference + build/devserver **validation** loops

### ngm-dev-blocks MCP (dedicated Angular Material generator)
- **available**: true (`npx -y @ngm-dev/cli mcp <abs-project-dir>`) — the official Angular Material MCP (ui.angular-material.dev/docs/mcp)
- **key-tools**:
  - `mcp__ngm-dev-blocks__get-all-block-names` — list which ready-made Material+Tailwind blocks exist (use at planning to see if a task's surface maps to a known block)
  - `mcp__ngm-dev-blocks__generate-angular-material-block` — **generate the new Angular Material + Tailwind block** for a component (use during execution to produce the replacement code)
  - `mcp__ngm-dev-blocks__setup-angular-material-blocks` — run once if `ngm-dev-cli.json` is missing in the project
- **purpose**: GENERATE the new Material+Tailwind code. Complementary to `angular-cli` (which does docs + validation) — the generator writes the code, `angular-cli`/`npm run build` validates it. **MCP is a starting point, not ground truth**: the executor still audits the old usage, adapts the generated block to the repo's specifics, and scoped-validates.

### Documentation Fallback Priority
1. MCP servers (PrimeNG MCP for the old side, Angular CLI MCP for the new side)
2. `node_modules/<package>/` source + CHANGELOG
3. Fallback URLs via WebFetch: `material.angular.dev` (match installed Material major), `v{N}.primeng.org` (match installed PrimeNG major)

Record for every component decision which source (MCP / node_modules / URL) was used.

## Target Versions

- `@angular/material`: major MUST equal the installed `@angular/core` major (currently **22** → `@angular/material@^22`)
- `@angular/cdk`: same major
- Resolve to explicit stable: `npm view @angular/material@<major> dist-tags` — **NEVER** `@latest`
- Install of `@angular/material`/`@angular/cdk` requires user confirmation first (executor Step 0 — installing a design system is an architectural decision, not a routine task step)
- Version bumps of Angular itself are OUT OF SCOPE — that belongs to the Angular version migration agents

## Component Pair Starter Table (EXAMPLES ONLY — not a decision list)

The Angular Material docs do not publish an official PrimeNG→Material mapping. The table below is a HINT from this repo's planning; each pair MUST be re-derived per task from fetched docs (executor Step 2) and recorded in the task evidence. If a PrimeNG component has no clean Material equivalent, or two plausible candidates exist → **ASK the user ONE question with the candidates** — never guess.

| PrimeNG | Angular Material candidate | Notes |
|---------|---------------------------|-------|
| `<p-button>` | `MatButton` / `MatIconButton` | severity/variant mapping differs (text/outlined/flat/stroked); check icon placement |
| `<p-badge>` | `MatBadge` | |
| `<p-tag>` | `MatBadge` or `MatChip` | **ambiguous — ask** |
| `<p-dialog>` | `MatDialog` + `MatDialogRef` | ref/data API differs from PrimeNG `DialogService` |
| `<p-table>` / `<p-dataView>` | `MatTable` + `MatTableDataSource` (+ `MatSort`/`MatPaginator` where applicable) | column defs differ; check sorting/filtering parity |
| `<p-select>` / `<p-dropdown>` | `MatSelect` | options via `MatOption` |
| `<p-multiselect>` | `MatSelect` with `multiple` | |
| `<p-checkbox>` | `MatCheckbox` | label moved out of input (PrimeNG 19+ dropped `[label]`) |
| `<p-radiobutton>` | `MatRadioButton` | |
| `<p-inputtext>` / inputs | `MatInput` inside `MatFormField` | `ngModel`/`formControl` binding replaces `p-model`/`[value]` |
| `<p-inputgroup>` / addon spans | `MatFormField` + `MatPrefix`/`MatSuffix` | |
| `<p-floatlabel>` | `MatFormField` + `MatLabel` (floating label is native) | |
| `MessageService` / `<p-messages>` | `MatSnackBar` | API differs (snackbar, not stackable messages) |
| `<p-confirmdialog>` / `ConfirmationService` | `MatDialog` confirm pattern | |
| `<p-toolbar>` | `MatToolbar` | |
| `<p-menu>` | `MatMenu` | |
| `<p-contextmenu>` | `MatMenu` in `Overlay` | |
| `<p-tooltip>` | `MatTooltip` | |
| `<p-card>` | plain markup + styles (no Material card primitive) | **verify — no direct equivalent** |
| `<p-paginator>` | `MatPaginator` | |
| `<p-calendar>` | `MatDatepicker` (+ `MatNativeDateModule`/adapters) | date model differs |
| `<p-accordion>` / `<p-panel>` | `MatExpansionPanel` | **verify behavior parity** |
| `<p-stepper>` | `MatStepper` | |
| `<p-tabview>` | `MatTabGroup` | |
| `<p-fileupload>` | `MatButton` + `<input type="file">` | no Material fileupload primitive |
| `<p-progressspinner>` | `MatProgressSpinner` | |
| `<p-progressbar>` | `MatProgressBar` | |
| `<p-skeleton>` | CSS shimmer (no Material primitive) | **no direct equivalent** |
| `<p-listbox>` | `MatSelectionList` | |
| `<p-tree>` | `MatTree` | |
| `primeicons` (pi-* classes) | `@angular/material-icons` (`mat-icon` ligatures) | icon NAMES differ (pi-* → material icon names) — check each icon used |
| `primeng/resources/*` CSS | Material theme (SCSS mixins or prebuilt theme) | global theming approach changes |
| `primeflex` layout classes (`.flex`, `.grid`, `.col-6`, `.g-*`, `.m-*`) | **Tailwind utilities** (see "PrimeFlex → Tailwind" section) | **value/measurement mapping — see section**; assume Tailwind is present (add/configure if missing) |

## Behavioral Differences (verify per component task, record in Edge Cases)

- **Theming**: PrimeNG CSS themes (theme-preset + `primeng/resources/*`) vs Material theme (SCSS mixins / prebuilt). Do not mix; document which Material theme the app adopts.
- **Form controls**: PrimeNG `p-model` / `[value]` / `[options]` → Material `ngModel` / `formControl` / `MatOption`. Form-control validation styling differs (`p-invalid` vs `NgForm` error classes).
- **Dialogs**: PrimeNG/OneCX dialog services (`DialogService`, `PortalDialogService`) vs `MatDialog` ref API. Check which dialog service the task's files actually use before mapping.
- **Toasts**: `MessageService` (stacked) vs `MatSnackBar` (single) — stacked-toast patterns need an adapter or a UI decision (ask).
- **Icons**: `primeicons` glyph names do not map 1:1 to Material ligatures — verify every `pi-*` icon in scope files against Material icon names.
- **Animations**: Material components require animations enabled — verify `provideAnimations()` (or `BrowserAnimationsModule`) is present in bootstrap before relying on transitions.

## PrimeFlex → Tailwind (value/measurement mapping)

PrimeFlex provides layout utilities that have **direct Tailwind equivalents**. This is a mapping task, not a component swap. **Assume Tailwind is present** in the target repo (its default spacing/grid scale, 0.25rem per unit); if Tailwind is NOT present, configure it first (add `tailwindcss`/PostCSS + a `tailwind.config` with `content` covering the app templates), then map.

**PrimeFlex default grid**: 12 columns; gutter `.g-<n>` = `0.55rem × n` per side; margin/padding `.m-<n>`/`.p-<n>` = `0.25rem × n`. Tailwind's default scale is `0.25rem` per unit, so margin/padding/spacing map **1:1** by number; the grid gutter does NOT map 1:1 (see notes).

| PrimeFlex | Tailwind equivalent | Notes |
|-----------|---------------------|-------|
| `.flex` / `.inline-flex` | `flex` / `inline-flex` | 1:1 |
| `.row` / `.col` (stack) | `flex flex-row` / `flex flex-col` | PrimeFlex `.row` is a flex row; `.col` stacks (flex-col) |
| `.flex-wrap` / `.no-wrap` | `flex-wrap` / `flex-nowrap` | 1:1 |
| `.justify-start` / `-center` / `-end` / `-between` / `-around` / `-evenly` | `justify-start` / `-center` / `-end` / `-between` / `-around` / `-evenly` | 1:1 |
| `.align-start` / `-center` / `-end` / `-baseline` | `items-start` / `-center` / `-end` / `-baseline` | PrimeFlex `align-*` = Tailwind `items-*` (align-items) |
| `.align-content-*` (row-gap) | `content-*` | 1:1 |
| `.justify-content-*` | `justify-*` | 1:1 |
| `.grid` | `grid` | 1:1 (CSS grid) |
| `.col-<n>` (n = 1–12) | `col-span-<n>` | PrimeFlex 12-col; Tailwind default is 12-col grid → **1:1**. If the repo customizes the column count, re-derive. |
| `.col` (auto) / `.col-auto` | `col-span-1` (or `auto-cols-*`) | PrimeFlex `.col` = equal flex-grow span |
| `.col-offset-<n>` | `col-start-<n+1>` | offset is 0-based in PrimeFlex vs 1-based in `col-start` |
| `.row-<n>` (grid row) | `row-span-<n>` | |
| `.g-<n>` (grid gutter) | `gap-<n>` (approx) | **NOT 1:1**: PrimeFlex `.g-4` = `0.55rem×4 ≈ 2.2rem`; Tailwind `gap-4` = `1rem`. Map by measurement — record the actual rem value in Edge Cases and pick the closest Tailwind step (or use an arbitrary `gap-[1.1rem]` for exact parity). |
| `.m-<n>` / `.mt-<n>` / `.mr-<n>` / `.mb-<n>` / `.ml-<n>` / `.mx-<n>` / `.my-<n>` | `m-<n>` / `mt-<n>` / `mr-<n>` / `mb-<n>` / `ml-<n>` / `mx-<n>` / `my-<n>` | **1:1** (both 0.25rem scale) |
| `.p-<n>` / `pt` / `pr` / `pb` / `pl` / `px` / `py` | `p-<n>` / `pt-<n>` / `pr-<n>` / `pb-<n>` / `pl-<n>` / `px-<n>` / `py-<n>` | **1:1** (both 0.25rem scale) |
| `.m-0` / `.p-0` | `m-0` / `p-0` | 1:1 |
| `.m-auto` / `.m-5` (PrimeFlex "auto") | `m-auto` / `m-5` | PrimeFlex `.m-auto` = auto margin |
| `.text-left` / `-center` / `-right` / `-justify` | `text-left` / `-center` / `-right` / `-justify` | 1:1 |
| `.text-<size>` (`.text-sm`, `-base`, `-lg`, `-xl`) | `text-sm` / `text-base` / `text-lg` / `text-xl` | 1:1 (PrimeFlex reuses Tailwind's naming) |
| `.text-<color>` (PrimeFlex color names) | `text-<tailwind-color>` | PrimeFlex color names (`.text-red-500`) align with Tailwind palette where the names match |
| `.bg-<color>`, `.border`, `.rounded`, `.shadow-*`, `.overflow-hidden`, `.cursor-pointer` | same names | 1:1 where the name matches |
| `.flex-grow` / `.flex-shrink` / `.flex-1` | `grow` / `shrink` / `flex-1` | PrimeFlex `.flex-grow`→`grow`, `.flex-shrink`→`shrink`, `.flex-1`→`flex-1` |
| `.gap` helpers / `.gap-<n>` (PrimeFlex v4 gap utilities) | `gap-<n>` | 1:1 in v4 |

**Mapping rules:**
- Re-derive each mapping per task from the fetched PrimeFlex docs / the installed `node_modules/primeflex/` source — the table is a HINT, not a decision list.
- Where a mapping is approximate (grid gutter) or has no exact Tailwind step, either use an arbitrary value (`gap-[1.1rem]`) for exact parity or the nearest step — **record the chosen value in the task's Edge Cases**.
- Verify the target repo's Tailwind `content` paths cover the templates being edited so the generated utilities are actually built; extend `tailwind.config` if not.
- If two plausible Tailwind classes exist for one PrimeFlex class (e.g. `.col` → `col-span-1` vs `flex-1`), STOP and ask ONE question with the candidates.
- PrimeFlex classes used **only inside** PrimeNG component markup are handled by that component task, not the PrimeFlex task; this mapping covers PrimeFlex classes on the app's own markup.

## Special Rules (this migration path)

- **Version lockstep**: `@angular/material` major == `@angular/core` major. If the executor finds a mismatch or a doc suggests another major → STOP and ask; never install a mismatched major, never use `--legacy-peer-deps` to force it.
- **Module federation**: this repo ships a module-federation shell — when adding `@angular/material` (and `@angular/cdk`) packages, verify and update the sharing lists in `module-federation.config.ts` (Material shared as a versioned entry like other `@angular/*` shared packages).
- **Bootstrap/module dedup**: Material module imports or providers that belong to the root bootstrap must not be duplicated per component — keep shared providers in `bootstrap.ts`/`app.module.ts`, component-scoped imports in the component.
- **PrimeNG packages are KEPT (never removed in this migration)**: `primeng`, `primeflex`, `primeicons` stay in `package.json` for the entire migration. The OneCX v8 libraries still depend on PrimeNG, so removing them would break the build. Their removal is **out of scope** — it happens at the OneCX v9 / Angular 22 upgrade (when the Material-compatible v9 libraries are added). Notify the developer of this at the end of Phase A.
- **Legacy styling untouched**: `@onecx/portal-layout-styles` and everything under `pre_loaders/**` are out of scope — grep hits there are recorded as "excluded — pre_loaders/legacy scope", never edited.
- **Visual check is mandatory for user-visible tasks**: devserver + clean build + report local URL to the developer before marking the task done.

## Task-Specific Applicability Rules

- One task per PrimeNG component family (contiguous numbering per phase) + one task per PrimeFlex→Tailwind surface + one final cleanup task (config sweep — **no package removal**). The full model is the 2-phase A/B structure in `UI_MIGRATION_PROGRESS.md` (Phase A = component + PrimeFlex→Tailwind tasks, Phase B = cleanup + error recovery). `primeng`/`primeflex`/`primeicons` are **kept** in `package.json` for the whole migration (required by the OneCX v8 libraries; removed only at the OneCX v9 / Angular 22 upgrade).
- Applicability = **entire-repo grep** (all dirs, all file types) for the OLD patterns of that component family:
  1. `@primeng/<component>` imports (`.ts`)
  2. `<p-<component>` template tags (`.html`, including dynamic/`*ngIf`-wrapped occurrences)
  3. Related services/tokens (`DialogService`, `MessageService`, `ConfirmationService`, `providePrimeNG`, `primeng.config.ts`)
  4. Component-specific CSS overrides (`p-<component>` classes, theme-token references) — this repo keeps a `primeng-overwrites.utils.ts` pattern; check it
  5. Spec files referencing the component (selectors, `TestBed` providers)
- ALL patterns 0 hits (excluding `pre_loaders/**`) → `[-]`. ANY pattern with hits → task applies; execute applicable sub-steps, document the rest.
- Ambiguous candidate pairs (see starter table) → STOP, ask ONE question with the candidates and the repo evidence.
- Component replacement is DOC-DRIVEN: never rely on memory or the starter table alone — re-derive each pair from the fetched docs per task.

## Known Working References in this Repo

- `src/app/shell/utils/styles/primeng-overwrites.utils.ts` (and its spec) — existing PrimeNG CSS override patterns; the Material equivalent of these overrides is a per-task decision (ask, don't guess).
- `src/app/shell/components/error-component/global-error.component.html` — existing example with PrimeNG-flavored markup in a global component.
- `module-federation.config.ts` — sharing config to extend when Material packages are added.
- `create-dummies.js` / `build-preloaders.sh` — DO NOT TOUCH (pre_loaders pipeline).

## Final Verification and Validation (final task)

- [ ] Zero `@primeng/*` imports in in-scope files (whole-repo grep, `pre_loaders/**` excluded)
- [ ] Zero `<p-...>` tags in in-scope templates
- [ ] No `providePrimeNG` / `primeng.config.ts` references in scope
- [ ] Zero PrimeFlex utility classes (`.col-*`, `.g-*`, `.m-*`, `.p-*`, `.row`, `.flex`, ...) on app markup in scope — all replaced by Tailwind utilities (whole-repo grep, `pre_loaders/**` excluded)
- [ ] Tailwind `content` config covers the edited templates; generated utilities are built
- [ ] `primeng`, `primeflex`, `primeicons` **still present** in `package.json` (intentionally kept — required by the OneCX v8 libraries; removed only at the OneCX v9 / Angular 22 upgrade, not now)
- [ ] `module-federation.config.ts` sharing lists include the final `@angular/material` + `@angular/cdk` packages
- [ ] `npm run build` → `npm run lint` → `npm run test` all green (full output read, not just exit code)
- [ ] Devserver visual pass reported to the developer
