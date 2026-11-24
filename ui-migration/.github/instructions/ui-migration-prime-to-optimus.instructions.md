---
description: "Use for the PrimeNG to Optimus UI target: documentation sources, package and identifier rename table, Optimus-specific checks and final verification."
---
# UI Migration Path Data: PrimeNG → Optimus UI

<!--
  Path data for the PrimeNG → Optimus UI (OpenNG) swap.
  ACTIVE ONLY when the progress file's `Target:` field = "Optimus UI".
  (Angular Material uses ui-migration-prime-to-material.instructions.md.)

  KEY FACT (verified against the Optimus UI docs, 2026-11):
  Optimus UI is the open-source MIT continuation/rebrand of PrimeNG. Optimus v1
  targets Angular 21 and is API-compatible with PrimeNG v21 for component APIs
  and selectors. Optimus evolves independently after v1. A package rename does
  NOT migrate OneCX v8's PrimeNG theme integration to Optimus:
    - component selectors KEEP the `p-` prefix  →  <p-button> is unchanged
    - icon classes KEEP the `pi-` prefix        →  pi pi-check is unchanged
    - ONLY the npm package + import specifier + a few identifiers change
  So "component replacement" here means: swap the IMPORT PACKAGE and
  identifiers, verify the (unchanged) `p-*` / `pi-*` selectors, AND perform the
  version-specific OneCX/theme/CSS compatibility work described in
  `ui-migration-onecx-version-compat.instructions.md`.

  Optimus UI has NO dedicated MCP. The authoritative sources are the two
  official URLs below — fetch them with web fetch (fall back to `curl`) and
  re-read them at runtime. The npm package is KNOWN and documented
  (@openng/optimus-ui) — do not guess it, but confirm the exact major/version
  against the fetched docs.

  KEEP-PRIME MODE applies only when installed OneCX v8 dependencies actually
  require PrimeNG/PrimeFlex. Verify their package manifests and stylesheet
  imports. Do not run a workspace-wide schematic that replaces dependencies
  unless it can preserve those consumers and the developer approves it.
-->

## Migration Path

- **Source UI library**: PrimeNG (`p-*` template tags, `@primeng/*` imports, `providePrimeNG`, primeicons/primeflex)
- **Target UI library**: **Optimus UI** (OpenNG) — v1 targets Angular 21 and is API-compatible with PrimeNG v21; v2 targets Angular 22. The app-owned package/identifier rename leaves `p-*` tags and `pi-*` classes unchanged, but does not by itself migrate OneCX theme integration or CSS behavior. Follow `.github/instructions/ui-migration-onecx-version-compat.instructions.md` for installed OneCX-major-specific requirements.
- **Tailwind**: PrimeFlex layout/spacing/grid classes → **Tailwind** utilities (SHARED — identical to the Material path; see "PrimeFlex → Tailwind" section, including the size-difference check).
- **In scope**: `src/**` (components, templates, SCSS, specs, module-federation config, package.json for NEW-side deps)
- **OUT of scope (never modify)**: `pre_loaders/**` (auto-generated version-pinned stubs), `node_modules`, `@onecx/portal-layout-styles` (legacy MFE styling intentionally retained per README)

---

## What the migration ACTUALLY changes (verified against the Optimus docs)

**The Optimus component API migration is a package/identifier rename, not a component swap.** Keep component selectors and icon class names, but app templates still need Tailwind utility changes and the version-specific OneCX/theme/CSS integration described below.

### What CHANGES (import + identifier + asset layer, `.ts` / `.mts` / `angular.json`)

| PrimeNG                                 | Optimus UI                                                    | Layer                                                                                                                                            |
| --------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `primeng`                               | `@openng/optimus-ui`                                          | package / import base                                                                                                                            |
| `primeng/<sub>` (e.g. `primeng/button`) | `@openng/optimus-ui/<sub>` (e.g. `@openng/optimus-ui/button`) | import subpath — **suffix kept**                                                                                                                 |
| `providePrimeNG`                        | `provideOptimus`                                              | config identifier (`@openng/optimus-ui/config`)                                                                                                  |
| `PrimeNGConfigType`                     | `OptimusConfigType`                                           | config identifier                                                                                                                                |
| `primeicons`                            | `@openng/icons`                                               | package / icon stylesheet                                                                                                                        |
| `primeicons/primeicons.css`             | keep `primeicons/primeicons.css`                              | OneCX v8 and v9 both use it. Import ONE icon font only; adding `@openng/icons/openng-icons.css` as well causes a `.pi` font clash. |
| `PrimeIcons`                            | `OpenngIcons`                                                 | icon constants class (old still compiles via a deprecated alias; use the new name)                                                               |
| `@primeuix/themes`                      | `@openng/optimus-ui-themes`                                   | theme preset package (e.g. `@openng/optimus-ui-themes/aura`)                                                                                     |
| `@primeuix/styled`                      | `@openng/optimus-ui-styled`                                   | styled-mode package                                                                                                                              |
| `@primeuix/styles`                      | `@openng/optimus-ui-styles`                                   | unstyled-mode package                                                                                                                            |
| `@primeuix/utils`                       | `@openng/optimus-ui-utils`                                    | utils package                                                                                                                                    |
| `@primeuix/motion`                      | `@openng/optimus-ui-motion`                                   | motion package                                                                                                                                   |

### What does NOT change (selectors / icons / PrimeFlex)

- **Component selectors**: `p-*` → `p-*` (e.g. `<p-button>`, `<p-table>`, `<p-dialog>` — **unchanged**). Do NOT rename or "swap" template tags.
- **Icon classes**: `pi-*` → `pi-*` (e.g. `pi pi-check` — **unchanged**); only the icon _package_ behind them changes (`primeicons` → `@openng/icons`).
- **PrimeFlex**: stays on the **OLD** side of this path and moves to **Tailwind** via the shared file (not part of the PrimeNG→Optimus rename).
- **Behavior / component API**: Optimus v1 is API-compatible with PrimeNG v21 for the documented component API and selectors. Do not assume OneCX providers, Shell theme propagation, CSS scoping, overlays, defaults or all styling behavior are automatically equivalent.

> The official `migrate-from-primeng` schematic performs a workspace-wide package/import migration and package installation. In a keep-Prime OneCX v8 app, do not run it blindly: first inspect exactly which package manifests and files it changes, and use it only if it can preserve OneCX v8 consumers and the developer approves. Otherwise perform a reviewed app-owned rename manually.

---

## Documentation Sources (Optimus has NO MCP — use web fetch, fall back to curl, then node_modules)

| Source                                                                                                                 | Tool / method                                                                                             | Purpose                                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PrimeNG MCP**                                                                                                        | `primeng/*`                                                                                         | audit the OLD PrimeNG usage (component API, template syntax, behavior) — SHARED with the Material path (if the MCP is down, fall back to `node_modules/primeng/`)                     |
| **Optimus v1 — installation**                                                                                          | web fetch `https://v1.optimus.openng.org/installation`                                                     | Angular 21 compatibility, package names, root `provideOptimus` setup.                                                                                                                 |
| **Optimus v1 — PrimeNG migration**                                                                                     | web fetch `https://v1.optimus.openng.org/migration/primeng`                                                | Rename/schematic behavior and PrimeNG v21 prerequisite. Re-read at runtime; do not rely on memory.                                                                                    |
| **Optimus v1 — Tailwind**                                                                                              | web fetch `https://v1.optimus.openng.org/tailwind`                                                         | Official Tailwind v3 plugin setup, utility tokens and CSS-layer guidance.                                                                                                             |
| **Optimus v1 — theming/configuration**                                                                                 | web fetch `https://v1.optimus.openng.org/theming/styled` and `https://v1.optimus.openng.org/configuration` | Token/preset structure, prefix, runtime configuration and overlay options. These docs do not replace OneCX v8 integration evidence.                                                   |
| **Optimus UI — upgrade**                                                                                               | web fetch `https://v1.optimus.openng.org/migration/update`                                                 | Angular-first / matching Optimus-major upgrade sequence and release-note requirement.                                                                                                 |
| **`node_modules/@openng/optimus-ui/`, `@openng/optimus-ui-themes/`, `@openng/icons/`** (+ `primeng/` for the OLD side) | Read/grep the installed `.d.ts`, `package.json`, `CHANGELOG`, `openng-icons.css`                          | **LAST-RESORT evidence source** — used to VERIFY the actual package name, subpath layout, identifier exports, and asset filename when the web is unreachable (see the fallback below) |

**How to verify (Gate 3 for this path):** probe `primeng/version` (OLD side) and fetch the version-matched Optimus migration, Tailwind and theming sources (web fetch; if policy-blocked, fall back to `curl -sSL`). Also inspect installed OneCX package versions and exports. `angular-cli/list_projects` is used for build/devserver validation. If web docs are unavailable, trace `node_modules/@openng/*`, OneCX theme providers, Tailwind plugin and PrimeFlex CSS; record the exact evidence. Never infer OneCX compatibility from the Optimus rename guide alone.

### PrimeNG MCP (shared)

- **available**: `npx -y @primeng/mcp`
- **key-tools**: `primeng/list`, `primeng/version`, `primeng/get_component` (`includeApi`, `includeExamples`), `primeng/search`, `primeng/get_example`, `primeng/validate_usage`
- **purpose**: audit the OLD PrimeNG usage — component API, template syntax, behavior. Verify `primeng/version` matches the installed PrimeNG major before trusting its data. (Note: Optimus v1 mirrors PrimeNG **v21** — if the repo is not yet on PrimeNG v21, the docs require updating to `primeng@21` first, which is a prerequisite to record, not a step this path performs.)
- **fallback**: `https://v{N}.primeng.org/` (version-pinned) and `node_modules/primeng/` source + CHANGELOG.

### Optimus UI docs (web fetch — no MCP)

- **Installation / migration / Tailwind / theming docs**: use versioned v1 URLs for Angular 21; record package versions, Angular peer, provider/preset setup, Tailwind plugin setup, and rename behavior. Versioned v1 docs are preferred over unversioned current docs for an Angular 21 app.
- Optimus component API compatibility is not proof of OneCX Shell theme integration. The OneCX v8 theme provider configures PrimeNG. For a v8 app, follow `ui-migration-onecx-version-compat.instructions.md`, inspect installed OneCX source, and verify any temporary adapter at runtime.
- **fallback (LAST-RESORT — VERIFY via `node_modules`, don't STOP immediately)**: if a URL is unreachable or silent → (1) `curl -sSL <url>`; (2) **trace the installed packages and VERIFY the rename in place of the web** — read `node_modules/@openng/optimus-ui/` (confirm the exact package name, subpath layout, and the `provideOptimus` / `OptimusConfigType` exports), `node_modules/@openng/optimus-ui-themes/`, and `node_modules/@openng/icons/openng-icons.css` (confirm the asset filename), and `node_modules/primeng/` for the OLD side; record `node_modules/<pkg>` as the source. The scoped `npm run build` is the final check — a rename that RESOLVES and BUILDS proves the traced evidence was correct. (3) **STOP and ask the developer ONLY if those packages are not installed either** (no `node_modules` evidence) — do NOT guess a rename or an icon name.

---

## NEW-side tooling (executor) — web fetch-driven (no MCP for Optimus)

- **Audit the OLD side** with the PrimeNG MCP exactly as in the Material path (`get_component`, `search`, `validate_usage`) — to confirm the OLD usage you are renaming is understood.
- **Confirm the NEW side by web fetch**: fetch the version-matched `migration/primeng` page for the package/identifier rename and the Tailwind/theming pages for setup. Component API compatibility does not establish OneCX theme integration; inspect OneCX package sources for the active major.
- **Validate the build/devserver** with `angular-cli/*` (`run_target`, `devserver_start` / `devserver_wait_for_build`) — target-agnostic Angular CLI validation, used for BOTH targets.
- **MCP discipline**: never blindly apply a rename from memory — confirm it against the fetched migration page; record the URL + the exact old→new pair for every decision. **If the web is unreachable → fall back to VERIFYING the rename against `node_modules/@openng/optimus-ui/` + `@openng/icons/` (last resort), and record `node_modules/<pkg>` as the source instead** (see the Documentation Sources fallback); only STOP if those packages are not installed.

---

## PrimeNG → Optimus UI "Mapping" (RENAME — selectors unchanged)

Because selectors and API are unchanged, there is **no component-pair table to build**. The per-family work is uniform: rename the import package + identifiers for that family's files, and confirm the `p-*`/`pi-*` tags are untouched.

Per-family rename + verify (the "mapping" for a task):

1. **Import base** — `primeng/<sub>` → `@openng/optimus-ui/<sub>` (keep the suffix; e.g. `primeng/button` → `@openng/optimus-ui/button`).
2. **Config identifiers** — `providePrimeNG` → `provideOptimus`, `PrimeNGConfigType` → `OptimusConfigType` (only where present).
3. **Icons** — keep `primeicons/primeicons.css` as the single icon font (do not add `@openng/icons/openng-icons.css` next to it); `PrimeIcons` → `OpenngIcons` in code (only where present). `pi-*` classes are **unchanged**.
4. **Selectors** — `p-*` template tags: **leave unchanged**. Grep to PROVE they are present and were not accidentally renamed.
5. **Scope guard** — apply ONLY to files the app owns; NEVER touch `pre_loaders/**` (kept on `primeng`) or any file that must remain on PrimeNG for the OneCX v8 libraries.

- If the migration page shows a rename that drifts from the table above, or a family with a non-obvious exception → mark it **AMBIGUOUS — ask the developer ONE question**; never guess.
- Record the source URL for each rename applied.

---

## PrimeFlex → Tailwind (SHARED — identical to the Material path)

PrimeFlex→Tailwind is **target-independent** — the layout migration is identical for Material and Optimus UI. Its full mapping table, the MANDATORY size-difference check, and the Tailwind tooling (Tailwind MCP / skills / doc sources) live in the shared, single-source-of-truth file:

→ [`.github/instructions/ui-migration-primeflex-to-tailwind.instructions.md`](ui-migration-primeflex-to-tailwind.instructions.md)

Read that file for every PrimeFlex→Tailwind task. Do NOT duplicate or re-derive the mapping here — the shared file is authoritative so it is maintained in exactly one place.

---

## Behavioral Differences (PrimeNG → Optimus)

- **v1 component APIs/selectors are compatible with PrimeNG v21** by design. Do not infer that OneCX providers, Shell theme propagation, style scoping, overlays, defaults, or Tailwind/PrimeFlex behavior are equivalent. Verify runtime behavior for the installed OneCX major.
- **Divergence is post-v1 only**: "From v1 onward, Optimus UI evolves independently and future versions will diverge from PrimeNG." If the app later upgrades past v1, re-check behavior against the current Optimus docs. For THIS migration, behavioral differences are **not expected**; if a diff surfaces at build/runtime, record it in the task's Edge Cases field and ask — never silently drop functionality.
- **Theming**: Optimus provides its own presets and Tailwind plugin. Do not claim that renaming a PrimeNG provider or choosing the same preset migrates OneCX Shell theme values. Verify root provider, dynamic Shell theme mapping, scoped style hooks and overlay scope for the installed OneCX major. Prefer semantic/component tokens and the official plugin over CSS override classes.
- **Icons**: `pi-*` glyph classes are unchanged; the icon _package_ changes (`primeicons` → `@openng/icons`). Verify the icon stylesheet import path is renamed, not the glyph names.

---

## Special Rules (this migration path)

- **Version / peer compatibility**: match the Optimus major to the installed `@angular/core` major — **Optimus v1 → Angular 21**, **Optimus v2 → Angular 22** (from the installation page). Pin the major (`@openng/optimus-ui@<major>`); an unpinned install resolves to the wrong Angular peer requirements. If the executor finds a mismatch → **STOP and ask**; never install a mismatched major, never use `--legacy-peer-deps` to force it.
- **Prerequisite — PrimeNG v21**: Optimus v1 mirrors PrimeNG **v21**. The docs require the project to be on `primeng@21` first. Record the installed PrimeNG major in the progress file; if it is below v21, flag it to the developer (updating to v21 is a prerequisite to record, not a step this path silently performs).
- **Version-aware coexistence**: inspect installed OneCX package versions/peer dependencies and actual CSS imports before deciding whether PrimeNG, PrimeFlex or PrimeIcons stay. In the current OneCX v8 / Angular 21 profile they are retained where v8 packages/styles still consume them; this is not a permanent blanket rule.
- **Optimus on OneCX v8**: the theme adapter in `ui-migration-onecx-version-compat.instructions.md` is REQUIRED, not optional. Without it Optimus has no preset (black or transparent buttons and overlays). Copy it from that file; do not write a palette or global colour rules.
- **Module federation**: this repo ships a module-federation shell — when adding the Optimus package (and any shared `@angular/*` deps), verify and update the sharing lists in `module-federation.config.ts` (list BOTH `primeng` and `@openng/optimus-ui` if both remain in play during the transition).
- **Bootstrap/module dedup**: the `provideOptimus` provider that belongs in the root bootstrap must not be duplicated per component — keep shared providers in `bootstrap.ts`/`app.config.ts`, component-scoped imports in the component. During the transition both `providePrimeNG` (for `pre_loaders`/OneCX v8) and `provideOptimus` (for app code) may coexist — keep both, dedup within each.
- **Deferred v9 cleanup is mandatory**: at final completion, write a concrete “Remove after Angular 22 / OneCX v9 upgrade” checklist. Assess the v8 theme adapter, any temporary Tailwind prefix/classes, Tailwind v3 plugin/config, PrimeFlex/PrimeNG/PrimeIcons and federation entries. Mark each remove/replace/retain with evidence. Do not remove a package in v8 unless installed OneCX dependencies, CSS, imports and `npm ls` prove it is no longer needed. See `ui-migration-onecx-version-compat.instructions.md`.
- **Legacy styling untouched**: `@onecx/portal-layout-styles` and everything under `pre_loaders/**` are out of scope — grep hits there are recorded as "excluded — pre_loaders/legacy scope (kept on PrimeNG)", never edited.
- **Visual check is mandatory for user-visible tasks**: devserver + clean build + report local URL to the developer before marking the task done. Unchanged selectors do not guarantee visual parity; check theme tokens, PrimeFlex/Tailwind collisions, overlays and icons.

## Task-Specific Applicability Rules

- One task per PrimeNG component family + PrimeFlex→Tailwind surface + compatibility/final cleanup tasks as required by the installed version profile. Keep dependencies only where installed OneCX peers, CSS and source/runtime usage require them; the final v9 cleanup handoff is mandatory.
- Each component-family task is a **rename + verify**, NOT a tag swap. Applicability = **entire-repo grep** (all dirs, all file types) for the OLD PrimeNG _import/identifier_ patterns of that component family:
  1. `@primeng/<component>` / `primeng/<component>` imports (`.ts`) — these get renamed to `@openng/optimus-ui/<component>`
  2. `<p-<component>` template tags (`.html`) — **MUST remain unchanged**; grep to confirm presence and that none were renamed (a `p-` tag edited to a non-`p-` prefix is a BUG)
  3. Related config/identifiers (`providePrimeNG`, `PrimeNGConfigType`, `primeng.config.ts`) — renamed to `provideOptimus` / `OptimusConfigType`
  4. Icon references (`PrimeIcons`) — renamed to `OpenngIcons`; keep the `primeicons/primeicons.css` stylesheet; `pi-*` classes unchanged
  5. Component-specific CSS overrides (`p-<component>` classes, theme-token references) — this repo keeps a `primeng-overwrites.utils.ts` pattern; check it (selector-based `p-*` overrides stay valid; only theme-token/preset imports change)
  6. Spec files referencing the component (imports, `TestBed` providers) — imports renamed, selectors unchanged
- **Scope discipline**: the rename applies ONLY to app-owned files. Any `@primeng/*` reference that lives under `pre_loaders/**` or belongs to a OneCX v8 library stays on `primeng`. When a file is ambiguous (owned by the app vs. the preloader/legacy layer) → STOP and ask.
- ALL in-scope patterns 0 hits (excluding `pre_loaders/**`) → `[-]`. ANY in-scope pattern with hits → task applies; execute the applicable sub-steps, document the rest.
- A rename that is not clear from the fetched migration page (or that would require touching a legacy/preloader file) → STOP, ask ONE question with the candidates and the repo evidence.
- The rename is DOC-DRIVEN: confirm every old→new pair against the fetched `migration/primeng` page per task — never rely on memory or the table alone.

## Known Working References in this Repo

- `src/app/shell/utils/styles/primeng-overwrites.utils.ts` (and its spec) — existing PrimeNG CSS override patterns. Because `p-*` selectors are unchanged, most of these overrides remain valid; only theme-token / preset imports change. Treat each override as a per-task decision (ask, don't guess).
- `src/app/shell/components/error-component/global-error.component.html` — existing example with PrimeNG-flavored markup in a global component.
- `module-federation.config.ts` — sharing config to extend when `@openng/optimus-ui` is added.
- `create-dummies.js` / `build-preloaders.sh` — DO NOT TOUCH (pre_loaders pipeline).

## Final Verification and Validation (final task)

- [ ] Zero `primeng/*` / `@primeng/*` imports in **app-owned** files (whole-repo grep, `pre_loaders/**` and OneCX v8 library files excluded) — all renamed to `@openng/optimus-ui/*`
- [ ] `<p-...>` tags **still present and unchanged** in in-scope templates (grep proves the selectors were NOT renamed — a rename to a non-`p-` prefix would be a failure)
- [ ] No `providePrimeNG` / `PrimeNGConfigType` / `primeng.config.ts` references in **app-owned** scope (renamed to `provideOptimus` / `OptimusConfigType`)
- [ ] Exactly one icon font stylesheet (`primeicons/primeicons.css`) is imported; `pi-*` classes unchanged
- [ ] `@openng/optimus-ui` present in `package.json` with the major matching `@angular/core` (Optimus v1→Angular 21 / v2→Angular 22, from the installation page)
- [ ] Zero PrimeFlex utility classes (`.col-*`, `.g-*`, `.m-*`, `.row`, `.flex`, ...) on app markup in scope — all replaced by Tailwind utilities **with size parity verified** (whole-repo grep, `pre_loaders/**` excluded)
- [ ] Tailwind `content` config covers the edited templates; generated utilities are built
- [ ] Legacy package decisions match installed peer dependencies, CSS imports and source/runtime usage; deferred v9 cleanup is recorded with evidence and conditions
- [ ] `module-federation.config.ts` sharing lists include the final `@openng/optimus-ui` + shared `@angular/*` packages
- [ ] OneCX v8 theme changes propagate to Optimus and overlays; runtime proof recorded (or explicitly blocked)
- [ ] Explicit v9 cleanup handoff lists every temporary v8 adapter/config/prefix/dependency with its removal condition; no blanket removal claim
- [ ] `npm run build` → `npm run lint` → `npm run test` all green (full output read, not just exit code)
- [ ] Devserver visual pass reported to the developer
