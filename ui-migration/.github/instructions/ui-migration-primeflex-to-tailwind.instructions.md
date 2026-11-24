---
description: "Use when replacing PrimeFlex utility classes with Tailwind: lookup table, measured-value rule, tw- prefix usage and size-parity evidence."
---
# PrimeFlex → Tailwind Migration Data (SHARED — target-independent)

<!--
  SHARED, target-INDEPENDENT data for the PrimeFlex → Tailwind swap.
  PrimeFlex→Tailwind is part of Phase A for BOTH targets (Angular Material AND
  Optimus UI) — the layout migration is identical regardless of which component
  library you swap PrimeNG into. This file is the SINGLE SOURCE OF TRUTH for the
  mapping so it is maintained in exactly one place.

  Referenced by:
    - ui-migration-prime-to-material.instructions.md (Material target)
    - ui-migration-prime-to-optimus.instructions.md   (Optimus UI target)
    - the shared hard rules (ui-migration-rules.instructions.md)

  It is NOT auto-injected (no applyTo) — the executor/validator read it on-demand
  whenever a PrimeFlex→Tailwind task is in scope (both targets).
  Separate scope from the Angular version migration (MIGRATION_PROGRESS.md).
-->

## Scope

PrimeFlex→Tailwind is a **mapping task, not a component swap**. PrimeFlex provides layout utilities (spacing, grid, flex, text) that have Tailwind equivalents; the goal is to replace the PrimeFlex classes on the **app's own markup** with size-equivalent Tailwind utilities. PrimeFlex classes used **only inside** PrimeNG component markup are handled by that component's task, not here. **Assume Tailwind is present** in the target repo (its default spacing/grid scale is `0.25rem` per unit); if it is NOT present, configure it first (add `tailwindcss`/PostCSS + a `tailwind.config` with `content` covering the app templates), then map.

**Do not assume matching class numbers have matching values or semantics.** PrimeFlex v4 has a non-linear spacing scale. For example, the installed PrimeFlex 4.0 CSS defines `.p-4` and `.gap-4` as `1.5rem !important`, whereas Tailwind v3 `p-4` / `gap-4` are `1rem`. PrimeFlex `.grid` is a flex-based 12-column layout, whereas Tailwind `.grid` is CSS Grid. These examples are warnings, not global mapping rules: inspect the installed PrimeFlex and Tailwind versions for every migration.

## Tailwind tooling (MCP + skills — MANDATORY for API lookups)

Tailwind is the NEW side of this mapping, and it has authoritative doc tooling. **Both targets** use it (it is target-independent).

**Tailwind MCP** (`mcp.tailwind.ai` — remote MCP server):

- **purpose**: authoritative Tailwind utility reference — exact class names, default scale values (spacing, sizing, grid), and which utility produces a given CSS value. Use it to VERIFY a chosen utility matches the measured PrimeFlex value, and to find the right arbitrary-value syntax.
- **config**: remote/URL MCP (no `npx`): `{"tailwind": {"url": "https://mcp.tailwind.ai"}}` — see `templates/mcp.example.json`.
- **verify**: probe a `tailwind/*` tool at Gate 3. If it errors as unconfigured → HALT-or-bootstrap (add the config from the template, with the developer's OK, then re-verify).

**Tailwind skills** (`npx @tailwind-app/skills`):

- **purpose**: Tailwind best-practice / migration guidance as agent skills — how to compose utilities, handle arbitrary values, and keep the `content`/`@source` coverage correct.
- **setup**: run `npx @tailwind-app/skills` (installs Tailwind guidance skills) where the project uses them; record in the progress file that it is available. (This is an additive best-practice aid — the MCP + the measured-value rule remain authoritative.)

**Doc sources (in priority order — the `node_modules` step is the LAST-RESORT fallback, VERIFY it in place of the web):**

1. **Tailwind MCP** (`tailwind/*`) — primary NEW-side reference.
2. `https://tailwindcss.com/docs` — official Tailwind documentation (web fetch; fall back to `curl -sSL` if policy-blocked). Do not use the historical `tailwindcss-com.vercel.app` mirror as the authoritative source.
3. `node_modules/primeflex/primeflex.css` — the ACTUAL installed PrimeFlex CSS (authoritative for the measured OLD-side values).
4. `node_modules/tailwindcss/` — the installed Tailwind default theme/scale (**LAST-RESORT** source for a Tailwind utility's value when the MCP and the docs are both unreachable — read the theme to VERIFY the exact `rem`/`px` value of the chosen utility, e.g. its spacing/sizing/font-size scale).
5. `https://v20.primeng.org/guides/primeflex` — PrimeFlex reference guide (version-pinned to the installed PrimeFlex major — re-derive against the installed major if it differs from v20).

**MCP discipline:** never blindly apply MCP output — confirm the returned utility/value matches the specific PrimeFlex class you are migrating. If the MCP / docs are unavailable, fall back to VERIFYING the value from `node_modules/primeflex/primeflex.css` + `node_modules/tailwindcss/` (record `node_modules/<pkg>` as the source). Record which source (Tailwind MCP / skills / node_modules / URL) drove each mapping decision.

## Use this lookup first (verified against PrimeFlex 4.0 and Tailwind 3.4; do not ask, just replace)

In the examples `tw-` is the Tailwind prefix. Use it when `tailwind.config.js` has `prefix: 'tw-'` (the OneCX v8 profile in `ui-migration-onecx-version-compat.instructions.md`); otherwise write the class without it. A variant goes before the prefix: `md:tw-w-1/2`.

**Spacing** (`p`, `px`, `py`, `pt`, `pr`, `pb`, `pl`, `m`, `mx`, `my`, `mt`, `mr`, `mb`, `ml`, `gap`): the PrimeFlex number is NOT the Tailwind number.

| PrimeFlex step | 0   | 1       | 2      | 3    | 4      | 5    | 6    | 7    | 8    |
| -------------- | --- | ------- | ------ | ---- | ------ | ---- | ---- | ---- | ---- |
| value          | 0   | 0.25rem | 0.5rem | 1rem | 1.5rem | 2rem | 3rem | 4rem | 5rem |
| Tailwind step  | 0   | 1       | 2      | 4    | 6      | 8    | 12   | 16   | 20   |

Example: `p-4` → `tw-p-6`, `mb-2` → `tw-mb-2`, `gap-3` → `tw-gap-4`, `px-4` → `tw-px-6`.

**Same name, same value (just add the prefix):** `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`, `flex`, `inline-flex`, `flex-wrap`, `flex-1`, `w-full`, `h-full`, `hidden`, `block`, `relative`, `absolute`, `overflow-hidden`, `overflow-auto`, `cursor-pointer`, `font-bold`, `font-semibold`, `text-center`, `text-left`, `text-right`. `text-3xl` is the exception: PrimeFlex is 1.75rem, use `tw-text-[1.75rem]`.

**Renamed:**

| PrimeFlex                                                 | Tailwind                                                                                |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `flex-column`                                             | `tw-flex-col`                                                                           |
| `justify-content-center` / `-between` / `-start` / `-end` | `tw-justify-center` / `-between` / `-start` / `-end`                                    |
| `align-items-center` / `-start` / `-end`                  | `tw-items-center` / `-start` / `-end`                                                   |
| `flex-grow-1` / `flex-shrink-0`                           | `tw-grow` / `tw-shrink-0`                                                               |
| `border-1` / `border-2`                                   | `tw-border tw-border-solid tw-box-border` / `tw-border-2 tw-border-solid tw-box-border` |
| `border-round`                                            | `tw-rounded-md`                                                                         |
| `border-round-sm` / `-md` / `-lg`                         | `tw-rounded` / `tw-rounded-md` / `tw-rounded-lg`                                        |
| `white-space-nowrap`                                      | `tw-whitespace-nowrap`                                                                  |
| `grid` (with `col-N` children)                            | `tw-grid tw-grid-cols-12` (add `tw-gap-6` where the old gutter was the default)         |
| `col-N`, `md:col-N`                                       | `tw-col-span-N`, `md:tw-col-span-N`                                                     |
| `w-N` percent such as `w-6`                               | `tw-w-1/2` (use the fraction that equals the percentage)                                |

**Theme colours** (Optimus Tailwind plugin; these are the only colour classes to use):

| PrimeFlex                      | Tailwind                                 |
| ------------------------------ | ---------------------------------------- |
| `surface-0`, `surface-card`    | `tw-bg-surface-0`                        |
| `surface-50`, `surface-ground` | `tw-bg-surface-50`                       |
| `surface-border`               | `tw-border-surface-300`                  |
| `border-400`                   | `tw-border-surface-400`                  |
| `text-color-secondary`         | `tw-text-surface-600`                    |
| `bg-primary`                   | `tw-bg-primary tw-text-primary-contrast` |

A class that is not listed: read its declaration in `node_modules/primeflex/primeflex.css`, then write the
same value as a Tailwind arbitrary value (`tw-w-[18rem]`, `tw-text-[1.75rem]`). Do not stop and do not ask.
Leave `ocx-*`, `p-*` component classes, `pi pi-*` icons, and classes the v8 OneCX components apply
to themselves untouched.

## Fallback hints (only for classes missing from the lookup above)

| PrimeFlex                                                                                | Tailwind equivalent                                                                                                            | Notes                                                                                                                                              |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.flex` / `.inline-flex`                                                                 | `flex` / `inline-flex`                                                                                                         | 1:1                                                                                                                                                |
| `.row` / `.col`                                                                          | Inspect the actual declarations and parent/child layout before choosing flex/grid utilities                                    | Their behavior depends on PrimeFlex version and surrounding `.grid` / `.col-*` structure; do not assume `.col` means `flex-col`.                   |
| `.flex-wrap` / `.no-wrap`                                                                | `flex-wrap` / `flex-nowrap`                                                                                                    | 1:1                                                                                                                                                |
| PrimeFlex `.justify-content-*`                                                           | Corresponding Tailwind `justify-*`                                                                                             | Property intent may match; verify selector spelling, generated declaration, responsive behavior and cascade.                                       |
| PrimeFlex `.align-items-*`                                                               | Corresponding Tailwind `items-*`                                                                                               | Property intent may match; verify selector spelling, generated declaration, responsive behavior and cascade.                                       |
| PrimeFlex `.align-content-*`                                                             | Corresponding Tailwind `content-*`                                                                                             | Verify exact source and resulting property; do not treat as row-gap by class name.                                                                 |
| `.grid`                                                                                  | `grid` only after confirming the old intent is CSS Grid; otherwise build the equivalent flex layout                            | PrimeFlex v4 `.grid` is flex-based, not CSS Grid; a direct rename can change layout.                                                               |
| `.col-<n>` (n = 1–12)                                                                    | Usually a Tailwind grid span only when the parent is intentionally CSS Grid; otherwise use measured flex-basis/width utilities | Similar column count does not make the models equivalent. Verify parent, gutters, wrapping and responsive behavior.                                |
| `.col` / `.col-auto`                                                                     | Derive from measured flex sizing and the complete row/grid context                                                             | Do not blindly map to `col-span-1` or `flex-1`; ask if the intended behavior is ambiguous.                                                         |
| `.col-offset-<n>`                                                                        | Derive from the original layout model; use `col-start-*` only if converting to CSS Grid                                        | Verify zero/one-based numbering and parent layout.                                                                                                 |
| `.row-<n>`                                                                               | Derive from the measured source behavior                                                                                       | Verify whether it represents a row layout, grid row, or unrelated utility in the installed PrimeFlex version.                                      |
| `.g-<n>` / `.gap-<n>`                                                                    | A measured Tailwind gap or arbitrary value                                                                                     | Verify the actual installed PrimeFlex selector; `.g-4` may not exist, and `.gap-4` values can differ substantially between PrimeFlex and Tailwind. |
| `.m-*`, `.mt-*`, `.mr-*`, `.mb-*`, `.ml-*`, `.mx-*`, `.my-*`                             | Measured Tailwind utility or arbitrary value                                                                                   | No blanket 1:1 rule. PrimeFlex and Tailwind use different scales in common releases.                                                               |
| `.p-*`, `.pt-*`, `.pr-*`, `.pb-*`, `.pl-*`, `.px-*`, `.py-*`                             | Measured Tailwind utility or arbitrary value                                                                                   | No blanket 1:1 rule. In the installed PrimeFlex 4.0, `p-4` is 1.5rem; Tailwind v3 `p-4` is 1rem.                                                   |
| `.m-0` / `.p-0`                                                                          | `m-0` / `p-0`                                                                                                                  | Often equal, but still verify the installed declarations and importance.                                                                           |
| `.m-auto` / `.m-5` (PrimeFlex "auto")                                                    | `m-auto` / `m-5`                                                                                                               | PrimeFlex `.m-auto` = auto margin                                                                                                                  |
| `.text-left` / `-center` / `-right` / `-justify`                                         | `text-left` / `-center` / `-right` / `-justify`                                                                                | 1:1                                                                                                                                                |
| `.text-<size>` (`.text-sm`, `-base`, `-lg`, `-xl`)                                       | Measured Tailwind text-size utility or arbitrary value                                                                         | Class spelling does not establish equal font size or line-height; verify both.                                                                     |
| `.text-<color>` (PrimeFlex color names)                                                  | `text-<tailwind-color>`                                                                                                        | PrimeFlex color names (`.text-red-500`) align with Tailwind palette where the names match                                                          |
| `.bg-<color>`, `.border`, `.rounded`, `.shadow-*`, `.overflow-hidden`, `.cursor-pointer` | same names                                                                                                                     | 1:1 where the name matches                                                                                                                         |
| `.flex-grow` / `.flex-shrink` / `.flex-1`                                                | `grow` / `shrink` / `flex-1`                                                                                                   | PrimeFlex `.flex-grow`→`grow`, `.flex-shrink`→`shrink`, `.flex-1`→`flex-1`                                                                         |
| `.gap` helpers / `.gap-<n>`                                                              | Measured Tailwind gap or arbitrary value                                                                                       | Verify the exact PrimeFlex version and declaration; same names can carry different values and `!important`.                                        |

## Mapping rules

- Use the lookup at the top of this file. For anything else, measure the PrimeFlex value and write it as an arbitrary Tailwind value. Record the measured value and the class you chose in the task evidence.
- Where there is no exact Tailwind step, use an arbitrary value (`tw-gap-[1.1rem]`) instead of the nearest step.
- Verify the target repo's Tailwind `content` paths cover the templates being edited so the generated utilities are built.
- Never stop to ask which of two Tailwind classes to use; pick the one whose computed value matches.

## ⚠️ Size-difference verification (MANDATORY — the PrimeFlex scale ≠ the Tailwind scale)

Do NOT map class-name-to-class-name. PrimeFlex and Tailwind define many utilities at **different sizes** (for example, in the installed PrimeFlex 4.0, `gap-4` is `1.5rem !important` while Tailwind v3 `gap-4` is `1rem`; a PrimeFlex `.text-<size>` may also differ from Tailwind's `text-<size>`). The result must preserve the measured intent and layout, not merely use a similar-looking class. For every mapped utility that carries a measurement:

1. Read the **actual CSS value** of the PrimeFlex class from the installed `node_modules/primeflex/primeflex.css` (or `https://v20.primeng.org/guides/primeflex`, re-derived to the installed PrimeFlex major) — not the class name.
2. Confirm the matching Tailwind utility's value via the **Tailwind MCP** (`tailwind/*`) or official `https://tailwindcss.com/docs`. **If both are unreachable, VERIFY the value from the installed `node_modules/tailwindcss/` default theme/scale (last resort).** Never infer a PrimeFlex→Tailwind size mapping from the class number: both default scales vary, and same-named utilities can differ in value, `!important`, or layout semantics.
3. When no Tailwind step matches exactly, use an **arbitrary value** (`m-[0.75rem]`, `gap-1.75rem`, `p-2.5`, `text-[0.875rem]`) to preserve the exact measurement.
4. Record in the task evidence: the **measured PrimeFlex source value** (e.g. `0.875rem`) + the **chosen Tailwind value/utility** (e.g. `text-[0.875rem]`), so size parity is auditable.

This rule is shared by **both** targets — Tailwind for Material and for Optimus, size-checked in both.
