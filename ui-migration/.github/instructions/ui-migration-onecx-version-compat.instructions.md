---
description: "Use for every Optimus UI migration on OneCX v8 / Angular 21: profile detection, v8 theme adapter, tw- Tailwind prefix, single icon font, and the Remove-after-OneCX-v9 handoff."
---
# Optimus on OneCX v8 (Angular 21): required setup, no guessing

Read this for every Optimus migration. Do not ask the developer about anything decided here.
No reference app exists in a real migration, so everything needed is in this file.

## 0. Detect the profile (one command)

```bash
node -e "for (const p of ['@angular/core','@onecx/angular-utils','@onecx/angular-accelerator','@openng/optimus-ui','tailwindcss','primeflex','primeng']) console.log(p, require(p+'/package.json').version)"
grep -o '"./theme/[a-z]*"' node_modules/@onecx/angular-utils/package.json
```

- Exports `./theme/primeng` only, accelerator imports PrimeFlex -> **v8 profile: do sections 1-7 exactly.**
- Exports `./theme/optimus` -> **v9 profile**: use `provideThemeConfig` from `@onecx/angular-utils/theme/optimus`,
  Tailwind v4 + `@openng/optimus-ui-tailwindcss` CSS import, **no** `tw-` prefix, no adapter. Skip 1-7.

## 1. Why a v8 adapter is mandatory

OneCX v8 `provideThemeConfig()` themes **PrimeNG only**. Optimus gets no preset, so buttons and overlays
render black or transparent and nothing follows the Shell theme. Keep `provideThemeConfig()` (OneCX
components need it) and add the adapter below. Never add a hardcoded palette, `--op-*` variables,
`light-dark()` or `!important` colour rules instead.

## 2. Install

```bash
npm i @openng/optimus-ui@1 @openng/optimus-ui-themes@1
npm i -D @openng/optimus-ui-tailwindcss@1 tailwindcss@3
```

Pin the Optimus major to 1 (Angular 21). Keep `primeng`, `primeflex`, `primeicons`: OneCX v8 needs them.

## 3. Adapter: create `src/app/optimus-theme.ts`

Verify first that `@onecx/angular-utils/theme/primeng` exports `CustomPreset`, `CustomUseStyle`,
`normalizeKeys`. If any is missing, stop and report it.

```ts
import {
  effect,
  inject,
  Injectable,
  provideEnvironmentInitializer,
  untracked,
} from '@angular/core';
import { CustomPreset, CustomUseStyle, normalizeKeys } from '@onecx/angular-utils/theme/primeng';
import { Base } from '@openng/optimus-ui/base';
import { Optimus, provideOptimus } from '@openng/optimus-ui/config';
import { UseStyle } from '@openng/optimus-ui/usestyle';
import { Theme } from '@openng/optimus-ui-styled';
import { definePreset } from '@openng/optimus-ui-themes';
import Aura from '@openng/optimus-ui-themes/aura';
import { PrimeNG } from 'primeng/config';

/** v8 scopes only PrimeNG styles. Reuse its implementation for Optimus, with distinct style IDs. */
@Injectable()
class OptimusV8UseStyle extends CustomUseStyle {
  override use(css: string, options: Parameters<CustomUseStyle['use']>[1] = {}) {
    return super.use(css, { ...options, name: `optimus-${options.name ?? 'style'}` });
  }
}

/** Remove when provideThemeConfig comes from OneCX v9 `theme/optimus`. */
export function provideOptimusTheme() {
  Theme.clearLoadedStyleNames();
  Base.clearLoadedStyleNames();
  return [
    { provide: UseStyle, useClass: OptimusV8UseStyle },
    provideOptimus({
      theme: {
        preset: definePreset(normalizeKeys(Aura), CustomPreset),
        options: { darkModeSelector: false },
      },
    }),
    provideEnvironmentInitializer(() => {
      const primeNG = inject(PrimeNG);
      const optimus = inject(Optimus);
      // v8 already maps Shell theme changes into primeNG.theme(); forward it, do not rebuild it.
      effect(() => {
        const theme = primeNG.theme();
        if (!theme) return;
        untracked(() =>
          optimus.setThemeConfig({
            theme: { ...theme, preset: definePreset(normalizeKeys(Aura), theme.preset) },
          }),
        );
      });
    }),
  ];
}
```

## 4. Register it once per bootstrap path

Add `provideOptimusTheme()` directly after `provideThemeConfig()` in **both** `src/app/app-module.ts` and
`src/app/remote.module.ts`. Delete every `provideOptimus()` / `providePrimeNG()` inside feature modules.

## 5. Feature modules and components

- Remove `AngularAcceleratorPrimeNgModule` from every feature module. It brings PrimeNG `p-*` components
  that clash with the Optimus ones. Import the specific `@openng/optimus-ui/<name>` modules the template
  uses (for example `ButtonModule`, `SelectModule`, `FloatLabelModule`). Keep `AngularAcceleratorModule`.
- Tooltip `appendTo`: PrimeNG `'target'` is Optimus `'self'`. Guard `this.container` before setting styles.
- Popover pass-through variable: apply the prefix replacement to the **whole** declaration:
  `replacePrimengPrefix(`${$dt('popover.arrow.left').name}: ${$dt('popover.arrow.left').variable}`, id)`
  with `$dt` imported from `@openng/optimus-ui-styled`.
- Do not reformat or re-indent lines you did not change (no whole-file prettier runs).

## 6. Styles and Tailwind

`tailwind.config.js`:

```js
const optimusUi = require('@openng/optimus-ui-tailwindcss');
module.exports = {
  content: ['./src/**/*.html', './src/**/*.ts'],
  prefix: 'tw-',
  plugins: [optimusUi],
};
```

`src/styles.scss` starts with exactly:

```scss
@tailwind components;
@tailwind utilities;
```

then keeps the existing Material theme import, `@import 'node_modules/@onecx/angular-accelerator/assets/styles.scss';`
(v8 components need its PrimeFlex) and `@import 'primeicons/primeicons.css';`.

Rules:

- Prefix is `tw-` on v8: PrimeFlex ships `!important` utilities with the same names (`.grid`, `.p-4`, `.gap-4`).
  Unprefixed Tailwind would break OneCX components or lose to PrimeFlex.
- No `@tailwind base` (preflight would reset the Shell), no `important: true`, no `@layer` ordering tricks,
  no custom colour classes (`surface-50`, `surface-border`, `border-round` ...). Use the plugin utilities.
- A border needs three classes because there is no preflight: `tw-border tw-border-solid tw-box-border`
  (same for `tw-border-2`).
- Responsive variant goes first: `md:tw-w-[30rem]`, `sm:tw-col-span-6`.
- Convert classes everywhere, not only `.html`: `styleClass="..."`, `[class]`, `styleClass = signal('...')`
  and inline `template:` strings in `.ts` files.
- Icons: one font only. Keep `primeicons/primeicons.css`; do not also import `@openng/icons/openng-icons.css`.
  Renaming `PrimeIcons` to `OpenngIcons` in code is fine.
- Do not add a manual `@openng/*` share loop to `module-federation.config.js`; leave the auto config.

## 7. Verify (build passing is not enough)

1. `npm run build`, `npm run lint`.
2. Spec of each changed file by path (see rules file). No spec: record `no spec file for <path>`.
3. Open the built `styles.css` and confirm: `.tw-p-6` exists, `.p-4` is still `1.5rem !important`,
   `.tw-grid` is `display:grid` while `.grid` is `display:flex`, `.tw-border-solid` exists, responsive
   `tw-` rules sit inside `@media`, and no `--op-` string exists.
4. In the running app: a datepicker panel and a select panel have an opaque background and visible border,
   in both `appendTo="body"` and container mode; buttons use the Shell primary colour; icons render.
   If the Shell cannot be opened (login, federation), write `runtime check BLOCKED: <reason>`. Never write PASSED.

## 8. Final handoff: "Remove after Angular 22 / OneCX v9"

End the migration with this list in `UI_MIGRATION_PROGRESS.md` and in the final answer, filled in with
the real files found:

| Item                                                                       | Action at v9                                                                                             |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `src/app/optimus-theme.ts` and both `provideOptimusTheme()` calls          | Delete. Use `provideThemeConfig` from `@onecx/angular-utils/theme/optimus`                               |
| `prefix: 'tw-'` and every `tw-` class (HTML, TS strings, inline templates) | Remove the prefix, rename `tw-x` to `x`                                                                  |
| Tailwind 3 + `@openng/optimus-ui-tailwindcss@1` config                     | Replace with Tailwind 4, Optimus 2 plugin CSS import, `@onecx/build-utils` PostCSS plugin                |
| PrimeFlex import via accelerator styles, `primeflex` package               | Remove only if the v9 accelerator no longer imports it (`grep primeflex` in its `assets/styles.scss`)    |
| `primeng`, `primeicons`, `@primeuix/*` packages                            | Remove only when `npm ls primeng` shows no OneCX package needing it and `grep -r "primeng" src` is empty |
| `@openng/optimus-ui*` v1                                                   | `ng update @angular/core@22 @angular/cli@22`, then `ng update @openng/optimus-ui@2`                      |

State that this is a plan for the upgrade: do not remove anything now.

Docs: https://v1.optimus.openng.org/tailwind, /theming/styled, /configuration, /migration/update;
https://tailwindcss.com/docs. The `onecx-docs-mcp` URL is an MCP endpoint, not a web page.
