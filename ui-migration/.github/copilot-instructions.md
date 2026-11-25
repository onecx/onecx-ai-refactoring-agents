# Copilot Instructions

## Purpose

This repository is a **showcase and test application** for OneCX microfrontend components and supported UI libraries. Its feature modules demonstrate and validate the usage of:

- **OneCX Angular Accelerator** (`ocx-*` components such as `ocx-content-container`, `ocx-data-list-grid`, `ocx-interactive-data-view`, `ocx-loading-indicator`, etc.)
- **Optimus UI** in app-owned showcases, with PrimeNG retained where the installed OneCX v8 libraries require it
- **PrimeNG / PrimeFlex** (OneCX v8 peer/runtime/style dependencies; do not assume the app-owned Optimus rename makes them removable)
- **Angular Material / CDK** (patched in webpack to work inside web components)
- **ngx-translate** (multi-language support via `OnecxTranslateLoader`)

Each route under `src/app/` is an isolated test case for a specific component or feature.

## Commands

```bash
npm run start        # Dev server at http://localhost:4200
npm run build        # Production build (sets NODE_ENV=production, outputs to dist/)
npm run lint         # ESLint (angular-eslint + typescript-eslint)
npm run test:styles  # Production CSS build + style-regression tests
```

`npm test` remains a no-op; use `npm run test:styles` for the migration's generated-CSS regression checks.

## Architecture

This is a **OneCX Angular 21 microfrontend (MFE)** that runs as a Module Federation remote inside the OneCX Shell. It is not a standalone application in production.

### Two bootstrap paths

| File                                                                          | Purpose                                                                |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `src/main.ts` → `src/app/app-module.ts`                                       | Standalone dev mode only. Not used in remote loading.                  |
| `src/remote-main.ts` → `src/remote-bootstrap.ts` → `src/app/remote.module.ts` | Remote/production path. Exposes `./RemoteModule` via `remoteEntry.js`. |

`RemoteModule` implements `DoBootstrap` and registers `AppEntrypointComponent` as the Custom Element `ocx-test-component-21` using `createAppEntrypoint` from `@onecx/angular-webcomponents`.

### Module Federation

- Configured in `module-federation.config.js` and applied via `ngx-build-plus` + `webpack.config.js`.
- Share scope is `angular_21` — all shared packages must declare this scope.
- `module-federation.config.js` auto-discovers all `dependencies` from `package.json`, reads each package's `exports` map to generate subpackage entries, and applies `getOneCXSharedRecommendations` from `@onecx/accelerator` to set the correct sharing strategy.
- The webpack config patches `@angular/cdk` and `@angular/material` modules, replacing `document.createElement(` calls with a custom wrapper to support web component encapsulation.

### Routing

Routes in `RemoteModule` use `matcher: startsWith('...')` from `@onecx/angular-webcomponents` instead of `path`. This is required for OneCX Shell compatibility. Each route lazy-loads an NgModule. The standalone `AppRoutingModule` (dev only) uses standard `path` routing.

### Feature modules

Each feature under `src/app/` (e.g. `overview`, `detail`, `data-list-grid-test`) is a self-contained lazy-loaded NgModule. Modules that use `ocx-*` components must import both `AngularAcceleratorModule` and `TranslateModule`. Do not import `AngularAcceleratorPrimeNgModule`: it adds PrimeNG `p-*` components that clash with the Optimus ones; import the specific `@openng/optimus-ui/<name>` modules instead.

## Key Conventions

### Component/directive generation

Angular schematics are configured in `angular.json` to generate non-standalone components/directives/pipes with `skipTests: true`. Do not generate standalone components unless intentionally overriding this.

### ESLint rules

- Component selector: `app-*` prefix, kebab-case element
- Directive selector: `app*` prefix, camelCase attribute
- `@angular-eslint/prefer-standalone` is turned off

### Prettier

- Print width: 100
- Single quotes
- HTML files use the `angular` parser

### i18n

Translation files live in `public/i18n/` (`en.json`, `de.json`). Keys use SCREAMING_SNAKE_CASE in a nested structure mirroring feature areas. Translation is wired via `provideTranslationPathFromMeta(import.meta.url, 'assets/i18n/')` in `RemoteModule`. `TranslateModule` must be imported in each feature module to use the `translate` pipe.

### Styles

- Global styles are in `src/styles.scss`. This Angular 21 / OneCX v8 app retains the accelerator's PrimeFlex styles for v8 internal component classes and uses the compatible Optimus Tailwind plugin for app-owned tokens/utilities.
- App-owned utilities currently use the temporary `tw-` Tailwind prefix to avoid collisions with OneCX v8's unprefixed PrimeFlex rules. Preserve that isolation unless you audit the collision and verify both app and library layouts. It is expected to be removed when upgrading to Angular 22 / OneCX v9.
- Do not add a hand-written surface palette, global PrimeNG/Optimus color aliases, or `!important` rules to fix theme colors. The v8-only `src/app/optimus-theme.ts` bridge should follow `.github/instructions/ui-migration-onecx-version-compat.instructions.md` and be removed/replaced by OneCX v9's supported Optimus theme provider.
- Compare Tailwind class values and grid semantics against the installed PrimeFlex CSS; same-named classes can have different values or meanings.
- After a v9 upgrade, remove only the temporary compatibility items listed in the "Remove after Angular 22 / OneCX v9" table of `.github/instructions/ui-migration-onecx-version-compat.instructions.md`.
- Component styles use SCSS (`inlineStyleLanguage: scss`)

### OneCX providers

`RemoteModule` provider setup follows a documented pattern. Relevant providers:

- `provideHttpClient(withInterceptorsFromDi())` — required
- `provideTranslateService(...)` with `OnecxTranslateLoader` and `MultiLanguageMissingTranslationHandler` — for i18n
- `provideAngularUtils()` — for permission checks / `PortalPage`
- `providePortalDialogService()` — for `PortalDialogService`
- `provideThemeConfig()` (from `@onecx/angular-utils/theme/primeng`) — for PrimeNG theming in the Shell
- `provideOptimusTheme()` (from `./optimus-theme`) — v8-only adapter that forwards the OneCX theme to Optimus; registered once in `AppModule` and `RemoteModule`, never in feature modules; delete at the OneCX v9 upgrade
- `provideAppInitializer(() => initializeRouter(...))` — required to sync MFE router with the Shell router

### Post-build

The `postbuild` script renames `styles.*.css` → `styles.css` in the dist output. This is required for the nginx/container setup.

### Structure

The whole application is under `src/app/`. Each feature module has its own folder. The `shared` folder contains shared components, directives, and pipes. The `core` folder contains the main `RemoteModule`, routing, and providers.

The `overview` module is the default route and contains a list of all showcase examples. Each example has its own route and lazy-loaded module. The `overview` module should contain technology-specific links to the showcase examples (e.g., "PrimeNG", "Angular Material", "OneCX Components").

The `onecx` module contains examples of OneCX components and features. The `primeng` module contains examples of PrimeNG components. The `material` module contains examples of Angular Material components. All showcase examples should be added to the proper module based on the technology used.

Inside each technology-specific module, the folder structure should be organized by feature area (e.g., `data-list-grid`, `interactive-data-view`, `content-container`, etc.).

For `primeng` module, each component example contains regular usage and appendTo="body" example if its applicable to the component.

For `material` module, each component example contains only regular usage of the component. Every example of angular material component is affected by the OverlayContainer provider done in remote.module.ts.