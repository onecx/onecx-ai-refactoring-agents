---
description: "Use when running the PrimeNG to Angular Material / Optimus UI migration. Project-specific rules and DO-NOT-REPEAT anti-patterns."
---
# Custom Project Instructions

<!--
  FILL IN YOUR PROJECT-SPECIFIC RULES BELOW.
  This file is auto-injected into all migration agents.

  Examples of what to add:
  - Workspace-specific component patterns and working examples
  - Custom library import conventions
  - Known working references in your repo (e.g., "See src/app/components/data-view.ts for DataView pattern")
  - Project-specific OneCX configuration details
  - Files or patterns to avoid modifying
  - Team coding standards and naming conventions
  - CSS/SCSS import patterns used in your project
  - Permission mapping conventions
-->

## Anti-Patterns (DO NOT REPEAT)

<!-- These are real mistakes from past migrations. Keep this list updated. -->

- **Halfway completion**: "I did 50% of the task" → task stays `[ ]`, complete ALL sub-steps
- **Guessed implementations**: "MCP gave unrelated code, I'll use it" → verify with workspace examples first
- **Modified wrong files**: "CSS broken in shell-ui, I'll fix it" → fix in YOUR component, not library files
- **Applied unrelated examples**: "Different microfrontend has similar component" → verify pattern matches YOUR repo
- **Duplicate configurations**: "Added to both component.ts and bootstrap.ts" → only one place per config
- **Hardcoded replacement decisions**: "Kept ocx-portal-viewport because previous run said so" → check CURRENT FETCHED DOCS
- **Incomplete migration**: "Migrated 3 imports, others look similar" → migrate ALL mentioned in docs
- **Permission mapping errors**: "Mapped all actions to #DELETE" → use correct permission per action type (#SEARCH, #IMPORT, #EXPORT, #EDIT)
- **CUSTOM_ELEMENTS_SCHEMA workaround**: "Added CUSTOM_ELEMENTS_SCHEMA to suppress unknown element" → NEVER do this; install the real package or reclassify the task to a later phase
- **Partial execution with bridge hacks**: "Did HTML replacement but deferred module/provider setup" → if you can't do ALL sub-steps, leave the task `[ ]` and reclassify to the correct phase
- **Reverting correct changes on build failure**: "Build failed after migration step, reverted everything" → diagnose root cause; if it's a missing package, reclassify; NEVER revert a correct migration change
- **Installing incompatible packages**: "Tried npm install with --legacy-peer-deps, it removed 258 packages" → always check `npm view <pkg>@<ver> peerDependencies` BEFORE installing; if incompatible, reclassify the task to a later phase
- **Test skipping**: "Tests will be fixed later" → fix in same invocation
- **Assuming Optimus is only a rename**: component selectors may stay `p-*`, but OneCX v8 still configures PrimeNG. Read `ui-migration-onecx-version-compat.instructions.md`; prove Shell theme propagation, token scoping and overlays.
- **Copying v9 setup into v8**: compare installed Angular, OneCX, Optimus, Tailwind and PrimeFlex majors first. The Angular 22 / OneCX v9 app is a visual/cleanup reference, not a directly copyable v8 provider or CSS pipeline.
- **Inventing colors or broad CSS overrides**: use the compatible Optimus Tailwind plugin and design tokens. Add no hardcoded palette, global `!important`, or alias rules unless installed APIs cannot satisfy a documented need and the user approves a removable v8-only adapter.
- **Assuming utility names/values match**: inspect installed PrimeFlex CSS and Tailwind output, including `!important`, layers and grid semantics. `.grid` and `.p-4` can have different meanings/values.
- **Blindly removing legacy UI dependencies**: scan OneCX peer dependencies, CSS imports, runtime references and `npm ls` before removal; list verified future v9 cleanup separately.
- **Omitting the upgrade handoff**: every completed Optimus migration must end with a concrete “Remove after Angular 22 / OneCX v9 upgrade” checklist for adapters, prefix/config, legacy CSS/dependencies and federation, each with a verification condition.
