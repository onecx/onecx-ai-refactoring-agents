# PrimeNG → Angular Material Migration Data

<!--
  ACTIVE ONLY when the progress file's `Target:` field = 'Angular Material'.
  (Optimus UI uses ui-migration-prime-to-optimus.instructions.md instead.)
  This file contains ALL data for the PrimeNG → Angular Material UI swap.
  It is NOT auto-injected (no applyTo). ui-migration-executor and the
  /migrate-primeng command read it on-demand (per the `Target:` field).
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

| MCP                                       | Tool prefix              | Purpose                                                                | Config shape (see `templates/mcp.example.json`) |
| ----------------------------------------- | ------------------------ | ---------------------------------------------------------------------- | ----------------------------------------------- |
| **PrimeNG MCP**                           | `mcp__primeng__*`        | audit the OLD PrimeNG usage (component API, template syntax, behavior) | `npx -y @primeng/mcp`                           |
| **Material-side MCP (docs + validation)** | `mcp__angular-cli__*`    | NEW-side Material API reference + build/devserver validation           | `npx -y @angular/cli mcp`                       |
| **Material-side MCP (generator)**         | `mcp__ngm-dev-blocks__*` | generate the new Angular Material + Tailwind blocks                    | `npx -y @ngm-dev/cli mcp <abs-project-dir>`     |

**How to verify:** list the configured MCP tools (e.g. probe `mcp__primeng__version` and `mcp__angular-cli__list_projects`). If a server is missing or errors as unconfigured:

- **First offer to add it** to the target repo's MCP file from `templates/mcp.example.json` (write to root `mcp.json` / `.mcp.json` / `.vscode/mcp.json` — whichever the target uses), **with the developer's OK**, then re-verify.
- If it CANNOT be configured (or the developer declines) → do **NOT** immediately HALT: use the **Documentation Fallback Priority** above to gather + VERIFY the needed evidence from `node_modules/` (the installed `primeng` / `@angular/material` / `@angular/cdk` `.d.ts` + `package.json`) in place of the MCP, and record `node_modules/<pkg>` as the source. **HALT only if `node_modules` also yields no evidence** (target package not installed / `.d.ts` absent).
- Only proceed once the evidence for the flow is either MCP-sourced or node_modules-verified — never proceed on a guessed mapping.

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
2. Fallback URLs via WebFetch: `material.angular.dev` (match installed Material major), `v{N}.primeng.org` (match installed PrimeNG major)
3. **`node_modules/<package>/` — LAST-RESORT evidence source; VERIFY it in place of the web.** When the MCP and the URLs are both unreachable/policy-blocked/returning nothing, do **NOT** immediately HALT and do **NOT** guess — trace the installed packages to gather AND verify the evidence, and record `node_modules/<pkg>` as the source:
   - OLD side: `node_modules/primeng/` (`package.json` subpath exports + installed major, the component's `.d.ts` = the real exported directives/services + inputs/outputs, `CHANGELOG`), `node_modules/primeflex/primeflex.css`, `node_modules/primeicons/primeicons.css`.
   - NEW side: `node_modules/@angular/material/<component>/…d.ts` (the real `MatXxx` export names + inputs/outputs) and `node_modules/@angular/cdk/`.
   - **The scoped `npm run build` is the final verification** — a chosen/derived import that RESOLVES and BUILDS proves the traced evidence was correct.
   - **HALT only if `node_modules` also yields no evidence** (target package not installed / `.d.ts` absent).

Record for every component decision which source (MCP / URL / node_modules) was used.

## Target Versions

- `@angular/material`: major MUST equal the installed `@angular/core` major (currently **22** → `@angular/material@^22`)
- `@angular/cdk`: same major
- Resolve to explicit stable: `npm view @angular/material@<major> dist-tags` — **NEVER** `@latest`
- Install of `@angular/material`/`@angular/cdk` requires user confirmation first (executor Step 0 — installing a design system is an architectural decision, not a routine task step)
- Version bumps of Angular itself are OUT OF SCOPE — that belongs to the Angular version migration agents

## Component Pair Starter Table (EXAMPLES ONLY — not a decision list)

The Angular Material docs do not publish an official PrimeNG→Material mapping. The table below is a HINT from this repo's planning; each pair MUST be re-derived per task from fetched docs (executor Step 2) and recorded in the task evidence. If a PrimeNG component has no clean Material equivalent, or two plausible candidates exist → **ASK the user ONE question with the candidates** — never guess.

| PrimeNG                                                                 | Angular Material candidate                                                      | Notes                                                                                              |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `<p-button>`                                                            | `MatButton` / `MatIconButton`                                                   | severity/variant mapping differs (text/outlined/flat/stroked); check icon placement                |
| `<p-badge>`                                                             | `MatBadge`                                                                      |                                                                                                    |
| `<p-tag>`                                                               | `MatBadge` or `MatChip`                                                         | **ambiguous — ask**                                                                                |
| `<p-dialog>`                                                            | `MatDialog` + `MatDialogRef`                                                    | ref/data API differs from PrimeNG `DialogService`                                                  |
| `<p-table>` / `<p-dataView>`                                            | `MatTable` + `MatTableDataSource` (+ `MatSort`/`MatPaginator` where applicable) | column defs differ; check sorting/filtering parity                                                 |
| `<p-select>` / `<p-dropdown>`                                           | `MatSelect`                                                                     | options via `MatOption`                                                                            |
| `<p-multiselect>`                                                       | `MatSelect` with `multiple`                                                     |                                                                                                    |
| `<p-checkbox>`                                                          | `MatCheckbox`                                                                   | label moved out of input (PrimeNG 19+ dropped `[label]`)                                           |
| `<p-radiobutton>`                                                       | `MatRadioButton`                                                                |                                                                                                    |
| `<p-inputtext>` / inputs                                                | `MatInput` inside `MatFormField`                                                | `ngModel`/`formControl` binding replaces `p-model`/`[value]`                                       |
| `<p-inputgroup>` / addon spans                                          | `MatFormField` + `MatPrefix`/`MatSuffix`                                        |                                                                                                    |
| `<p-floatlabel>`                                                        | `MatFormField` + `MatLabel` (floating label is native)                          |                                                                                                    |
| `MessageService` / `<p-messages>`                                       | `MatSnackBar`                                                                   | API differs (snackbar, not stackable messages)                                                     |
| `<p-confirmdialog>` / `ConfirmationService`                             | `MatDialog` confirm pattern                                                     |                                                                                                    |
| `<p-toolbar>`                                                           | `MatToolbar`                                                                    |                                                                                                    |
| `<p-menu>`                                                              | `MatMenu`                                                                       |                                                                                                    |
| `<p-contextmenu>`                                                       | `MatMenu` in `Overlay`                                                          |                                                                                                    |
| `<p-tooltip>`                                                           | `MatTooltip`                                                                    |                                                                                                    |
| `<p-card>`                                                              | plain markup + styles (no Material card primitive)                              | **verify — no direct equivalent**                                                                  |
| `<p-paginator>`                                                         | `MatPaginator`                                                                  |                                                                                                    |
| `<p-calendar>`                                                          | `MatDatepicker` (+ `MatNativeDateModule`/adapters)                              | date model differs                                                                                 |
| `<p-accordion>` / `<p-panel>`                                           | `MatExpansionPanel`                                                             | **verify behavior parity**                                                                         |
| `<p-stepper>`                                                           | `MatStepper`                                                                    |                                                                                                    |
| `<p-tabview>`                                                           | `MatTabGroup`                                                                   |                                                                                                    |
| `<p-fileupload>`                                                        | `MatButton` + `<input type="file">`                                             | no Material fileupload primitive                                                                   |
| `<p-progressspinner>`                                                   | `MatProgressSpinner`                                                            |                                                                                                    |
| `<p-progressbar>`                                                       | `MatProgressBar`                                                                |                                                                                                    |
| `<p-skeleton>`                                                          | CSS shimmer (no Material primitive)                                             | **no direct equivalent**                                                                           |
| `<p-listbox>`                                                           | `MatSelectionList`                                                              |                                                                                                    |
| `<p-tree>`                                                              | `MatTree`                                                                       |                                                                                                    |
| `primeicons` (pi-\* classes)                                            | `@angular/material-icons` (`mat-icon` ligatures)                                | icon NAMES differ (pi-\* → material icon names) — check each icon used                             |
| `primeng/resources/*` CSS                                               | Material theme (SCSS mixins or prebuilt theme)                                  | global theming approach changes                                                                    |
| `primeflex` layout classes (`.flex`, `.grid`, `.col-6`, `.g-*`, `.m-*`) | **Tailwind utilities** (see "PrimeFlex → Tailwind" section)                     | **value/measurement mapping — see section**; assume Tailwind is present (add/configure if missing) |

## Behavioral Differences (verify per component task, record in Edge Cases)

- **Theming**: PrimeNG CSS themes (theme-preset + `primeng/resources/*`) vs Material theme (SCSS mixins / prebuilt). Do not mix; document which Material theme the app adopts.
- **Form controls**: PrimeNG `p-model` / `[value]` / `[options]` → Material `ngModel` / `formControl` / `MatOption`. Form-control validation styling differs (`p-invalid` vs `NgForm` error classes).
- **Dialogs**: PrimeNG/OneCX dialog services (`DialogService`, `PortalDialogService`) vs `MatDialog` ref API. Check which dialog service the task's files actually use before mapping.
- **Toasts**: `MessageService` (stacked) vs `MatSnackBar` (single) — stacked-toast patterns need an adapter or a UI decision (ask).
- **Icons**: `primeicons` glyph names do not map 1:1 to Material ligatures — verify every `pi-*` icon in scope files against Material icon names.
- **Animations**: Material components require animations enabled — verify `provideAnimations()` (or `BrowserAnimationsModule`) is present in bootstrap before relying on transitions.

## PrimeFlex → Tailwind (SHARED — identical to the Optimus path)

PrimeFlex→Tailwind is **target-independent** — the layout migration is identical for Material and Optimus UI. Its full mapping table, the MANDATORY size-difference check, and the Tailwind tooling (Tailwind MCP / skills / doc sources) live in the shared, single-source-of-truth file:

→ [`.claude/instructions/ui-migration-primeflex-to-tailwind.instructions.md`](ui-migration-primeflex-to-tailwind.instructions.md)

Read that file for every PrimeFlex→Tailwind task. Do NOT duplicate or re-derive the mapping here — the shared file is authoritative so it is maintained in exactly one place.

## Special Rules (this migration path)

- **Version lockstep**: `@angular/material` major == `@angular/core` major. If the executor finds a mismatch or a doc suggests another major → STOP and ask; never install a mismatched major, never use `--legacy-peer-deps` to force it.
- **Module federation**: this repo ships a module-federation shell — when adding `@angular/material` (and `@angular/cdk`) packages, verify and update the sharing lists in `module-federation.config.ts` (Material shared as a versioned entry like other `@angular/*` shared packages).
- **Bootstrap/module dedup**: Material module imports or providers that belong to the root bootstrap must not be duplicated per component — keep shared providers in `bootstrap.ts`/`app.module.ts`, component-scoped imports in the component.
- **Version-aware dependency handling**: do not remove PrimeNG/PrimeFlex/PrimeIcons solely because app-owned code migrated. Inspect the installed OneCX peer dependencies, stylesheets and source/runtime usage; record evidence for each retain/remove decision. If OneCX v8 requires them, retain until the compatible OneCX upgrade and include them in the final conditional cleanup handoff.
- **Legacy styling untouched**: `@onecx/portal-layout-styles` and everything under `pre_loaders/**` are out of scope — grep hits there are recorded as "excluded — pre_loaders/legacy scope", never edited.
- **Visual check is mandatory for user-visible tasks**: devserver + clean build + report local URL to the developer before marking the task done.

## Task-Specific Applicability Rules

- One task per PrimeNG component family + PrimeFlex→Tailwind surface + any evidence-required compatibility/cleanup task. Do not remove packages during component tasks; decide cleanup from installed dependency/style evidence and record v9 upgrade conditions at final verification.
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
- [ ] **Tailwind size parity verified** — each replaced utility's measured value matches its PrimeFlex source value (arbitrary values used where the Tailwind scale differs; see the size-difference rule above)
- [ ] Tailwind `content` config covers the edited templates; generated utilities are built
- [ ] Legacy package decisions match installed peer dependencies, CSS imports and source/runtime usage; deferred v9 cleanup is recorded with evidence and conditions
- [ ] `module-federation.config.ts` sharing lists include the final `@angular/material` + `@angular/cdk` packages
- [ ] `npm run build` → `npm run lint` → `npm run test` all green (full output read, not just exit code)
- [ ] Devserver visual pass reported to the developer
